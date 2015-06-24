<article>
I remember the early days of JavaScript where you needed a simple function for
just about everything because the browser vendors implemented features 
differently, and not just edge features, basic features, like`addEventListener`
`attachEvent`.  Times have changed but there are still a few functions each
developer should have in their arsenal, for performance for functional ease 
purposes.

## [`debounce`][1]

The debounce function can be a game-changer when it comes to event-fueled
performance.  If you aren't using a debouncing function with a`scroll`, 
`resize`, `key*` event, you're probably doing it wrong.  Here's a `debounce`
function to keep your code efficient:

    // Returns a function, that, as long as it continues to be invoked, will not
    // be triggered. The function will be called after it stops being called for
    // N milliseconds. If `immediate` is passed, trigger the function on the
    // leading edge, instead of the trailing.
    function debounce(func, wait, immediate) {
    	var timeout;
    	return function() {
    		var context = this, args = arguments;
    		var later = function() {
    			timeout = null;
    			if (!immediate) func.apply(context, args);
    		};
    		var callNow = immediate && !timeout;
    		clearTimeout(timeout);
    		timeout = setTimeout(later, wait);
    		if (callNow) func.apply(context, args);
    	};
    };
    
    // Usage
    var myEfficientFn = debounce(function() {
    	// All the taxing stuff you do
    }, 250);
    window.addEventListener('resize', myEfficientFn);
    

The `debounce` function will not allow a callback to be used more than once
per given time frame.  This is especially important when assigning a callback 
function to frequently-firing events.

## [`poll`][2]

