# Looking back at Retrollect.

## Synopsis

This is a list of the dos and don'ts of building a cross platform HTML5 application as learned during the development of the Iphone and Android application [Retrollect](http://retrollectapp.com/).

Topics that will be touched on:

* CSS transforms and gotchas
* What to expect when using experimental features (-webkit prefix)
* Optimizing for mobile
* Debugging on mobile
* Images and reflow
* Animations and touch move events
* Writing custom JavaScript API interfaces for getting data into and out of native device APIs

### About Retrollect

> Retrollect is the free mobile app that lets you assemble the highlights of your experiences and create a visual mash-up of your life!

Retrollect allows you to create slide shows containing 8 pieces of content from your phone, Twitter account, Facebook status updates, Instagram, or your phone's camera in a visually interesting way. Retrollect uses the aesthetics of the vintage View-Master to add a sense of nostalgia to your collections of content.

Retrollect does this using a codebase mostly comprised of JavaScript, HTML, and CSS.

## Why HTML?

So you are going to develop a mobile application and are weighing your options, there is **Objective-c** combined with the cocoa APIs for iOS, **Java** and Android's API system, there are **JavaScript** to native code compilers like Titanium Appcelerator, and then there is the option to build straight **HTML, JavaScript, and CSS** using normal web app practices or an embedded web view.

We've all heard a lot about the shortcomings of HTML5 as a replacement for native code. There's the tired argument about [HTML5 vs Native Apps][html5-vs-native-apps] and how HTML5 not good enough yet. I tend to agree and it frustrates me that my web apps aren't as fast or responsive as native code would be, but things are moving forward faster than ever. Looking at a lot of the experiments and [absolutely amazing things people are pushing with it](#examples-of-the-good-stuff), I get really inspired. I think what is really needed is an objective look at what could be done with it, what has been done, and what needs to happen to help push the spec and it's implementations across vendors forward.

The question is how do we do this without absolutely loosing our minds?

### HTML5 is a great choice if:

* You are fairly proficient with HTML, CSS, and JavaScript.
* You want to be able to deploy bug fixes etc without going through an app store
* The stuff in the HTML5 spec like Web Workers, placeholders, Geoloaction, and all that is super sexy to you.

### HTML5 is a not so great if:

* You absolutely **NEED** precise native look and feel in your app
* You have extremely complex UI

# Do: Make fluid layouts

"But the iphone's screen size is a set size!", you say?

Just make it fluid man, the point of using HTML5 in the mobile context is to not be tied to any platform beyond a browser. Browsers (even embedded web-views) can be different sizes, think iPhone, iPad, most Android handsets have different screen dimensions.

Making your design fluid (grow to fit any width and height) will save you a ton of headaches when putting your app on more than one device.

# Don't: Use `img` tags, actually don't use anything that renders images either

**Mobile webkit HATES, HATES, HATES images**. They are extremely slow to render and manipulate.

Also avoid:

* `-webkit-gradient`: It paints a bitmap on the viewport and is nearly the same as using images as far as the rendering engine is concerned. <sup>[Making an iPad HTML5 App & making it really fast][1]</sup>
* `text-shadow`, `box-shadow`, `opacity`, and alpha background colors also behave similar to using images, these should be avoided wherever performance is a concern.

# Do: Use semantically correct but _minimal_ html

Good:

    <div class="list">
      <a href="#">Item 1</a>
      <a href="#">Item 2</a>
      <a href="#">Item 3</a>
    </div>

Bad:

    <div id="wrapper">
      <div id="inner-wrapper">
        <ul class="list">
          <li class="list-item">
            <a href="#">Item 1</a>
          </li>

          <li class="list-item">
            <a href="#">Item 2</a>
          </li>

          <li class="list-item">
            <a href="#">Item 3</a>
          </li>
        </ul>
      </div>
    </div>

Fewer elements means less work for the browser, and less confusion for your development team (this topic could be a whole separate presentation on managing complexity in CSS and Markup.)

