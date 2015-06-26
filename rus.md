Я помню первые дни JavaScript, где вам нужно было реализовывать простую функцию почти для всего, потому что производители браузеров реализовывали функции по-разному. Это относилось не только к специфичным функциям, но и к  основным тоже, таким как ```addEventListener``` ```attachEvent```.
Времена изменились, но есть еще несколько функций, которые каждый разработчик должен иметь в своем арсенале, для выполнения простых рутиных задач.

## [```debounce```][1]

Функция ```debounce``` может быть незаменимой, когда дело доходит до производительности событий. Если вы не используете функцию ```debounce``` с событиями ```scroll```, ```resize```, ```key*```, вы, вероятно, используете ее неправильно. Вот реализация функции ```debounce```, чтобы сделать ваш код эффективным:

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

Функция ```debounce``` не будет позволять обратному вызову использоваться более чем один раз в определенный период времени. Это особенно важно при назначении функции обратного вызова для частой "стрельбы" событиями.

## [```poll```][2]

Как я уже говорил с функцией ```debounce```, иногда у вас не получится ее подключить для обозначения желаемого состояния - если событие не существует. В этом случае вы должны проверять состояние с интервалами:

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
    

Техника "опроса" уже давно используется в вебе и будет оставаться эффективной в будущем!

## [```once```][3]

Есть моменты, когда вы предпочитаете, чтобы данная функциональность произошло только один раз, аналогично тому, как вы бы использовать событие ```onload```. Этот код предоставляет вам эту функциональность:

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
    
Функция ```once``` гарантирует, что данная функция может быть вызвана только один раз, таким образом, предотвращает дублирование инициализации!

## [`getAbsoluteUrl`][4]

Получение абсолютного URL из переменной строки не так просто, как вы думаете.
Существует конструктор ```URL```, но он может действовать, если вы не предоставите требуемые параметры (которые иногда не можете). Вот хитрый трюк для получения абсолютного ```URL``` из строки ввода:

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

The "burn" element ```href``` handles and URL nonsense for you, providing a
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
keep in their toolbox.  Have a function I missed?  Please share it!

 [1]: http://davidwalsh.name/javascript-debounce-function
 [2]: http://davidwalsh.name/javascript-polling
 [3]: http://davidwalsh.name/javascript-once
 [4]: http://davidwalsh.name/get-absolute-url
 [5]: http://davidwalsh.name/detect-native-function
 [6]: http://davidwalsh.name/add-rules-stylesheets
 [7]: http://davidwalsh.name/element-matches-selector