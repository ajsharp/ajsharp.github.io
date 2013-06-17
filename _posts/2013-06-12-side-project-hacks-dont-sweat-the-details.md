---
layout: post
title: Side Project Hacks: Don't Sweat the Details
---

## *Conserve momentum. Don't sweat the details. Focus on Shipping.*

Lately, I’ve been spending a lot of my nights and weekends working on a side project called [Octocall](https://octocall.com), a service to simplify meetings and conference calls for businesses.

Working on a side project is a lot like working on a very early-stage startup: It’s pre-launch, time and resources are *extremely limited*, and **the primary goal is to ship something and learn.**

This point is important to re-iterate: Side projects are not about perfecting the product, nailing the implementation, besting the [competition](http://www.uberconference.com), being feature-complete, or making money.

The goal of a side project is singular and simple: **ship something.**

Once you accomplish this critical first step of launching, only then should you worry about revenue, user acquisition, competition, etc.

## Details, Time and Momentum

[Maintaining momentum](http://executebook.com) is *critical* in this unique pre-launch stage. Focus on core value and the MVP instead of sweating the details.

Time and momentum are the two most precious resources on a side project. Get caught up tweaking that shade of gray for 15 minutes, and you’ve lost precious time instead of making progress towards shipping. Nailing that shade of gray isn’t real progress, it’s just tinkering.

These types of details are a massive time-suck and they deplete momentum. Build a sense for recognizing these rabbit holes and avoid them like the plague.

As an example, I was recently working on a “quick call” feature for Octocall. This feature is for users who want to jump on a call with others immediately. In order to select the call participants, an autocomplete UI component seemed appropriate.

I’ve always found the existing open-source autocomplete plugins clunky to work with, so from the get-go I ruled out wasting time integrating an existing library, and I decided to write my own.

The mechanics of a basic autocomplete UI component are pretty straightforward:

1. Take in user input from a form field

1. Send the user input to the server to search for matches

1. Display matches in a drop-down selector below the text input field

1. Allow the user to make a selection

![](https://d233eq3e3p3cv0.cloudfront.net/max/700/0*-08KpOU19yjnZwji.png)

*Octocall autocomplete UI*

## Little Details

After about 1-2 hours, the core functionality of the autocomplete dialog worked. You could type in a few letters, receive results from the server, select someone and they’d be added to the call. At this point, my momentum was high, having made great progress in a short timeframe.

But the part that tripped me up was the selection of one of the participant matches. Personally, when I see an autocomplete component, I like to be able to make the selection without using the mouse.

There are two ways to do this: with the “up” and “down” arrow keys, and with “Ctrl-p” and “Ctrl-n”, if you’re on OS X. I like the latter much better because your fingers don’t leave the home row, and this UI convention is supported throughout OS X whenever text input is accepted. So, I figured that it’d be cool to support that functionality when selecting a contact for an Octocall.

It’s important to note that at this point I had a fully functional UI component. The thing *worked*. The basic “up” and “down” arrow key navigation worked *perfectly*. I was riding high on a wave of momentum. But the ctrl-n / ctrl-p thing was *irking me*. The detail-oriented nut in me was crying out: “keep going!”

So I did. And I got really, really close. And then I gave up.

After about 30-45 minutes of working on the ctrl-p / ctrl-n functionality, I realized I was wasting precious time working on the *tiniest little detail* that most people don’t even know exists, and probably wouldn’t use it if they did.

I did a quick final test -- does Gmail let you navigate autocomplete results using this technique? Nope. Cut it. Even if they did, it wouldn’t have mattered. *I should never even have gone down this road.*

The only thing left to do was to cut my losses, try to rebuild my momentum and *get back to work*. Because my little rabbit hole expedition wasn’t work, it was tinkering with meaningless details, **gambling with my momentum**. And I busted.

The core of the autocomplete component works really well. It lets you jump on a call with as many people as you want in seconds. That’s what’s important, that’s the core value of the feature.

## Momentum: The Hidden Cost

Coming out of one of these tangents, you feel like you were in a trance and you just returned back to your senses. You’re left with this feeling of, “wait, shit, I just burned 30 minutes trying to build this silly little thing, and it was mostly to impress myself to see if I could do it, and I didn’t even finish.” And now you’re a little more bummed than you were before, and you’ll probably take a break to stretch, and then spend another 10 minutes making a cup of coffee, and maybe playing with the dogs for a few minutes while you’re up, and maybe make a PB&J (and God, how I love a PB&J).

The hidden cost of these little rabbit holes isn’t time -- it’s **momentum**. As precious as time is at this stage, momentum is your fuel, and it’s nearly impossible to replenish. And chasing down rabbit holes *drains your momentum* faster than anything.

A gut full of momentum is like rocket fuel to progress and productivity, but an empty tank means yet another un-finished side project. So preserve your momentum as if it were an irreplaceable precious metal, the one and only thing that will enable you to ship your side project. Because it is.