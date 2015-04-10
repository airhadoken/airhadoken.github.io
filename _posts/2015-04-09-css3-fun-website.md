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

CSS3 supports applying a transformation matrix to elements in three dimensions.  This concept is familiar to graphics programmers, and often to digital artists, but not very intuitive to me.  Fortunately there are convenience functions that generate transformation matrices from the primitive operations `translate`, `rotate`, and `scale`.  Each of these can be used in two dimensions or three, but three dimensions are always recommended to allow for hardware acceleration of transitions.  In the example below, a marquee-like element with id `marq` is given a `perspective` origin 500px above the view plane of the page, and each `card` child of the marquee is given a transformation on the X (side-to-side) and Z (towards/away from the viewer) axes according to its position in the element children.  

The `transform-origin` property below specifies that the center of the transformation is the center of the card element; it's not relevant for translate operations, and is shown here only for reference.

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

[Note: the following demo contains some Webkit corrections; Safari currently doesn't understand non-prefixed `transform` and `perspective` properties]

<div class="box">
<div style="left: 20%;
    right: 0px;
    position: relative;
    perspective: 500px;
    -webkit-perspective: 500px;
    height: 600px;
    transform: scale3d(0.5, 0.5, 1);
    -webkit-transform: scale3d(0.5, 0.5, 1);">
  <div style="height: 600px;
    width: 400px;
    left: -200px;
    float: left;
    position: absolute;
    transform-origin: 50% 50% 0;
    transform: translate3d(0px, 0px, 0px);
    -webkit-transform: translate3d(0px, 0px, 0px);
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
    -webkit-transform: translate3d(100px, 0px, -50px);
    z-index: 19;
    background-color: blue;">
    <p style="color: white; margin-top: 50%;">Behind</p>
    <p style="color: white;">translate3d(100px, 0, -50px)</p>
  </div>
  <div style="clear:both; position: absolute; top: 98%">perspective: 500px</div>
</div>
</div>

# Transitions

Transitions are an animation-lite in CSS, allowing for simple animation using much less CSS markup than the full animation description.  Every element can define which (numerically valued) CSS properties take time to change, how much time those changes take, and what timing function to use (ease in, ease out, ease in/out, linear, etc.).  To trigger a transition once defined requires only that the CSS property on the element changes, either directly in the element's `style` attribute or through a class or ordering change.

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
  .card:nth-child(1) .cardover { opacity: 0;}
  .card:nth-child(2) .cardover { opacity: 0.5;}
  .card:nth-child(3) .cardover { opacity: 0.8440417392452615;}
```

Setting the transform and opacity on the card and card overlay, respectively, creates a move-forward fade-in effect, shown below on a timed transform.  The script here is doing just one thing: re-attaching the first card to the end.  This is why the first/last card doesn't transition out like the others.  To get smoother effects where element restructuring is concerned requires some trickery.  The actual site uses a "holder" container sitting in front of the marquee to grab the first element and fade it out while the others move forward.

<div class="box">
<style type="text/css">
  #flipdemo {
    left: 20%;
    right: 0px;
    position: relative;
    perspective: 500px;
    -webkit-perspective: 500px;
    height: 600px;
    transform: scale3d(0.5, 0.5, 1);
    -webkit-transform: scale3d(0.5, 0.5, 1);
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
    -webkit-transform: translate3d(0px, 0px, 0px);
    z-index: 20;
  }
  #flipdemo > .card:nth-child(2) {
    transform: translate3d(100px, 0px, -50px);
    -webkit-transform: translate3d(100px, 0px, -50px);
    z-index: 19;
  }
  #flipdemo > .card:nth-child(3) {
    transform: translate3d(200px, 0px, -100px);
    -webkit-transform: translate3d(200px, 0px, -100px);
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
<script type="text/javascript">window.flipdemo = "flipdemo";</script>
</div>
<script type="text/javascript">
    function flip() {
      var el = document.getElementById(flipdemo);
      el.appendChild(el.firstElementChild);
      setTimeout(flip, 3000);
    }
    setTimeout(flip, 3000);
</script>

# Box Shadows

CSS3 allows for block elements to have any number of shadows with blur radii.  I kept this fairly simple for cards; my only desire from a design perspective was to enhance the visual notion that subsequent cards were behind the frontmost one.

```css
  .card {
    box-shadow: rgba(34, 34, 34, 0.7) 0px 0px 30px 30px;
  }
```

One unfortunate limitation of shadows is that they can only take solid colors for their color values; gradients are not supported.  Since the background on my site is a slight gradient, I had to just use a best-fit color to make it look like the shadow blends into the background property.

<div id="shadowdemo" class="box">
<style type="text/css">
  #shadowdemo .card {
    width: 400px;
    height: 400px;
    margin-left: auto;
    margin-right: auto;
    box-shadow: rgba(34, 34, 34, 0.7) 0px 0px 30px 30px;
    margin-bottom: 35px;
  }
</style>
<div class="card">Box shadow feathering over 30px</div>
</div>

# Text Shadows

The navigation arrows were given a glowy effect through composited text shadows, which function similar to box shadows but stem from the outline of the text. 

```css
  #ra, #la {
    text-shadow: 5px 0px 10px white,
                 0px 5px 10px white,
                 0px -5px 10px white;
  }
  
  #la {
    transform: scaleX(-1.0);
  }

  #ua {
    text-shadow: 5px 0px 10px white,
                 -5px 0px 10px white,
                 0px -5px 10px white;
  }
