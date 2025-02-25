---
layout: post
title: Rendering Performance
subhead: |
  Users notice if sites and apps don't run well, so optimizing rendering performance is crucial!
authors:
  - paullewis
date: 2015-03-20
updated: 2018-08-17 
description: |
  Users notice if sites and apps don't run well, so optimizing rendering performance is crucial!
tags:
  - blog # blog is a required tag for the article to show up in the blog.

---

Users of today’s web
[expect that the pages they visit will be interactive and smooth](https://paul.kinlan.me/what-news-readers-want/)
and that’s where you need to increasingly focus your time and effort. Pages
should not only load quickly, but also run well; scrolling should be
stick-to-finger fast, and animations and interactions should be silky smooth.

To write performant sites and apps you need to understand how HTML, JavaScript and CSS is handled by the browser, and ensure that the code you write (and the other 3rd party code you include) runs as efficiently as possible.

## 60fps and Device Refresh Rates

<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/hjgQvYAu7IMrwEkmLyT9.jpg", alt="User interacting with a website.", width="612", height="486" %}
</figure>

Most devices today refresh their screens **60 times a second**. If there’s
an animation or transition running, or the user is scrolling the pages, the
browser needs to match the device’s refresh rate and put up 1 new picture, or
frame, for each of those screen refreshes.


Each of those frames has a budget of just over 16ms (1 second / 60 = 16.66ms).
In reality, however, the browser has housekeeping work to do, so all of your
work needs to be completed inside **10ms**. When you fail to meet this
budget the frame rate drops, and the content judders on screen. This is often
referred to as **jank**, and it negatively impacts the user's experience.

## The pixel pipeline

There are five major areas that you need to know about and be mindful of when
you work. They are areas you have the most control over, and key points in the
pixels-to-screen pipeline:

<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/gfQC4uOnIbLVtkdvbSY4.jpg", alt="The full pixel pipeline.", width="800", height="122" %}
</figure>

* **JavaScript**. Typically JavaScript is used to handle work that will result in visual changes, whether it’s jQuery’s `animate` function, sorting a data set, or adding DOM elements to the page. It doesn’t have to be JavaScript that triggers a visual change, though: CSS Animations, Transitions, and the Web Animations API are also commonly used.
* **Style calculations**. This is the process of figuring out which CSS rules apply to which elements based on matching selectors, for example, `.headline` or `.nav > .nav__item`. From there, once rules are known, they are applied and the final styles for each element are calculated.
* **Layout**. Once the browser knows which rules apply to an element it can begin to calculate how much space it takes up and where it is on screen. The web’s layout model means that one element can affect others, for example the width of the `<body>` element typically affects its children’s widths and so on all the way up and down the tree, so the process can be quite involved for the browser.
* **Paint**. Painting is the process of filling in pixels. It involves drawing out text, colors, images, borders, and shadows, essentially every visual part of the elements. The drawing is typically done onto multiple surfaces, often called layers.
* **Compositing**. Since the parts of the page were drawn into potentially multiple layers they need to be drawn to the screen in the correct order so that the page renders correctly. This is especially important for elements that overlap another, since a mistake could result in one element appearing over the top of another incorrectly.

Each of these parts of the pipeline represents an opportunity to introduce jank, so it's important to understand exactly what parts of the pipeline your code triggers.

Sometimes you may hear the term "rasterize" used in conjunction with paint.
This is because painting is actually two tasks: 1) creating a list of draw
calls, and 2) filling in the pixels.

The latter is called "rasterization" and so whenever you see paint records in
DevTools, you should think of it as including rasterization. (In some
architectures creating the list of draw calls and rasterizing are done in
different threads, but that isn't something under developer control.)

You won’t always necessarily touch every part of the pipeline on every frame.
In fact, there are three ways the pipeline _normally_ plays out for a given
frame when you make a visual change, either with JavaScript, CSS, or Web
Animations:

### 1. JS / CSS > Style > Layout > Paint > Composite

<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/gfQC4uOnIbLVtkdvbSY4.jpg", alt="The full pixel pipeline.", width="800", height="122" %}
</figure>

If you change a “layout” property, so that’s one that changes an element’s
geometry, like its width, height, or its position with left or top, the browser
will have to check all the other elements and “reflow” the page. Any affected
areas will need to be repainted, and the final painted elements will need to be
composited back together.

### 2. JS / CSS > Style > Paint > Composite

<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/aD7UKMbegndQijj36PHi.jpg", alt="The  pixel pipeline without layout.", width="800", height="122" %}
</figure>

If you changed a “paint only” property, like a background image, text color, or
shadows, in other words one that does not affect the layout of the page, then the browser
skips layout, but it will still do paint.

### 3. JS / CSS > Style > Composite


<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/bfBPdciP9OSYPFI67xiY.jpg", alt="The pixel pipeline without layout or paint.", width="800", height="122" %}
</figure>

If you change a property that requires neither layout nor paint, and the
browser jumps to just do compositing.

This final version is the cheapest and most desirable for high pressure points
in an app's lifecycle, like animations or scrolling.

Note: If you want to know which of the three versions above changing any given CSS property will trigger head to [CSS Triggers](https://csstriggers.com). And if you want the fast track to high performance animations, read the section on [changing compositor-only properties](stick-to-compositor-only-properties-and-manage-layer-count).

Performance is the art of avoiding work, and making any work you do as
efficient as possible. In many cases it's about working with the browser, not
against it. It’s worth bearing in mind that the work listed above in the
pipeline differ in terms of computational cost; some tasks are more expensive
than others!

Let’s take a dive into the different parts of the pipeline. We’ll take a look
at the common issues, as well how to diagnose and fix them.

## Browser Rendering Optimizations
<a href="https://www.udacity.com/course/browser-rendering-optimization--ud860">
<figure>
{% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/BRtqtw4gW9q2VBcQC0aW.jpg", alt="Udacity course screenshot", width="360", height="220" %}
</figure>
</a>

Performance matters to users. Web developers need to build apps that react
quickly and render smoothly. Google performance guru Paul Lewis is here
to help you destroy jank and create web apps that maintain 60 frames per
second performance. You'll leave this course with the tools you need to
profile apps and identify the causes of jank. You'll explore the browser's
rendering pipeline and uncover patterns that make it easy to build
performant apps.

This is a free course offered through [Udacity](https://www.udacity.com)
    

[Take Course](https://www.udacity.com/course/browser-rendering-optimization--ud860)

