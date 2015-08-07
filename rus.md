Когда JavaScript только появился, нам нужно было
писать простые функции почти для всего, потому что производители
браузеров реализовывали одни и те же вещи по-разному. 
И это относилось не только к самым новым функциям, но и к обычным 
тоже, таким как ```addEventListener``` или ```attachEvent```.
Сейчас всё изменились, но всё есть еще несколько функций 
для выполнения простых рутиных задач, которые каждый
разработчик должен иметь в своём распоряжении.

### [```debounce```][1]

Функция ```debounce``` может сыграть важную роль когда дело касается
производительности событий. Если вы не используете ```debounce```
с событиями ```scroll```, ```resize```, ```key*```, скорее всего, вы
делаете что-то не так. Ниже приведен код функции ```debounce```, которая
поможет повысить производительность вашего кода:

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

```debounce``` не позволит обратному вызову исполняться чаще,
чем один раз в заданный период времени. Это особенно важно 
при назначении функции обратного вызова для часто вызываемых событий.

### [```poll```][2]

Функцию ```debounce``` не всегда возможно подключить для обозначения
желаемого состояния: если событие не существует — это будет не возможно.
В этом случае вы должны проверять состояние с помощью интервалов:

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
    

Техника «опроса» уже давно используется в вебе и будет 
продолжать использоваться в будущем.

### [```once```][3]

Иногда бывает нужно, чтобы функция выполнилась только один раз, как если
бы вы использовали событие ```onload```. Функция ```once``` даёт
такую возможность:

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
    
```once``` гарантирует, что заданная функция будет 
вызвана только один раз, что предотвращает повторную инициализацию.

### [`getAbsoluteUrl`][4]

Получить абсолютный URL из строчной переменной не так просто, как кажется.
Существует конструктор ```URL```, но он «барахлит», если вы не
предоставите требуемые параметры (которых у вас может не быть). 
Вот хитрый способ получить абсолютный ```URL``` из строки:

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

Передаваемый на вход ```href``` и URL не имеют значения, функция 
в любом случае вернёт надежный абсолютный URL в ответ.

### [`isNative`][5]

Знание является ли функция нативной может быть полезно, если вы
хотите её переопределить. Этот код поможет вам знать наверняка:

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
    
Функция выглядит не очень аккуратно, но она работает!

### [`insertRule`][6]

Все мы знаем, что можно получить список элементов по селектору
(с помощью ```document.querySelectorAll```) и добавить каждому из 
них стили, но более эффективным будет задать те же стили селектору 
(как вы делаете в таблице стилей):

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
    
Это будет особенно полезным когда вы работаете с динамическим сайтом
с большим количеством AJAX. Если вы добавите стили для конкретного
селектора, вам не придется каждый раз добавлять эти же стили для
элементов, которые могут соответствовать этому селектору сейчас 
или в будущем.

### [`matchesSelector`][7]

Как правило, мы проверям входные данные, прежде чем продолжать 
что-то делать: удостверяемся, что значение верно, что содержимое 
формы валидно, и т.д.
Но как часто мы проверяем, что имеющийся элемент подходит 
для дальнейших действий? Для проверки соответствия элемента
заданному селектору, вы можете использовать функцию ```matchesSelector```:

    function matchesSelector(el, selector) {
    	var p = Element.prototype;
    	var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
    		return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
    	};
    	return f.call(el, selector);
    }
    
    // Usage
    matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')

Теперь у вас есть семь JavaScript функции, которые каждый разработчик
должен иметь в своем арсенале. Есть функция, которую я пропустил?
Пожалуйста, поделитесь!

 [1]: http://davidwalsh.name/javascript-debounce-function
 [2]: http://davidwalsh.name/javascript-polling
 [3]: http://davidwalsh.name/javascript-once
 [4]: http://davidwalsh.name/get-absolute-url
 [5]: http://davidwalsh.name/detect-native-function
 [6]: http://davidwalsh.name/add-rules-stylesheets
 [7]: http://davidwalsh.name/element-matches-selector
