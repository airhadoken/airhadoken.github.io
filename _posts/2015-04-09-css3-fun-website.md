---
title: "CSS3 Fun From My New Website"
author: "Bradley Momberger"
excerpt: "So I just finished revamping [my website](http://shinythingsnetwork.com/), on which I've been working with regularity for some two weeks.  The last time I did up my website was in 2003, and it was both the manifestation of a starry-eyed desire to get my friends into an art collective with me, and a fairly dry two-column layout I had built up in Dreamweaver.  A lot changes on the web in 12 years, and so I've tried to incorporate as much of the new hotness from CSS3 as I can, and even a touch of HTML5 goodness."
---

<style type="text/css">
.box {
  display: block;
  border: 1px dotted gray;
  clear: both;
  font-size: 2em;
  position: relative;
}
.box::before {
  content: "Demo";
  padding-left: 1ex;
}
.box p {
  text-align: center;
}
</style>

> "MTV has pushed the [electronic music] genre hard enough, devoting much of its 30 minutes of daily music programming to videos depicting pulsating, psychedelic shapes; after 16 years, MTV has finally completed its devolution into the Shiny Things Network" 
>
> --Steven Thompson, [A.V. Club review of The Chemical Brothers' *Dig Your Own Hole*](http://www.avclub.com/review/the-chemical-brothers-emdig-your-own-holeem-18095)

So I just finished revamping [my website](http://shinythingsnetwork.com/), on which I've been working with regularity for some two weeks.  The last time I did up my website was in 2003, and it was both the manifestation of a starry-eyed desire to get my friends into an art collective with me, and a fairly dry two-column layout I had built up in Dreamweaver.  A lot changes on the web in 12 years, and so I've tried to incorporate as much of the new hotness from CSS3 as I can, and even a touch of HTML5 goodness.

# 3D Transforms

CSS3 supports applying a transformation matrix to elements in three dimensions.  This concept is familiar to graphics programmers, and often to digital artists.

```css
  #marq, #holder {
    left: 47%;
    right: 0px;
    position: relative;
    perspective: 500px;
    height: 100%; /*transform-origin on child elements is bound by bounding box of parent */
    max-height: 600px;
  }

  .card {
    height: 600px;
    width: 400px;
    left: -200px;
    float: left;
    position: absolute;
    transform-origin: 50% 50% 0;
  }

  .card:nth-child(1) { transform: translate3d(0px, 0px, 0px); z-index: 20;}
  .card:nth-child(2) { transform: translate3d(100px, 0px, -50px); z-index: 19;}
  ...
  .card:nth-child(10) { transform: translate3d(900px, 0px, -450px); z-index: 11;}
```

Here, cards are direct children of the marquee.  It's important to note that some browsers (Firefox, mobile browsers) require that the direct parent of the transformed elements has a perspective defined; other browsers (Chrome) allow any ancestor.  I think I spent two days trying to figure out why my cards wouldn't scale commensurate with moving along the Z axis in Firefox; turns out that my `perspective` property was set on the parent of the marquee instead of the marquee itself.

<div class="box">
<div style="left: 47%;
    right: 0px;
    position: relative;
    perspective: 500px;
    height: 600px;
    transform: scale3d(0.5, 0.5, 1);">
  <div style="height: 600px;
    width: 400px;
    left: -200px;
    float: left;
    position: absolute;
    transform-origin: 50% 50% 0;
    transform: translate3d(0px, 0px, 0px);
    z-index: 20;
    background-color: red;
    opacity: 0.7">
    <p>In front</p>
    <p>translate3d(0, 0, 0)</p>
  </div>
  <div style="height: 600px;
    width: 400px;
    left: -200px;
    float: left;
    position: absolute;
    transform-origin: 50% 50% 0;
    transform: translate3d(100px, 0px, -50px);
    z-index: 19;
    background-color: blue;">
    <p style="color: white; margin-top: 50%;">Behind</p>
    <p style="color: white;">translate3d(100px, 0, -50px)</p>
  </div>
  <div style="clear:both; position: absolute; top: 98%">perspective: 500px</div>
</div>
</div>

# Transitions

```css
  .card {
    transition-property: opacity, left, transform;
    transition-duration: 0.4s;
  }

  .card .cardover {
    height: 100%;
    width: 100%;
    position: absolute;
    float: left;
    border-radius: 20px;
    background-color: rgb(51, 51, 51);
    transition-property: opacity;
    transition-duration: 0.4s;
  }
```

Transitions are an animation-lite in CSS, allowing for simple animation using much less CSS markup than the full animation description.  Every element can define which (numerically valued) CSS properties take time to change, how much time those changes take, and what timing function to use (ease in, ease out, ease in/out, linear, etc.).  To trigger a transition once defined requires only that the CSS property on the element changes, either directly in the element's `style` attribute or through a class or ordering change.

<div class="box">
<style type="text/css">
  #flipdemo {
    left: 47%;
    right: 0px;
    position: relative;
    perspective: 500px;
    height: 600px;
    transform: scale3d(0.5, 0.5, 1);
  }
  #flipdemo > .card {
    height: 600px;
    width: 400px;
    left: -200px;
    float: left;
    position: absolute;
    transform-origin: 50% 50% 0;
    background-color: red;
    transition-property: opacity, left, transform;
    transition-duration: 0.4s;
  }
  #flipdemo > .card:nth-child(1) {
    transform: translate3d(0px, 0px, 0px);
    z-index: 20;
  }
  #flipdemo > .card:nth-child(2) {
    transform: translate3d(100px, 0px, -50px);
    z-index: 19;
  }
  #flipdemo > .card:nth-child(3) {
    transform: translate3d(200px, 0px, -100px);
    z-index: 18;
  }
  #flipdemo > .card > .cardover {
    height: 100%;
    width: 100%;
    position: absolute;
    float: left;
    border-radius: 20px;
    background-color: rgb(51, 51, 51);
    transition-property: opacity;
    transition-duration: 0.4s;
  }
  #flipdemo > .card:nth-child(1) > .cardover { opacity: 0;}
  #flipdemo > .card:nth-child(2) > .cardover { opacity: 0.5;}
  #flipdemo > .card:nth-child(3) > .cardover { opacity: 0.8440417392452615;}