As I mentioned with the `debounce` function, sometimes you don't get to plug
into an event to signify a desired state -- if the event doesn't exist, you need
to check for your desired state at intervals:

    function poll(fn, callback, errback, timeout, interval) {
        var endTime = Number(new Date()) + (timeout || 2000);
        interval = interval || 100;
    
        (function p() {
                // If the condition is met, we're done! 
                if(fn()) {
                    callback();
                }
                // If the condition isn't met but the timeout hasn't elapsed, go again
                else if (Number(new Date())  0;
        },
        function() {
            // Done, success callback
        },
        function() {
            // Error, failure callback
        }
    );
    

Polling has long been useful on the web and will continue to be in the future
!

## [`once`][3]

There are times when you prefer a given functionality only happen once, similar
to the way you'd use an`onload` event.  This code provides you said
functionality:

    function once(fn, context) { 
    	var result;
    
    	return function() { 
    		if(fn) {
    			result = fn.apply(context || this, arguments);
    			fn = null;
    		}
    
    		return result;
    	};
    }
    
    // Usage
    var canOnlyFireOnce = once(function() {
    	console.log('Fired!');
    });
    
    canOnlyFireOnce(); // "Fired!"
    canOnlyFireOnce(); // nada
    

The `once` function ensures a given function can only be called once, thus
prevent duplicate initialization!

## [`getAbsoluteUrl`][4]

Getting an absolute URL from a variable string isn't as easy as you think.  
There's the
 `URL` constructor but it can act up if you don't provide the required
arguments (which sometimes you can't).  Here's a suave trick for getting an 
absolute URL from and string input:

    var getAbsoluteUrl = (function() {
    	var a;
    
    	return function(url) {
    		if(!a) a = document.createElement('a');
    		a.href = url;
    
    		return a.href;
    	};
    })();
    
    // Usage
    getAbsoluteUrl('/something'); // http://davidwalsh.name/something
    

The "burn" element `href` handles and URL nonsense for you, providing a
reliable absolute URL in return.

## [`isNative`][5]

Knowing if a given function is native or not can signal if you're willing to
override it.  This handy code can give you the answer:

    ;(function() {
    
      // Used to resolve the internal `[[Class]]` of values
      var toString = Object.prototype.toString;
      
      // Used to resolve the decompiled source of functions
      var fnToString = Function.prototype.toString;
      
      // Used to detect host constructors (Safari > 4; really typed array specific)
      var reHostCtor = /^\[object .+?Constructor\]$/;
    
      // Compile a regexp using a common native method as a template.
      // We chose `Object#toString` because there's a good chance it is not being mucked with.
      var reNative = RegExp('^' +
        // Coerce `Object#toString` to a string
        String(toString)
        // Escape any special regexp characters
        .replace(/[.*+?^${}()|[\]\/\\]/g, '\\$&')
        // Replace mentions of `toString` with `.*?` to keep the template generic.
        // Replace thing like `for ...` to support environments like Rhino which add extra info
        // such as method arity.
        .replace(/toString|(function).*?(?=\\\()| for .+?(?=\\\])/g, '$1.*?') + '$'
      );
      
      function isNative(value) {
        var type = typeof value;
        return type == 'function'
          // Use `Function#toString` to bypass the value's own `toString` method
          // and avoid being faked out.
          ? reNative.test(fnToString.call(value))
          // Fallback to a host object check because some environments will represent
          // things like typed arrays as DOM methods which may not conform to the
          // normal native pattern.
          : (value && type == 'object' && reHostCtor.test(toString.call(value))) || false;
      }
      
      // export however you want
      module.exports = isNative;
    }());
    
    // Usage
    isNative(alert); // true
    isNative(myCustomFunction); // false
    

The function isn't pretty but it gets the job done!

## [`insertRule`][6]

We all know that we can grab a NodeList from a selector (via 
`document.querySelectorAll`) and give each of them a style, but what's more
efficient is setting that style to a selector (like you do in a stylesheet
):

    var sheet = (function() {
    	// Create the <style> tag
    	var style = document.createElement('style');
    
    	// Add a media (and/or media query) here if you'd like!
    	// style.setAttribute('media', 'screen')
    	// style.setAttribute('media', 'only screen and (max-width : 1024px)')
    
    	// WebKit hack :(
    	style.appendChild(document.createTextNode(''));
    
    	// Add the <style> element to the page
    	document.head.appendChild(style);
    
    	return style.sheet;
    })();
    
    // Usage
    sheet.insertRule("header { float: left; opacity: 0.8; }", 1);
    

This is especially useful when working on a dynamic, AJAX-heavy site.  If you
set the style to a selector, you don't need to account for styling each element 
that may match that selector (now or in the future
).

## [`matchesSelector`][7]

Oftentimes we validate input before moving forward; ensuring a truthy value,
ensuring forms data is valid, etc.  But how often do we ensure an element 
qualifies for moving forward?  You can use a`matchesSelector` function to
validate if an element is of a given selector match:

    function matchesSelector(el, selector) {
    	var p = Element.prototype;
    	var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
    		return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
    	};
    	return f.call(el, selector);
    }
    
    // Usage
    matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')
    

There you have it:  seven JavaScript functions that every developer should
keep in their toolbox.  Have a function I missed?  Please share it!</article>

![Track.js Error Reporting][8]

## [Recent Features**][9]

*   ![6 Things You Didn&#8217;t Know About Firefox OS][10]### 
    [6 Things You Didn’t Know About Firefox OS][11]
    
    [Firefox OS][12] is all over the tech news and for good reason:  Mozilla's
    finally given web developers the platform that they need to create apps the way 
    they've been creating them for years -- with CSS, HTML, and JavaScript.  Firefox
    OS has been rapidly improving,
    ...

*   ![CSS Gradients][10]### [CSS Gradients][13]
    
    [][14] With [CSS border-radius][15], I showed you how CSS can bridge the
    gap between design and development by adding rounded corners to elements.  CSS 
    gradients are another step in that direction.  Now that CSS gradients are 
    supported in Internet Explorer 8+, Firefox, Safari, and Chrome,
    ...

## [Incredible Demos**][16]

*   ![CSS Scoped Styles][10]### [CSS Scoped Styles][17]
    
    There are plenty of awesome new attributes we've gotten during the HTML5
    revolution:
     [placeholder][18], [download][19], [hidden][20], and more.  Each of these
    attributes provides us a different level of control over an element on the page,
    but there's a new element attribute that allows.
    ..

*   ![MooTools&#8217; AutoCompleter Plugin][10]

## [Recently on David Walsh Blog**][21]

*   ![Improving WordPress Commenting with Postmatic][10]### 
    [Improving WordPress Commenting with Postmatic][22]
    
    We've set out to create a fantastic commenting plugin for WordPress. It's
    called Postmatic and what it does is a first for any blogging system: to allow 
    synchronous 100% email and web-based commenting. The web folks can engage via 
    the web. The email folks can stick.
    ..

*   ![git: Delete All Branches but Master][10]### 
    [git: Delete All Branches but Master][23]
    
    Maintenance is incredibly important in any project, but if you want to take
    your professionalism to the next level, you should keep your git environment in 
    shape.  Unfortunately I'm not that guy -- I leave git branches laying around, 
    even after they've been merged into
   `master`....

*   ![4 Selling Points For Your Next Website Design Pitch][10]### 
    [4 Selling Points For Your Next Website Design Pitch][24]
    
    An important skill for those looking to build a successful web design
    career is learning to effectively sell your services by explaining to clients 
    the benefits of redesigning their website. While different clients will have 
    different things that are important to them in a new website,
    ...

*   ![7 Essential JavaScript Functions][10]### 
    [7 Essential JavaScript Functions][25]
    
    I remember the early days of JavaScript where you needed a simple function
    for just about everything because the browser vendors implemented features 
    differently, and not just edge features, basic features, like
   `addEventListener` and `attachEvent`.  Times have changed but there are
    still a few functions each developer should.
    ..

*   ![File Input accept Attribute][10]### [File Input accept Attribute][26]
    
    The HTML5 revolution provided us several simple but important attributes
    like
   `<a href="http://davidwalsh.name/download-attribute">download</a>`, 
    `<a href="http://davidwalsh.name/autofocus">autofocus</a>`, 
    `required`, 
    `<a href="http://davidwalsh.name/novalidate">novalidate</a>`, and 
    `<a href="http://davidwalsh.name/html5-placeholder">placeholder</a>`
   `accept`.  The `accept` attribute is useful for `input[type=file]` elements
    .  Let's have a look at it! The HTML I'll.
    ..

 [1]: http://davidwalsh.name/javascript-debounce-function
 [2]: http://davidwalsh.name/javascript-polling
 [3]: http://davidwalsh.name/javascript-once
 [4]: http://davidwalsh.name/get-absolute-url
 [5]: http://davidwalsh.name/detect-native-function
 [6]: http://davidwalsh.name/add-rules-stylesheets
 [7]: http://davidwalsh.name/element-matches-selector
 [8]: img/trackjs.jpg
 [9]: http://davidwalsh.name/tutorials/features
 [10]: img/
 [11]: http://davidwalsh.name/firefox-os
 [12]: http://www.mozilla.org/en-US/firefoxos/
 [13]: http://davidwalsh.name/css-gradients
 [14]: http://davidwalsh.name/demo/css-gradient.php
 [15]: http://davidwalsh.name/css-rounded-corners
 [16]: http://davidwalsh.name/tutorials/demos
 [17]: http://davidwalsh.name/scoped-css
 [18]: http://davidwalsh.name/html5-placeholder
 [19]: http://davidwalsh.name/download-attribute
 [20]: http://davidwalsh.name/html5-hidden
 [21]: http://davidwalsh.name/page/1
 [22]: http://davidwalsh.name/improving-wordpress-commenting-postmatic
 [23]: http://davidwalsh.name/git-delete-branches-master
 [24]: http://davidwalsh.name/4-selling-points-website-design-pitch
 [25]: http://davidwalsh.name/essential-javascript-functions
 [26]: http://davidwalsh.name/accept-attribute