Using a minimal amount of markup is really important when using css transforms and expecting reasonably responsive animations.

# Don't use `position: fixed;`

It's not as broken on Mobile Safari but should still be avoided, it really blows up on android. Caused weird errors with text fields on Android. [Fixed positioning in Mobile Safari &raquo;][2]

# Do: Avoid reflows

Group DOM Queries and manipulation into separate chunks, do all your manipulation at once instead of spread out.

[Quote from Stoyan Stefanov on phpied.com:][3]

> Anything that changes input information used to construct the rendering tree can cause a repaint or a reflow, for example:
>
> * Adding, removing, updating DOM nodes
> * Hiding a DOM node with display: none (reflow and repaint) or visibility: hidden (repaint only, because no geometry changes)
> * Moving, animating a DOM node on the page
> * Adding a stylesheet, tweaking style properties
> * User action such as resizing the window, changing the font size, or (oh, OMG, no!) scrolling

## These guys are usually the culprits on elements:

clientHeight, clientLeft, clientTop, clientWidth, focus(), getBoundingClientRect(), getClientRects(), innerText, offsetHeight, offsetLeft, offsetParent, offsetTop, offsetWidth, outerText, scrollByLines(), scrollByPages(), scrollHeight, scrollIntoView(), scrollIntoViewIfNeeded(), scrollLeft, scrollTop, scrollWidth

## See also:

* [When does JavaScript trigger reflows and rendering?][4]
* [Don’t Be Trigger Happy; How To Not Trigger Layout][5]
* [How (not) to trigger a layout in WebKit][6]

# Don't: Try to make html behave like native code

Im looking at you pinned header and footer with a scrolling list view in between. While we were able to implement an acceptable solution to this common native UI pattern it still doesn't feel right, we got most of the way in a short amount of time but spending time on the rest proved to be nearly counterproductive.

The Retrollect team spent a lot of time trying to force this behavior into our HTML5 app with a bunch of hacks which ended up being a hit to performance or backed us into other problems like circumventing Maobile Safari's rendering optimizations.

Things that are difficult to translate to into pure HTML5 implementations:

* Scrolling
* Swipe to Delete
* Responsive accelerometer based UI

# Do: Write tests

This is crucial and can actually speed up your development even though it will feel slow at first. It is also important to write tests that are more than testing internal javascript APIs between objects. You should be doing BDD here, you want to make sure when a user clicks a thing that something happens - write a test for that.

good test:

    describe('foo.hasContent()', function(){
      when('the slots are empty', function(){
        it('should NOT have content', function(){
          expect(disc.hasContent()).toBe(false);
        });
      });

      when('the slots are NOT empty', function(){
        it('should have content', function(){
          expect(disc.hasContent()).toBe(true);
        });
      });
    });

bad test:

    describe('foo.hasContent()', function(){
      when('the slots are empty', function(){
        it('should should call .length', function(){
          spyOn(disc._hiddenContent, 'length');

          expect(disc._hiddenContent.length).toHaveBeenCalled();
        });
      });
    });


# Don't: Assume you know anything or that anything actually works

You got to play it like you are new to the game. What is going on in these mobile browsers if far different than anything I have ever experienced. For example:

* the autofocus attribute on iputs started crashing mobile safari for apparently no reason. It took about 2 hours to find out removing the autofucs attribute from our login form fixed the problem. Why? Who knows...
* When initially launching our app setInnerHTML would not work, it says it did but no html was getting set. We had to keep calling `setInnerHTML` in a timer that checked that the html was actually added. [See "Problems with Safari and innerHTML"][7]

Examine the problems and explore its solution like a scientist. Make guesses, apply them, did it work? nope do it again..

# Do: Write lots of docs about all the work arounds you will end up using

I am a huge fan of docco, take a look at these guys:

