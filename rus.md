# 7 жизненно важных функций в JavaScript

Когда JavaScript только появился, нам нужно было
писать простые функции почти для всего, потому что производители
браузеров реализовывали одни и те же вещи по-разному. 
И это относилось не только к самым новым функциям, но и к обычным, 
таким как ```addEventListener``` или ```attachEvent```.
Сейчас всё изменилось, но всё есть еще несколько функций 
для выполнения простых рутиных задач, которые каждый
разработчик должен иметь в своём распоряжении.

## `debounce`

Функция [`debounce`][1] может сыграть важную роль когда дело касается
производительности событий. Если вы не используете `debounce`
с событиями `scroll`, `resize`, `key*`, скорее всего, вы что-то 
делаете не так. Ниже приведен код функции `debounce`, которая
поможет повысить производительность вашего кода:

    // Возвращает функцию, которая не будет срабатывать, пока продолжает вызываться.
    // Она сработает только один раз через N миллисекунд после последнего вызова.
    // Если ей передан аргумент `immediate`, то она будет вызвана один раз сразу после
    // первого запуска.
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
     
    // Использование
    var myEfficientFn = debounce(function() {
    // All the taxing stuff you do
    }, 250);
    window.addEventListener('resize', myEfficientFn);

`debounce` не позволит обратному вызову исполняться чаще,
чем один раз в заданный период времени. Это особенно важно 
при назначении функции обратного вызова для часто вызываемых событий.

## `poll`

Функцию `debounce` не всегда возможно подключить для обозначения
желаемого состояния: если событие не существует — это будет не возможно.
В этом случае вы должны проверять состояние с помощью интервалов:

    function poll(fn, callback, errback, timeout, interval) {
      var endTime = Number(new Date()) + (timeout || 2000);
      interval = interval || 100;
      
      (function p() {
        // В случае успешного выполнения условия
        if(fn()) {
          callback();
        }
        // Условие не выполнилось, но время ещё не вышло (тик интервала)
        else if (Number(new Date()) < endTime) {
          setTimeout(p, interval);
        }
        // Условие не выполнилось, а отведённое время вышло
        else {
          errback(new Error('timed out for ' + fn + ': ' + arguments));
        }
      })();
    }
    
    // При использовании: убедитесь что элемент видим
    poll(
        function() {
            return document.getElementById('lightbox').offsetWidth > 0;
        },
        function() {
            // Done, success callback
        },
        function() {
            // Error, failure callback
        }
    );

Техника «[опроса][2]» уже давно используется в вебе и будет продолжать использоваться в будущем.

## `once`

Иногда бывает нужно, чтобы функция выполнилась только один раз, как если
бы вы использовали событие `onload`. Функция [`once`][3] даёт такую возможность:

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
     
    // Пример использования
    var canOnlyFireOnce = once(function() {
    	console.log('Запущено!');
    });
     
    canOnlyFireOnce(); // "Запущено!"
    canOnlyFireOnce(); // Не запущено
    
`once` гарантирует, что заданная функция будет вызвана только один раз,
что предотвращает повторную инициализацию.

## `getAbsoluteUrl`

[Получить абсолютный URL][4] из строчной переменной не так просто, как кажется.
Существует конструктор `URL`, но он «барахлит», если вы не
предоставите требуемые параметры (которых у вас может не быть). 
Вот хитрый способ получить абсолютный `URL` из строки:

    var getAbsoluteUrl = (function() {
    	var a;
        
    	return function(url) {
    		if(!a) a = document.createElement('a');
    		a.href = url;
            
    		return a.href;
    	};
    })();
        
    // Пример использования
    getAbsoluteUrl('/something'); // http://davidwalsh.name/something

Передаваемый на вход `href` и URL не имеют значения, функция 
в любом случае вернёт надежный абсолютный URL в ответ.


## `isNative`

