---
layout: post
title: Building a Refactoring Talk - Part 1
---

Lately, I've been working on a talk I'm doing on refactoring. I'll be this giving talk at [Ruby Hoedown 2010](http://rubyhoedown.com) in early September in Nashville, TN and at [Sunnyconf](http://sunnyconf.com) in Phoenix, AZ later in the month. 

The goals for this talk are extremely aggressive, but it's helpful to lay them out:

1. Lay out core principles of refactoring
1. Present conceptual framework for executing small refactorings
1. Demonstrate common refactoring techniques
1. Demonstrate web-app specific anti-patterns and techniques

At it's core, this talk will not have a "cookbook" agenda, but rather a "patterns" agenda. My hope is to impart some piece of lasting knowledge on the audience so it can be applied in their domains. A "recipe" approach wouldn't really work here, as refactoring is inherently a "before and after" process.

## Examples, examples

One of the most difficult parts of constructing this talk is selecting appropriate examples through which to demonstrate refactoring techniques. One of the primary considerations I've been wrestling with is that of domain complexity in my examples.

### Should I "dumb down" the domain?  

I'm hesitant to "dumb down" the domain, mostly because one of the largest barriers in large refactorings is domain complexity itself. Plus, I find overly contrived examples in presentations off-putting and leaving me wanting more. I don't want to exclude portions of the audience due to a small oversight like this. Plus, being that I already work in a fairly complex domain, constructing these examples would be a fairly simple task.

On the other hand, there's also a good chance that an overly complex domain will work against absorption of the key subject matter, but in a different way. I wouldn't want the audience or reader to struggle to understand the refactoring concepts and techniques simply because they're too busy trying to grasp dizzying complexities of fault-tolerant aerospace systems, or whatever. Still, while domain complexity indeeds adds to the overall difficulty of performing large refactorings, refactoring is a sufficiently difficult topic on it's own -- it doesn't need any help in that area. 

So, for the example material, I'm hoping to find an adequate middleground. Examples that reside within sufficiently complex domains that effectively demonstrate large-scale refactoring techniques that neither exclude people with overly contrived examples nor paralyze attention spans with the minutiae of some extremely complex domain. 

The hard part, of course, is identifying the middleground. And I am bad at gray areas and middlegrounds. 

This seems like enough direction to run with. Now, off to find that perfect domain. As always, thoughts and comments are welcome.