</style>
<div id="flipdemo" style="">
  <div class="card" style="background-color: red;">
    <div class="cardover">&nbsp</div>
    <p style="text-align: center; margin-top: 50%;">First</p>
  </div>
  <div class="card" style="background-color: blue;">
    <div class="cardover">&nbsp</div>
    <p style="text-align: center; margin-top: 50%; background-">Second</p>
  </div>
    <div class="card" style="background-color: green;">
    <div class="cardover">&nbsp</div>
    <p style="text-align: center; margin-top: 50%">Third</p>
  </div>
</div>
<script type="text/javascript">
function flip() {
  var el = document.getElementById("flipdemo");
  el.appendChild(el.firstElementChild);
  setTimeout(flip, 3000);
}
setTimeout(flip, 3000);
</script>
</div>

# Box Shadows

# Viewport-based units

# Flex Layouts

# Custom elements

# Text Shadows

# Gradients

```css
  body {
    background-color: rgb(51, 51, 51); /* compatibility -- ignored when the next property is recognized */
    background: linear-gradient(to bottom, rgb(34, 34, 34) 0%, rgb(68, 68, 68) 100%);
  }

  .card {
    background-color: yellow; /* compatibility -- ignored when the next property is recognized */
    background: linear-gradient(to bottom, #eeebda 0%,#f1ea96 100%);
  }
```

# Responsive Design

I'll admit, I did responsive the wrong way.  The right way these days is to design for a small mobile viewport first, then expand your design to desktop with media selectors.  I prefer working on desktop, reading the Web on desktop, etc.,
and so my site is desktop-first.  That said, I still want to throw a bone to people looking at my site on mobile browsers, even me if I'm out somewhere and want to show the site off to someone on my phone.



# Other Mobile Friendly Tweaks

One thing I did is add jQuery Mobile to my scripts.  In addition to automatically doing a small amount of layout changes to make pages mobile friendlier, it gave me easy "swipeleft" and "swiperight" events, so the user can flick between cards in the marquee on mobile (even though I've preserved the arrow buttons).

Another is the ["ideal viewport"](http://www.quirksmode.org/mobile/metaviewport/) I learned about at HTML5 DevConf last year from Peter-Paul Koch of QuirksMode.  There is one true way of setting up your mobile viewport to use the pixel size and zoom settings that the browser/device developers want you to use.  Because of the responsive design, I want the layout width and height to match the ideals (which is usually 1:1 with pixels, but e.g. 1:2 on a Retina display).

```html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
</head>
```