```

Note that to get something that truly looks like a glow effect, I needed to put several shadows on the text, offset to the right, top, and bottom for the right arrow (the left arrow is just the right one reversed with a transform); and the top, right, and left for the up arrow.

<div class="box" style="background-color: rgb(51, 51, 51); color: white">
<div style="text-align: center">
<span style="text-shadow: 5px 0px 10px white, 0px 5px 10px white, 0px -5px 10px white; font-size: 144pt;">&#x21f0;</span>
<span style="text-shadow: 5px 0px 10px white, -5px 0px 10px white, 0px -5px 10px white; font-size: 144pt;">&#x21E7;</span>
</div>
</div>


# Flex Layouts

Of all the spiffy new things provided by CSS3, flex layouts are easily the most important, and the most maddening.

The good:

* Flex is fabulous for automatically correcting spacing to get just the right look for an arbitrary number of elements in a container.  
* Flex removes the need to make everything float to wrap elements to the container size.
* Items in flex can **be reordered in the display without changing their document order** (It was a serious holy-crap moment when I learned this).
* Items in a flex container can define for themselves how much they should grow and shrink in spacing relative to other elements.
* Flex has sane and sensible options for aligning items along the flex direction *and* perpendicular to it.

The bad: 

* Flex is a serious hazard when working with images.
* Flex provides no support for stretching or shrinking text.
* Flex is new enough to still have little browser based gotchas.
* Problems with flex are usually solved with [massive capital investment in more flex boxes](https://www.youtube.com/watch?v=nEI19kJ5GfU).

```css
  .card .item {
    position: relative;
    float:left;
    width: 100%;
    height: 90%;
    top: 10%;
    display: flex;
    justify-content: space-around;
    flex-direction: column;
  }

  .card .item > header {
    border-bottom: 1px dotted rgb(51, 51, 51);
    position: absolute;
    top: -9%;
    left: 0; /* needed for Firefox */
    font-size: 1.4em;
    display: inline !important;
  }

  .card .item > * {
    display: inline-flex;
    flex-flow: inherit;
    justify-content: inherit;
    margin-left: 1%;
    margin-right: 1%;
    width: 98%;
    text-align: center;
    flex-grow: 2;
    flex-shrink: 2;
    overflow-y: hidden;
  }

  .card .item > *:last-child {
    overflow-y: visible;
  }