without docs:

    WebSlideView.prototype._appendCss = function (callback) {
      utils.requireType(callback, 'function');

      var sheetsToLoad = [
        '/stylesheets/slide.css',
        '/stylesheets/slide_edit.css',
        '/stylesheets/slide_create.css',
        '/stylesheets/web-slide.css',
        '/stylesheets/ie-slide.css',
        '/assets/' + defaults.version + '/themes/stylesheets/slide-themes.css'
      ];

      var sheetsLoaded = 0;

      var wrappedCallback = function (data) {
        $('head').append(
          '<style type="text/css" class="webSlideCss">' + data + '</style>'
        );

        sheetsLoaded++;

        if (sheetsLoaded === sheetsToLoad.length) { callback(); }
      };

      $(sheetsToLoad).each(function (i, sheetName) {
        $.get(sheetName, wrappedCallback);
      });
    };

with docs: http://www.cl.ly/280Q0y3P2t2R0b0t1h0m

# Do: Debug as much as possible in a desktop webkit browser (Google Chrome, or Safari)

* debugging in a mobile browser is painstakingly slow and not accurate (no exceptions etc.).


# Android Specifics

The following is more Android specific - this is different than html5 feature detection. Android has these features, they are just busted.

Depending on the complexity of your app, peppered if statements might become necessary:

    if (navigator.userAgent.indexOf('Android') != -1) {
      // ...
    }

# Don't: Use 3D Transforms on Android

3D Transforms are hardware accelerated on iOS and have the bugs worked out for the most part, it is ideal the use them on iOS devices. But they don't work very well on Android, you should use normal transforms instead.

# Do: Pay attention to heap size on Android

We had a haunting issue with base64 encoding images and it turned out to be limits on the heap size, [take a look at this message board convo][8].

# Don't: assume XHRs to behave the same on mobile as it does on the handsets

# El Fin

## Further Reading

* [Mistakes Made Building Netflix for iPhone][9]
* [The Problem with Sencha Touch and PhoneGap][10]
* [Dive Into HTML5][11]
* [HTML5 for Web Designers][12]
* [CSS3 For Web Designers][13]

<a name="examples-of-the-good-stuff" />

### Neat stuff done with HTML5

* http://www.effectgames.com/demos/canvascycle/
* http://sebleedelisle.com/2011/04/multi-touch-game-controller-in-javascripthtml5-for-ipad/
* http://9elements.com/io/projects/html5/canvas/
* http://popcornjs.org/
* http://www.chromeexperiments.com/
* http://pistolslut.com/

[later]: http://later.com "No time..."
[later-image]: http://1.bp.blogspot.com/_R2MTjgtcbf4/TEPGVl-ZjfI/AAAAAAAAB7I/JNrYjDRtxkU/s400/Sabertooth%2BZombie%2BRainfest.jpg "No time to get images"
[html5-vs-native-apps]: http://techcrunch.com/2011/02/09/html5-versus-native-apps/ "HTML5 Is An Oncoming Train, But Native App Development Is An Oncoming Rocket Ship"
[1]: http://mir.aculo.us/2010/06/04/making-an-ipad-html5-app-making-it-really-fast/
[2]: http://doctyper.com/archives/200808/fixed-positioning-on-mobile-safari/
[3]: http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/
[4]: http://mir.aculo.us/2010/08/17/when-does-javascript-trigger-reflows-and-rendering/
[5]: http://functionsource.com/post/dont-be-trigger-happy-how-to-not-trigger-layout
[6]: http://gent.ilcore.com/2011/03/how-not-to-trigger-layout-in-webkit.html
[7]: http://blog.johnmckerrell.com/2007/03/07/problems-with-safari-and-innerhtml/
[8]: http://forum.cyanogenmod.com/topic/3728-whats-your-vm-heap-size/
[9]: http://www.readwriteweb.com/mobile/2011/03/sxsw-mistakes-made-building-netflix-for-iphone-and-source-code.php
[10]: http://www.moneytoolkit.com/2011/05/the-problem-with-sencha-touch-and-phonegap/
[11]: http://diveintohtml5.org/
[12]: http://www.abookapart.com/products/html5-for-web-designers
[13]: http://www.abookapart.com/products/css3-for-web-designers
