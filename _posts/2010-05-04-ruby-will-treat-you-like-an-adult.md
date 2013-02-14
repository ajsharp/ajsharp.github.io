---
layout: post
title: Ruby Will Treat You Like an Adult
---

One of the things that I love most about Ruby is that it is an "adult's language". It is a very powerful language; as such, it doesn't put many restrictions on what you can do with your objects and classes. 

Most of are familiar with the ability open up and modify any class in Ruby, including what might be referred to in other languages as "primitives". Take the following example:

{% highlight ruby %}
1 + 1 # => 2
 
class Fixnum
  def +(num)
    puts "You're playing with fire there buddy!"
  end
end
 
1 + 1 # => "You're playing with fire there buddy!"
{% endhighlight %}

Ok, we can open up any class and play around and be really destructive. Ok, so maybe we're not going to get ourselves into much trouble trying to do something contrived like override `+`, but Ruby provides many other ways to subvert object oriented principles. Probably the most well-known of these is the `Object#send` method. 


`send` if used correctly, allows you to completely subvert encapsulation without even missing a beat.

{% highlight ruby %}
class Person  
  private
    def dirty_secrets
      "OMG! You can totally see my secrets!"
    end
end
 
Person.new.send(:dirty_secrets) # => "OMG! You can totally see my secrets!"
{% endhighlight %}

Whoa! Aren't private methods supposed to be private!? Well, if we had tried to simply call `Person.new.dirty_secrets` our secrets would have been safe, and Ruby raises an error in this case. However, when we use `send`, we don't get a warning, no slap on the wrist, nothing. Nope, we just move happily long, and our secrets are out. 

Ruby truly is an adult's language. While Ruby is extremely object-oriented (at least in the sense that nearly _everything_ in Ruby is an object), we're given the power to completely subvert and override encapsulation, a basic principle of object-oriented programming. Kinda cool, but as I found out the hard way, this can be dangerous if you're not careful. 

At this point, there's a decent chance you're thinking to yourself, "Listen GUY, I'm a good developer, I would never do something so arrogant as to use send on a private method. Well...at least not in _production_ code, but I suppose I've done it in my tests before. But that's not a big deal. They're _just_ tests."

So let's drop the contrived examples and look at some real code. Indeed, I got a bit arrogant, and it came back to bite me, and by "bite me" I mean that I introduced a bug into what my tests told me were a stable code-base. 

### Bunyan

About a month or so ago, I released a project called [bunyan](http://github.com/ajsharp/bunyan). Bunyan is a very simple project that provides a thin wrapper around a [MongoDB capped collection](http://www.mongodb.org/display/DOCS/Capped+Collections). I created it because capped collections are really powerful, and we had a need to begin using them to log additional data on every request. Bunyan sits on top of the [mongo ruby driver](http://github.com/mongodb/mongo-ruby-driver) to keep the API clean, simple and familiar.

Shortly after we began using it, it quickly became a bit of a nuisance for other developers who didn't have Mongo installed on their local dev machines to start up a copy of our app. Essentially, I had created another external dependency on another piece of software that was impeding our development process. This is not good. So I decided that Bunyan should fail silently, output a message to `$stderr` that it couldn't connect to Mongo. Cool.

At this point I need to explain a bit about how Bunyan works. Basically, Bunyan defines _very_ few methods of it's own. In order to keep it as lightweight as possible, Bunyan uses `method_missing` to pass nearly all method calls through to a Mongo capped collection. In addition to keeping things simple, this also means that all calls to Mongo other than initializing the connection happen right there in the `method_missing` method.

{% highlight ruby %}
module Bunyan
  class Logger
    include Singleton
    # ...
    def method_missing(method, *args, &block)
      begin
        collection.send(method, *args) if database_is_usable?
      rescue
        super(method, *args, &block)
      end
    end
    # ...
  end
end
{% endhighlight %}

So, in implementing this silent failure feature, I need to ensure that if there is a problem connecting to Mongo, then method calls do not get passed on to a Mongo collection that doesn't exist. The way I chose to accomplish this was to use a `@disabled` configuration option I introduced in the initial release. 

This is a configuration option that the user can set from the Bunyan configuration block as a way to turn Bunyan off without commenting out the whole configuration block. Naturally, I figured I would set `@disabled = true` in a rescue block if an error occurs while connecting to Mongo (In retrospect, I'm not too crazy with the idea of "over-loading" a configuration attribute like that, as a user-disabled Bunyan is somewhat different than Bunyan needing to be turned due to connection problems).

#### Configuration refactor

So here's where things get tricky. Initially, I was handling all the configuration stuff and the logging in one big class. It soon became apparent that I needed a separate configuration class to handle all this logic. Without going into too much detail, it's safe to say this was a fairly major refactor of the configuration logic ([commit](http://github.com/ajsharp/bunyan/commit/f9d609aacd403742e8d5f0f1d99d92d7100cda9b)). 

So, when I went about to actually do this, I forgot that I had moved the `@disabled` variable and more importantly, the `disabled` accessor method, to the configuration class. In other words, doing what I did on line 12 below had absolutely no effect on anything.

{% highlight ruby %}
module Bunyan
  class Logger
    include Singleton
    # ...
    private
      def initialize_connection
        begin
          @db = Mongo::Connection.new.db(config.database)
          @connection = @db.connection
          @collection = retrieve_or_initialize_collection(config.collection)
        rescue Mongo::ConnectionFailure => ex
          @disabled = true
          $stderr.puts 'An error occured trying to connect to MongoDB!'
        end
      end
    # ...
  end
end
{% endhighlight %}

Even worse, my tests told me so, because they were failing. I should've known better right then. But no, I was arrogant. Here's a re-enactment of the conversation my test suite and I had that day:
> **_Ruby_**: No go bro! Yer doin it wrong.

> **_Alex_**: No way man! I'm setting the `@disabled` variable right there. 

> **_Ruby_**: I'm telling you bro, that mess is *not* disabled!

> **_Alex_**: Pfft! It's probably just some shared-state crap between my tests because I'm using the `Singleton` module to do all of this. Yea, that _has_ to be it.

### instance\_variable\_get should come with a poison label on the box

At this point, I made the biggest mistake of the day -- **I changed my tests to make my failing code pass**. If you find yourself feeling the need to do this, before you type another stroke, take a five minute break and re-think your inks, because you're doing it wrong ;) 

The second example below is the one to focus on, and specifically line 20 where I felt the need to use `instance_variable_get` to subvert encapsulation and pull the instance variable out of the class directly. 

Again, I cannot stress enough how bad of an idea this really is.

{% highlight ruby %}
describe 'when a mongod instance is not running' do
  before do
    Mongo::Connection.stub!(:new).and_raise(Mongo::ConnectionFailure)
  end
 
  it 'should not blow up' do
    lambda {
      Bunyan::Logger.configure do |c|
        c.database 'doesnt_matter'
        c.collection 'b/c mongod isnt running'
      end
    }.should_not raise_exception(Mongo::ConnectionFailure)
  end
 
  it 'should mark bunyan as disabled' do
    Bunyan::Logger.configure do |c|
      c.database 'doesnt_matter'
      c.collection 'b/c mongod isnt running'
    end
    Bunyan::Logger.instance.instance_variable_get(:@disabled).should == true
  end
end
{% endhighlight %}

Of course, the reason the tests were failing is because I should've been setting `@config.disabled = true`. `@disabled` was a deprecated property.

### Listen to your tests, don't subvert encapsulation, and don't be a jerk

The moral of this story is that you should listen to your test suite when it's trying to tell you something.