```

What's this?  Every child of my nice flex column is also itself a flex row? Why?

It turns out that this is the easiest way of making the bad parts of a flex layout go away, thus the gibe above about massive capital investment. In general, if a flex layout isn't doing the right thing with the content inside of it, the problem can be solved just by making the ornery content another, internal, flex layout.  Images get righted, centering and spacing look better, multiple items can be run across properly, etc.

Also, you'll notice that the header element in each card item is treated specially.  The rest of the flex layout is given an offset to compensate for the header being up top rather than evenly spaced with everything.

<div id="flexdemo" class="box" style="height: 500px">
<style type="text/css">

 #flexdemo .card {
  width: 40%;
  height: 400px;
  border: 1px dotted blue;
  float: left;
  margin-left: 5%;
 }

 #flexdemo .card .item {
    position: relative;
    width: 100%;
    height: 90%;
    top: 10%;
    display: flex;
    justify-content: space-around;
    flex-direction: column;
    font-size: 14pt;
  }

 #flexdemo .card .item > * {
    display: inline-flex;
    flex-flow: inherit;
    justify-content: inherit;
    margin-left: 1%;
    margin-right: 1%;
    width: 98%;
    text-align: center;
    flex-grow: 2;
    flex-shrink: 2;
    overflow-y: hidden;
  }

  #flexdemo .card .item > header {
    border-bottom: 1px dotted rgb(51, 51, 51);
    position: absolute;
    top: -9%;
    left: 0; /* needed for Firefox */
    font-size: 1.4em;
    display: inline !important;
  }
</style>
<br>
<div class="card">
<div class="item">
  <p> Column layout!</p>
  <p> even spacing </p>
  <p> When text is very long, word wrap wraps correctly, but be careful not to shrink the flex container too much. </p>
  <header>Flex header (absolute)</header>
</div>
</div>
<div class="card">
<div class="item" style="flex-direction: row">
  <header>Flex header (absolute)</header>
  <p> Row layout!</p>
  <p> spacing is even here too </p>
  <p> When text is very long, word wrap wraps correctly, but be careful not to shrink the flex container too much. </p>
</div>
</div>
<p class="clear: both">&nbsp;</p>
</div>

### Flex gotchas

One very important thing to know about flex is that the spacing is implemented by auto-adjusting *margins*, so when overrides are needed, the margins along the flex direction (top/bottom for column, left/right for row) are the targets.  We see this in the responsive CSS for small screens (more on this later):

```css
  @media all and (max-height: 650px) {
    .card .item > p {
      flex-shrink: 0;
      margin-bottom: 2px;
      margin-top: 2px;
    }
  }
```

Above, where it is stated that images are a hazard in flex layouts, this is due to the flex rendering in some browsers (Mozilla) only shrinking the image in one dimension while leaving the other its full height/width.  This can be mitigated by wrapping an image in a block element so that it's not a direct child of a flex layout.  As an unexpected fringe benefit of this, the image is usually scaled proportionally.  The artwork section of my site has entries like this:

```html
  <div class="item">
    <header>"Lamp on Table"</header>
    <div class="noshrink"><img data-src="./site-images/lampbw.png" /></div>
    <p>Charcoal on paper, 2001</p>
  </div>
```

The `noshrink` class contains a couple more embellishments to ensure that my images are displayed correctly.

```css
  .card .item .noshrink {
    flex-shrink: 1;
    flex-flow: row;
    align-items: center;
    overflow: hidden;
  }
```

Finally, because font sizes don't scale with flex layouts, it can become quite annoying making sure that the full text of textual elements shows up in places where a flex is used.  Thus the overflow-y being carefully controlled in places, and other element styles being wrapped up in responsive media queries (see Responsive Design below).

# The `initial` Property Value

This is something that should have been in CSS long ago.  It's a "reset button" for properties, that just says to an element type "you know all those ways I've been tweaking and manipulating you throughout this document? Well here you're free to just be yourself and do what comes naturally."  It prevents the need to know the spec details of each element, like its display type or the browser-defined text color, when making exceptions to a design rule in places.  In the CSS example below, a paragraph is treated like a paragraph in card items, not like an inline flex like everything else inside a card item is:

```css
  .card .item > p {
    display: initial;
  }