Если вы хотите переопределить функцию, то полезно знать, является
ли она нативной:

    ;(function() {
        
      // Используется для получения внутреннего `[[Class]]` значений
      var toString = Object.prototype.toString;
      
      // Используется для получения декомпилированного кода функций
      var fnToString = Function.prototype.toString;
      
      // Используется для определения родительских конструкторов (Safari > 4; специфично для типизированных массивов)
      var reHostCtor = /^\[object .+?Constructor\]$/;
        
      // Создадим регулярное выражение используя в качестве шаблона общедоступный нативный метод.
      // Мы выбрали `Object#toString` потому что его, скорее всего, не изменяли.
      var reNative = RegExp('^' +
        // Принудительно переводим `Object#toString` в строку
        String(toString)
        // Экранируем все специальные символы регулярного выражения
        .replace(/[.*+?^${}()|[\]\/\\]/g, '\\$&')
        // Заменяем все упоминания `toString` на `.*?` для поддержания обобщённого вида.
        // Заменяем конструкции типа `for ...`, что бы поддерживать окружения
        // вроде Rhino, который добавляет дополнительную информацию (например, арность).
        .replace(/toString|(function).*?(?=\\\()| for .+?(?=\\\])/g, '$1.*?') + '$'
      );
      
      function isNative(value) {
        var type = typeof value;
        return type == 'function'
          // Используем `Function#toString` что бы обойти собственный
          // метод `toString` value и не дать нас обмануть.
          ? reNative.test(fnToString.call(value))
          // Проверяем родительский объект, так как некоторые окружения представляют
          // штуки вроде типизированных массивов в виде DOM методов, что может 
          // не соответствовать нормальному нативному шаблону
          : (value && type == 'object' && reHostCtor.test(toString.call(value))) || false;
      }
      
      // Экспортируете то, что сочтёте нужным
      module.exports = isNative;
    }());
        
    // Пример использования
    isNative(alert); // true
    isNative(myCustomFunction); // false
    
Функция выглядит не очень аккуратно, но она работает!


## `insertRule`

Все мы знаем, что можно получить список элементов по селектору
(с помощью `document.querySelectorAll`) и добавить каждому из 
них стили, но более эффективным будет задать те же стили селектору 
(как вы делаете в таблице стилей), с помощью функции [`insertRule`][6]:

    var sheet = (function() {
    	// Создаём элемент <style>
    	var style = document.createElement('style');
        
    	// Если хотите, то добавляем атрибут `media` (и/или медиа-выражения)
    	// style.setAttribute('media', 'screen')
    	// style.setAttribute('media', 'only screen and (max-width : 1024px)')
        
    	// Хак для WebKit :(
    	style.appendChild(document.createTextNode(''));
        
    	// Добавляем на страницу элемент <style>
    	document.head.appendChild(style);
        
    	return style.sheet;
    })();
        
    // Пример использования
    sheet.insertRule("header { float: left; opacity: 0.8; }", 1);
    
Это будет особенно полезно когда вы работаете с динамическим сайтом
с большим количеством AJAX. Если вы добавите стили для конкретного
селектора, вам не придётся каждый раз добавлять эти же стили для
элементов, которые могут соответствовать этому селектору сейчас 
или в будущем.


## `matchesSelector`

Как правило, мы проверям входные данные, прежде чем продолжать 
что-то делать: удостверяемся, что значение верно, что содержимое 
формы валидно, и т.д.

Но как часто мы проверяем, что имеющийся элемент подходит 
для дальнейших действий? Для проверки соответствия элемента
заданному селектору вы можете использовать функцию [`matchesSelector`][7]:

    function matchesSelector(el, selector) {
        var p = Element.prototype;
        var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
            return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
        };
        return f.call(el, selector);
    }
     
    // Пример использования
    matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')

Теперь у вас есть семь JavaScript-функций, которые каждый разработчик
должен иметь в своем арсенале. Есть функция, которую я пропустил?
Пожалуйста, поделитесь!

 [1]: http://davidwalsh.name/javascript-debounce-function
 [2]: http://davidwalsh.name/javascript-polling
 [3]: http://davidwalsh.name/javascript-once
 [4]: http://davidwalsh.name/get-absolute-url
 [5]: http://davidwalsh.name/detect-native-function
 [6]: http://davidwalsh.name/add-rules-stylesheets
 [7]: http://davidwalsh.name/element-matches-selector
