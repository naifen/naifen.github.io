---
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
layout: single
title:  "An Adventure in React.js & PWA"
date:   2018-09-28 22:10:23 -0700
categories: react.js pwa javascript web mobile
---
Making a PWA with React.js utilizing Ruby China's public API.

## Intro

A PWA made with React.js using RubyChina's APIs. It's built with mobile first in mind, responsive desktop UI & UX is also included. I used to use Redux a lot,
this time I want to see if I can use React's new Context API only.

## Motivation

As "rubyist"😄, I browse various of ruby sites on a daily basis to learn new stuffs. <https://ruby-china.org/> is one of them, which is the biggest ruby community in China.

The official Ruby China IOS client is awesome and I enjoyed it a lot. Recently I switched to an Android device and couldn't find a decent and up-to-date client. (tho the site itself is responsive, it doesn't feel like an app)

Since the ruby-china maintainers made great APIs, I figured why not roll my own client? It's 2018 now, PWA is a thing, React's became many folks go-to UI framework. So I decided to combine these two and give it a try.

My goal here is to focus on the mobile UI (Material Design <https://material-ui.com/> is my choice) & UX similar to the offcial IOS version. The app can be saved to desktop and work just like a native client(which is what a PWA suppose to be like). I'll try not to bloat it with too many npm packages, rather just keep it snappy, minimal and reliable.

## Demo

### Live Demo
[Ruby China PWA Demo][rbcnpwa-demo]

Open in your phone's browser and install to desktop. (*Tested on Chrome, Firefox for Andriod.*)

It can also be installed on Desktop with Chrome, see [this instruction][desktop-pwa-instruction]

### Screenshots

![splash-screen](/assets/images/splash-screen.jpg){:height="288px" width="144px"}
![home-screen](/assets/images/home-screen.jpg){:height="288px" width="144px"}
![topic-screen](/assets/images/topic-screen.jpg){:height="288px" width="144px"}
![login-screen](/assets/images/login-screen.jpg){:height="288px" width="144px"}
![signup-screen](/assets/images/signup-screen.jpg){:height="288px" width="144px"}
![desktop-home](/assets/images/desktop-home.png){:height="296px" width="214px"}
![desktop-topic](/assets/images/desktop-topic.png){:height="296px" width="214px"}
![startup](/assets/images/startup.gif){:height="576px" width="288px"}

## What I've learned

### * Think twice when navigating between screens with React Router in a PWA.

When using React-Router, switching between routes will casue components to mount,
unmount, this may cause unwanted user experience. For example, in the
following screenshot, when navigating from topics list page to a single topic page
then navigate back:

![re-mount](/assets/images/re-mount.gif){:height="576px" width="288px"}

The new screen is re-rendered, so the user will lose previouse scrolling location on
the screen, because navigate between React Router's caused ``TopicListContainer``
unmount and mount. Even if using Redux or the new React Context api to cache
result of json response from API requests, storing scrolling location could be
an *ugly* solution IMHO.

What I would like to do is to introduced a new layer of screen on top of
``TopicListContainer`` when user naviate to a single ``TopicItem``, and remove that
layer when user navigate back, this way the ``TopicListContainer`` will not be
mount and unmount.

### * Problem customizing Create React App's default service worker.

Latest CRA comes with Service Worker built in, which is required for PWA.
However customization instuctions are not clear documented. I tried to follow
methods introduced in this PR <https://github.com/facebook/create-react-app/pull/4169>, which didn't work for me, will keep exprimenting/

### * PWA, HTTPS and Mixed Contents.

"insecured" contents(eg, imgs served with non-https) on a page served with
https will casue firefox android PWA bounce out and fallback to the browser
<https://support.mozilla.org/en-US/kb/mixed-content-blocker-firefox-android>

### * React Router not firing up componentsDidMount in PWA?

Still investigating this issue.

### * React Context API can work with Redux.

Context API is a brilliant way to pass states down to children
components. But it's not meant to replace Redux, they can live together.

[rbcnpwa-demo]: https://ruby-china.surge.sh/
[desktop-pwa-instruction]: https://medium.com/@kennethrohde/progressive-web-apps-coming-to-all-chrome-platforms-80e31272e2a8