```

# Custom elements

I'm of two minds about custom elements, as far as their declaractive power goes.  On one hand, unlike a class designation, they can't be composited together.  On the other hand, when setting landmarks on a page as a sort of special container, they look much better in the HTML markup.  I had a few false starts in the layout where I tried doing certain items as custom elements, then pulled back and made them classed divs.  One that did make it through to the end is the `two-by-two` element, which makes a layout grid of two divs containing two other elements.

Note that according to the HTML5 spec, a custom element should always have at least one hyphen in the tag name.  Thus, `card` is not an appropiate custom tag name, but `marq-card` is OK.

```css
  .card .item two-by-two > div {
    display:flex;
    flex-flow: row;
    align-items: baseline;
  }

  .card .item two-by-two > div > * {
    width: 49%; /* implicit in Webkit, needs to be explicit in Moz */
  }

  .card .item two-by-two > div + div {
    align-items: initial;
  }
```

```html
  <div class="card">
    <div class="item">
      <two-by-two>
        <div>
          <div>(0, 0)</div>
          <span>(0, 1)</span>
        </div>
        <div>
          <p>(1, 0)</p>
          <section>(1, 1)</section>
        </div>
      </two-by-two>
    </div>
  </div>
```

So we get a nice grid in a flexbox.  Below in the demo, the card (dotted blue border) is a column-oriented flexbox, while the grid elements (dotted red border), are elements in the flex rows making up the 2x2 grid.  Having 49% width enforced ensures that the rows don't wrap to the next line in Mozilla, which is a bit touchier with flexboxes.  The `section` element at (1, 1) preserves its natural sizing differences from the other elements.

<div class="box" id="griddemo">
<style type="text/css">
  #griddemo .card > .item {
    padding-left: 1%;
    padding-right: 1%;
  }

  #griddemo two-by-two > div {
    display:flex;
    flex-flow: row;
    align-items: baseline;
  }

  #griddemo two-by-two > div > * {
    width: 49%; /* implicit in Webkit, needs to be explicit in Moz */
    text-align: center;
    border: 1px dotted red;
  }

  #griddemo two-by-two > div + div {
    align-items: initial;
  }

  #griddemo .card {
    border: 1px dotted blue;
    width: 400px;
    height: 400px;
    display: flex;
    flex-flow: column nowrap;
    justify-content: space-around;
    margin-left: auto;
    margin-right: auto;
  }
</style>

  <div class="card">
    <div class="item">
      <two-by-two>
        <div>
          <div>(0, 0)</div>
          <span>(0, 1)</span>
        </div>
        <div>
          <p>(1, 0)</p>
          <section>(1, 1)</section>
        </div>
      </two-by-two>
    </div>
  </div>
</div>

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

The effect of these gradients is subtle, but it makes the elements involved (marquee cards and the page background) look a little less flat and a little more textured.  For browsers that don't support gradients, a simple hack is to specify the color for compatibility, then override it with the gradient.

<div id="graddemo" class="box" style="color: white; background: linear-gradient(to bottom, rgb(34, 34, 34) 0%, rgb(68, 68, 68) 100%);">
<style type="text/css">
  #graddemo .card {
    width: 400px;
    height: 400px;
    margin-left: auto;
    margin-right: auto;
    background: linear-gradient(to bottom, #eeebda 0%,#f1ea96 100%);
    color: black;
  }
</style>
<div class="card">linear gradient<br>top #eeebda<br>bottom #f1ea96</div>
<p>linear gradient<br>top rgb(34, 34, 34), bottom rgb(68, 68, 68)</p>
</div>

# Responsive Design

I did responsive the wrong way.  The right way these days is to design for a small mobile viewport first, then expand your design to desktop with specialized large-viewport properties.  I prefer working on desktop, reading the Web on desktop, etc.,
and so my site is desktop-first.  That said, I still want to throw a bone to people looking at my site on mobile browsers, even me if I'm out somewhere and want to show the site off to someone on my phone.

The primary component of responsive design is the width and height properties in CSS media queries.  Media type selectors themselves are part of the CSS2 standard from 2004, and were a neat way of making print-friendly versions of a Web page without needing to serve a completely separate document.  With the arrival of CSS3, the media query, combining the type selector with any number of media feature selectors, expanded on this idea such that different screen sizes could render a page appropriate to their respective widths and heights.

```css
@media all and (max-width: 950px) {
    #ra, #la, #ua {
      position:relative;
      font-size: 48pt;
      transform: scaleY(0.5);
      right: auto;
    }
    #la {
      transform: scale3d(-1.0, 0.5, 1);
      clear: left;
      float: left;
    }
    #ua {
      margin-top: 15%;
      left: -98px;
      margin-left: 17.6vw;
      margin-right: -50vw;
    }
    #ra {
      float: right;
      right: 63vw;
    }
    #marq, #holder {
      left: 35vw;
    }
    .card {
      max-width: 63vw;
      left: 0;
      box-shadow: rgba(51, 51, 51, 0.8) 18px 20px 24px 18px;
    }

    .card header {
      font-size: 1.2em;
    }

    #info {
      font-size: 2.8vw;
      margin-left: 0;
      margin-top: 0;
    }
  }
