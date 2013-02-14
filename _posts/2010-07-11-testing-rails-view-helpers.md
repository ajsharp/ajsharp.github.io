---
layout: post
title: Testing rails view helpers
---

Rails view helpers are easy to overlook in your test suite, but they want your testing love just like your models. I generally use view helpers for one of two reasons: 1.) when I'm trying to output something that's a bit too hairy to leave in a view or 2.) when I want to DRY up some view code. Both of these reasons warrant testing this code.

At first, view helpers can be kinda awkard to test, but in this post I'll show that testing view helpers can be quite simple. 

## Mongoflow == Rubyflow + Mongoid + Rails 3 + Refactoring Love

Recently [@theriffer](http://twitter.com/theriffer) and I have been hacking on [mongoflow](http://mongoflow.heroku.com), a [MongoDB](http://mongodb.org) link aggregator heavily inspired by [rubyflow](http://rubyflow.com). In fact, the super-awesome [Peter Cooper](http://peterc.org/) was gracious enough to open source the original rubyflow codebase. We've decided to go with [mongoid](http://mongoid.org) as the Mongo ODM (object-document mapper) and rails 3. Refactoring Peter's original version of Rubyflow to rails 3 has been a fun little project. (To view the code for MongoFlow, check out the [github project](http://github.com/brainscott/mongoflow).) 

## Rails 3 form error helpers

As I was clicking through the app, I noticed some deprecation warnings flying by in the server output. As it turns out, the `error_messages_for` and `f.error_messages` methods [have been deprecated in rails 3 beta 3](http://weblog.rubyonrails.org/2010/4/13/rails-3-0-third-beta-release) and moved to plugins. I was never really crazy about the default error message style, so I decided to write my own super-simple helper method to show error messages for an object. 

Keep in mind that rails view helpers are simply modules that get included into an `ActionView` template. Thus, the methods you define in a helper are available to you in templates as instance methods. However, we often use helpers to abstract creating HTML markup, and we often do this by leveraging the rails view helper methods (such as `content_tag`). 

So in order to test view helpers that use the rails helper methods, we need to simulate the scope in which we would otherwise be calling our helper methods - a view template instance.

{% highlight ruby %}
 module ApplicationHelper
  def form_errors(obj)
    content_tag :ul, :class => 'errorExplanation' do
      obj.errors.full_messages.collect { |msg| content_tag :li, msg }.join('')
    end
  end
end
 
describe ApplicationHelper do
  class MockView < ActionView::Base
    include ApplicationHelper
  end
  
  describe '#form_errors' do
    class FakeItem
      include Mongoid::Document
 
      field :title
      validates_presence_of :title
      validates_length_of   :title, :minimum => 8
    end
 
    before(:each) { @template = MockView.new }
    after(:all)   { FakeItem.destroy_all         }
 
    it 'should display multiple errors hash in a list' do
      item = FakeItem.create
      msg1 = item.errors.full_messages[0]
      msg2 = item.errors.full_messages[1]
 
      @template.form_errors(item).should == "<ul class=\"errorExplanation\"><li>#{msg1}</li><li>#{msg2}</li></ul>"
    end
 
    it 'should display one error in a list' do
      item = FakeItem.create :title => "2short"
      msg = item.errors.full_messages[0]
 
      @template.form_errors(item).should == "<ul class=\"errorExplanation\"><li>#{msg}</li></ul>"
    end
  end
end
{% endhighlight %}

## Rails to the rescue!

Luckily, creating an instance of `ActionView::Base` is _absolutely trivial_. All we need to do is create a new class that extends `ActionView::Base`, include our helper module, and instantiate it. That's it. No params, nothing. Boom. Awesome. You can see where I've created the `MockView` class on line 11. 

By extending from `ActionView::Base`, we get access to all the other helper modules that rails would give you in a view template, in our test suite. If we had just created a plain-vanilla ruby class for `MockView`, trying to call the `content_tag` method on it would raise a `NoMethodError`. 

The second class, `FakeItem`, is an actual valid Mongoid model, with a `title` field and two validations on that field. 

The reason for that I've created a proper Mongoid class (as opposed to stubbing certain methods or creating a mock object) is that Mongoid leverages the new `ActiveModel` API, which is where validations and validation errors (among other things) are handled in rails 3. The implementation of the `form_errors` method is dependent upon the `ActiveModel` api, and I would rather have access to the real errors objects, as opposed to attempting to stub out the portion of the api we're using in the `form_errors` method, which could get messy. Plus, creating classes in Ruby is so trivial, so why not!?

In the first example, the model instance will get two errors placed on it by `ActiveModel`. To test the `#form_errors` method, I'm simply asserting it outputs the correct html markup around the error messages coming from the model. A bit brute force, yes; but it's the best way that I know of to test the entire `#form_errors` method, rather than just pieces of it.

An alternative might have been to place message expectations on the various calls to the `content_tag` method, but that's a lot of magic, and not much benefit. And I [hate magic](http://www.youtube.com/watch?v=AYxu_MQSTTY) ;)

I hope you've found this post useful. I'd love to see how other people are testing view helpers, so please post what you're doing in the comments.
