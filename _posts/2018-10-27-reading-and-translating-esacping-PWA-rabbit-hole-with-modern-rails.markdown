---
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
layout: single
title:  "Reading and translating 'esacping PWA rabbit hole with modern rails'"
date:   2018-10-27 15:45:43 -0700
categories: jekyll update
---
SPA is getting more and more popular in recent years. Javascript frameworks
for making SPA came out one after another, from Backbone, Ember, Angular
to React, Vue, etc just to name a few. They are getting better and better,
and the communities keep growing. Cool kids in frontend world consider SPA as the
go-to solution for their projects. Is server-generated pages dead?

## Intro

One day, I came cross this great medium article
[Escaping the SPA Rabbit Hole with Turbolinks][esrh-link] which DHH gave
claps to. It's a long article, I read it thoroughly. It inspired me and
gave me a chance to reconsider the SPA hype and the good ol' Rails-way.
I also translated into Chinese and post at Ruby China, here's the
[link][rbcn-link].

## What I've learned

### SPA is a great way to make certain type of apps.
Imagining building a dashboard for a massive size system, or a web-based
chat app like WhatsApp or Slack, where you need to maintain very complex
frontend states, retrieve and send a large amount
of data, dynamically update UI frequently. PWA is probably an ideal option.
For example, a SPA using React for UI and Redux for state management, it works
especially well in this situation. Reusable components are DRY and it's unidirectional data flow
makes things a lot easier and cleaner than bi-directional data binding.
Of course, this is another big topic and out of the scope of this post, I
might write another post about it some other day, But here's a good
quote I want to share before I move on:
> Shared mutable state is the root of all evil.

SPA, if done right, can not only bring very pleasant user experience but
also make developers' life much easier. But keep one thing in mind, SPA
and tools behind it like, React, is invented by the brilliant team in
Facebook to deal with sophisticated problems they have in their big fancy
apps. The thing we need to find out is, are we going to have the same
type of problems in our apps?

### SPA is also an overkill for many other apps.
To be honest, IMHO SPA is an overkill for "most" apps, like blogs,
small-to-medium size E-commerce apps and other "simpler" app. The efforts you
input doesn't output an adequate amount of outcome. Let's put it this
way, you maintain 2 apps -- SPA and API server, rather than a single Rails
Monolith app, you spend maybe 1.5-2 times more of your time and amount of
code, which might as well introduce 1.5-2 times more bugs. But in the end, you
find out the initial loading time of your SPA is no faster than that
plain old Rails app. Could this be real? yes very likely. Benchmarks
shows the SPA is not necessary to load faster the first time. You may need
to make multiple API requests in your SPA on a single page rendering, on the
other hand in a Rails app, it's all done on the server while rendering
the page. To eliminate multiple requests, you might need to introduce
GraphQL into your app, which makes it's even more works to do.

### The modern Rails way and its pros and cons.
Turbolinks, SJR and Stimulus are 3 major components of the "Modern Rails",
with the new Rails UJS, developers now can get rid of jQuery entirely from
their project if desired(Don't get me wrong, I ain't against jQuery).
It's quite straight forward to include them in your app in a modern way
by using webpack, it looks like this:
{% highlight javascript %}
import * as ActiveStorage from "activestorage";
import Rails from 'rails-ujs';
import Turbolinks from 'turbolinks';
import { Application } from "stimulus";
import { definitionsFromContext } from "stimulus/webpack-helpers";
# ...

ActiveStorage.start();
Rails.start();
Turbolinks.start();

const application = Application.start();
const context = require.context("controllers", true, /.ts$/);
application.load(definitionsFromContext(context));
# ...
{% endhighlight %}
Now you have all the Rails tools you may need in your project without using sprokets.

Basecamp's web client is believed as a good example of utilization of
these 3 technologies and it runs pretty smooth and fast, I even asked my
no-techy friends, can they feel the difference between it and other SPA
apps, they can't really tell. I myself knew the differences are there but it's
just so insignificant to tell. Plus Rails has introduced outstanding
tools along it's releases to help making app running faster and more
efficient like Russian doll caching, etc. Other handy tools such as,
Active Storage, Active Jobs, Action Cable and upcoming Action Text enables
making a MVP or early stage app much faster. I think it is a good question
to ask if you're not sure whether to choose Rails as your tech stack: "Is
your app going to be more complex and serve more requests than basecamp's
, github's or shopify's?" Of course, it's no doubt there are Rails
pros in these companies, they do stuff like "tuning" the framework, etc.
to meet their business needs. But I have good experience
with the Rails community, there are always people to give you
constructive advices.

However, Rails could be a bottleneck for some tech-stack, it takes more
efforts to scale, if that's the case, congratulation, your app has a lot
users!

[esrh-link]: https://medium.com/@jmanrubia/escaping-the-spa-rabbit-hole-with-turbolinks-903f942bf52c
[rbcn-link]: https://ruby-china.org/topics/37531