```

Some things of note here: the box shadow for cards is adjusted so as to not cover the top or left, where info text and paging buttons may be placed depending on orientation; navigation arrows are rehomed and resized; and the spatial relation between the main elements has changed.

### Viewport-based units

A huge boon to trying to do responsive design for smaller and rather variable screens is the ability to tie the size of certain elements to the layout dimensions of the viewport.  CSS3 has four "viewport" units now, each one representing one percent of the appropriate dimension (`vw` => width, `vh` => height, `vmin` => minimum of width and height, `vmax` => maximum idem).

In the CSS example above (for a landscape orientation), font sizes and element positioning are adjusted such that:

* the info text for the site is adjusted to always be just under 35% of the width (the 2.8vw size is the result of trial and error)
* the marquee takes up the rightmost 65% of the viewport width (63% of which is used for the actual card width).
* The right arrow is placed just left of the marquee, and the up arrow is placed centered between the left and right arrows.  The negative left margin ensures that there is no line break before the right arrow, while the `left` and `margin-left` properties interplay in such a way to preserve the centering no matter the width.

# Other Mobile Friendly Tweaks

One thing I did is add jQuery Mobile to my scripts.  In addition to automatically doing a small amount of layout changes to make pages mobile friendlier, it gave me easy `"swipeleft"` and `"swiperight"` events, so the user can flick between cards in the marquee on mobile (though I've also preserved the arrow buttons).

Another is the ["ideal viewport"](http://www.quirksmode.org/mobile/metaviewport/) I learned about at HTML5 DevConf last year from Peter-Paul Koch of QuirksMode.  There is one true way of setting up your mobile viewport to use the pixel size and zoom settings that the browser/device developers want you to use.  Because of the responsive design, I want the layout width and height to match the ideals (which is usually 1:1 with pixels, but e.g. 1:2 on a Retina display).

```html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
</head>
```
# Conclusion

CSS3 is a solid tool for front end devlopers who want to get their hands into design.  It's espeically brilliant now that most of the vendor prefix nonsense has gone away (except for stock Webkit, which means that my site is explicitly not supported on desktop Safari).  Compared to previous efforts to design pages and sites, CSS3 provides a sizable expansion in expressive power, though few things are made explicitly easier than before.  Importantly, and a huge relief to front end developers, stuff that used to be the exclusive domain of scripting (like animation and flexible layout) is being pushed off into declarative styles.

I constructed the design of my website without the aid of an IDE for Web design, just a lot of live work from the Chrome developer tools.  I've had the vision of my website kicking around in my head for quite a while, and it feels great to see it in action.  I learned a ton about what's available for Web design now and I'm excited about putting my new knowledge into action again, massive recurrence of flexboxes included.