# DOM-shim для всех браузеров включая IE < 8

Многие javascript-программисты сталкивались с не поддерживанием некоторых функций DOM JS API в некоторых браузерах (не будем показывать пальцем). Наверняка, многие знакомы с замечательными библиотеками [es5-shim](https://github.com/kriskowal/es5-shim) и [DOM-shim](https://github.com/Raynos/DOM-shim) для решения проблем совместимости между разными браузерами, а [DOM-shim](https://github.com/Raynos/DOM-shim) к тому же, «подтягивает» браузер до уровня DOM4.

В данной же статье я расскажу, как сделать DOM-shim в IE6 и IE7, чтобы навсегда забыть о существовании этих браузеров.

## Проблемма

Тут и рассказывать нечего. IE ранее 9й версии не поддерживает очень много из стандартного DOM API:

* addEventListener
* removeEventListener
* dispachEvent
* classList
* и т.д.

И если IE8 можно «подтянуть» имплементировав необходимый функционал с помощью [DOM-shim](https://github.com/Raynos/DOM-shim), то IE6 и IE7, к сожалению, не поддерживаются этой библиотекой из-за того, что в IE < 8 нету Node.prototype.

## Решение

Собственно, решение достаточно очевидное и я использовал его в своих проектах еще год назад, когда только начал изучать javascript — behavior: url(.htc) [[Introduction to DHTML Behaviors](http://msdn.microsoft.com/en-us/library/ms531079(v=vs.85).aspx)]. Но тогда мне пришлось отказаться от этого решения, потому что, это очень-очень сильно тормозило загрузку страницы — секунд 30 приходилось ждать, уже после того, как страница загрузилась для относительно небольшой страницы.
Спустя какое-то время, случайно наткнулся на информацию о том, как ускорить раз в 100 загрузку страниц с использованием behavior — оказывается нужно просто добавить [lightweight=true](http://msdn.microsoft.com/en-us/library/ms531426(v=vs.85).aspx#LightweightHTCs) и всё будет работать достаточно быстро.
Теперь сделать DOM-shim для IE < 8 не составит никакого труда, что я и сделал.

## Приступим

Для начала нам понадобятся оригинальные [es5-shim](https://github.com/kriskowal/es5-shim) и [DOM-shim](https://github.com/Raynos/DOM-shim) и [DOM-shim](https://github.com/Raynos/DOM-shim), берём их и переделываем, исправляем баги и улучшаем эмуляцию Object.defineProperty:

    if (!object.__defineGetter__) {
      if(descriptor["ielt8"]) {
        object["get" + property] = descriptor["get"];
        object["set" + property] = descriptor["set"];
      }
    else throw new TypeError(ERR_ACCESSORS_NOT_SUPPORTED);

Вместо того, чтобы просто выбрасывать исключение **ERR_ACCESSORS_NOT_SUPPORTED** мы проверим на наличие специального флага и если он есть, сохраним геттер и сеттер под специальными именами.
Т.к. в IE < 8 нету Node.prototype (который и передаётся в Object.defineProperty), создадим специальный объект, в который будет аккумулировать наши геттеры и сеттеры:

    var elementProto = window.HTMLElement && window.HTMLElement.prototype ||
          /*ie8*/window.Element && window.Element.prototype ||
          /*ielt8*/(window["_ielt8_Element_proto"] = {});

Передавать мы его будем как обычно, только добавим в description флаг «ielt8»:

    Object.defineProperty(elementProto, "classList", {"get" : ..., "ielt8" : true}

Теперь мы собираем все геттеры нашей shim-библиотеки в один объект. Остаётся только «навесить» его на все элементы на странице.
Создадим dom-shim.ielt8.htc файл:

    <PUBLIC:COMPONENT lightWeight="true">
    <PUBLIC:PROPERTY NAME="classList" GET="getClassList" />
    <SCRIPT>
    getClassList = window._ielt8_Element_proto.getclassList
    </SCRIPT>
    </PUBLIC:COMPONENT>

и добавим его ко всем элементам на странице:

    * {
     behavior: url(dom-shim.ielt8.htc)
    }

И собственно всё. Теперь у нас есть Node.classList даже в IE6!

Аналогично добавляются следующие свойства:
* addEventListener
* removeEventListener
* dispatchEvent
* attributes (исправлено)
* children (исправлено)
* firstElementChild
* lastElementChild
* nextElementSibling
* previousElementSibling
* childElementCount
* querySelectorAll
* querySelector
* insertAfter (нестандартная)
* getElementsByClassName
* compareDocumentPosition
* DOCUMENT_POSITION_DISCONNECTED
* DOCUMENT_POSITION_PRECEDING
* DOCUMENT_POSITION_FOLLOWING
* DOCUMENT_POSITION_CONTAINS
* DOCUMENT_POSITION_CONTAINED_BY

## Итого

Ссылка на библиотеку: [github.com/termi/ES5-DOM-SHIM](https://github.com/termi/ES5-DOM-SHIM)
Хочу заметить, что моя библиотека для DOM-shim сильно отличается от [DOM-shim](https://github.com/Raynos/DOM-shim/) в силу того, что я разрабатывал её последние 8 месяцев ничего не зная про [DOM-shim](https://github.com/Raynos/DOM-shim/) (и даже про github :))

Рабочий пример можно скачать [тут](http://h123.ru/-/examples/ES6-DOM4-SHIM/parallax/) или [тут](http://h123.ru/-/examples/ES6-DOM4-SHIM/simple/)

## Ограничения

.htc-файлы должны находится на том же домене, что и .html-файл. Например для файла example.org/index.html, .htc-файл должен находится на example.org, а для файла test.example.org/index.html на test.example.org. Это ограничение существенно усложняет использование библиотеки — нельзя просто выложить .htc-файл на статик-сервер и забыть о нём. Обязательно нужно убедится в наличии всех нужных .htc-файлов на тот же домен, где и сайт.
[Same-domain limitation](http://css3pie.com/documentation/known-issues/#x-domain)
[Решение Same-domain limitation через nginx proxy](https://github.com/termi/ES5-DOM-SHIM/blob/master/extra/SameDomainLimitation.SOLVE_RUS.odt?raw=true)
