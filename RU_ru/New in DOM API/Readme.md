# Новинки DOM API

## Преамбула

В данной статье я расскажу о новинка в DOM API, которые мы можем использовать уже сейчас или в ближайшем будущем. Публикация статьи приурочена к радостному событию [реализации](https://plus.google.com/u/0/115203843155141445032/posts/VGDqpLHsUjn) некоторых новых DOM4 API методов в Google Chrome.
Многие методы и свойства можно использовать уже сейчас, некоторые из них работают через префиксы, но к каждому методу или свойству я постараюсь дать Polyfill, реализующий их или отбрасывающий браузерные префиксы.
  
## DOM4 Mutation methods

[Спецификация](http://www.w3.org/TR/dom/)

У `Element.prototype` появился ряд очень интересных методов, которые будут знакомы любителям jQuery, однако работают несколько по-другому.  

* `append(...(Node|string))`
* `preppend(...(Node|string))`
* `before(...(Node|string))`
* `after(...(Node|string))`
    
    Данные метод вставляет в текущую ноду n-нное количество нод. Ноды передаются как аргументы функции (Например: `node.append(otherNode1, otherNode2)`.
    При это, вы можете передать вместо ноды текст, и он автоматически преобразуется в _TextNode_, как если бы вы вызвали `document.createTextNode(text)`.
    Функция `append` вставляет ноды в конец своего списка нод, `preppend` - в начало, функции `before` и `after` - перед и после текущей ноды, соответственно.
     
    @param {(Node|string)} nodesAndText
* `delete`
    Метод удаляет текущую ноду из родителя.
* `replace(...(Node|string))`
    Метод заменяет текущую ноду, одной или несколькими нодами, которые указываются в качестве параметров метода.
    
    Все эти методы не имеют возвращаемого значения.

Вот такой паттерн позволит вам передавать в этот метод **NodeList** или массив с нодами:
    
    element.append.apply(element, document.querySelectorAll("div"))

В `document` было добавлено два метода из этих шести: `append` и `preppend`.

DOM4 Mutation methods [реализованы](https://plus.google.com/u/0/115203843155141445032/posts/VGDqpLHsUjn) в последней версии Google Chrome.
_Polyfill этих методов для всех браузеров есть в моей [библиотеке](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L2300)._
    
## DOM Selector API 2: NodeRef и :scope

[Спецификация](http://www.w3.org/TR/selectors-api2/)

* `document.querySelector(string, NodeRef{(Node|NodeList|Array.<Node>)})`
* `document.querySelectorAll(string, NodeRef{(Node|NodeList|Array.<Node>)})`
* `document.find(string, NodeRef{(Node|NodeList|Array.<Node>)})`
* `document.findAll(string, NodeRef{(Node|NodeList|Array.<Node>)})`
    
Старые добрые методы querySelector\[All\] обзавелись вторым параметром, но только в реализации для `document`. А также добавились новые методы find\[All\].

Второй параметр позволяет указать контекст, в котором мы будет искать ноды по селектору. Например:

    document.findAll("li", [ulElement1, ulElement2])
    
Найдёт все элементы `<li>` в двух элементах _ulElement1_ и _ulElement2_.

Псевдо класс _:scope_ позволяет делать действительно класные вещи. Это ещё один способ указать контекст поиска - он указывает на текущий элемент, в котором  мы проводим поиск по селектору.
Он позволяет искать по ранее не валидным селекторам ">.class" или "~tagname". Просто укажите _:scope_ вначале и данные селекторы станут валидными. Вся мощь _:scope_ видна, когда
мы применяем его вместе с NodeRef:

    document.querySelectorAll(":scope > p", [divElement1, divElement2])

Найдёт всех _непосредственных_ детей с тэгом _p_ у _divElement1_ и _divElement2_! Хочу заметить, что вы можете не применять
псевдо-класс :scope для методов `find` и `findAll`. Для этих методов такой вызов считается валидным:

    document.findAll("> p", [divElement1, divElement2])

_Подержку `find[All]` и **:scope** в `querySelectorAll`, соответствующую спецификации, я сейчас делаю._

## Element.prototype.matches

[Спецификация](http://www.w3.org/TR/selectors-api2/#matches)

Этот метод ранее назывался **matchesSelector**. Он проверяет ноду на соответствие CSS-селектору.
_Маленький polyfill с отбрасывание браузерных префиксов: [gist](https://gist.github.com/2369850). Более расширенный вариант у меня в [библиотеке](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L2219)._

## [classList](https://developer.mozilla.org/en-US/docs/DOM/element.classList)

[Спецификация](http://dom.spec.whatwg.org/#domtokenlist)

DOM API для работы с CSS-классами элемента.

Методы:

* `add(...class{string})` Добавляет _class_ в **element.className**
* `remove(...class{string})` Удаляет _class_ из **element.className**
* `toggle(boolean: class{string})` Удаляет _class_ в случеи его наличия из **element.className** и возвращает **false**. Добавляет в случае его отсутсвие в **element.className** и возвращает **true**.
* `contains(boolean: class{string})` Проверяет _class_ на его наличие в **element.className**. Возвращает **true** или **false** соответственно.

Ранее, методы `add` и `remove` работали только с одним классом за раз, а недавно в стандарт была [добавлена возможность](http://lists.w3.org/Archives/Public/www-dom/2012JulSep/0101.html) работать с несколькими CSS-классами:

    document.documentElement.classList.add("test1", "test2", "test3")
    document.documentElement.classList.remove("test1", "test2", "test3")

_Polyfill для `classList`, пока старой спецификации, есть у меня в [библиотеке](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L1867). В скором времени будет доступна и новая версия, которая будет патчить старую реализацию._

## Events Constructor

[Спецификация](http://www.w3.org/TR/domcore/#events)

В новом стандарте DOM API определены конструкторы для событий. Теперь мы можем забыть про мучения с

    event = document.createEvent("click")
    event.initMouseEvent( /*тут что-то, я даже сам не помню что*/ )

А просто написать:

    event = new Event("click")

В конструктор мы можем передать любое текстовое значение в качестве `e.type`, вторым параметром передаётся объект, который содержит инициализационные параметры **bubbles** и **cancelable**. **bubbles** установленное в **false** предотвратит всплытие события. **cancelable** установленное в **false** предотвратит отмену события через метод `preventDefault`.
По-умолчанию, **bubbles** и **cancelable** равны **false**.

    event = new Event("some:event", {bubbles : true, cancelable : false})
    event = new Event("dbclick1")
    element.dispachEvent(event)

Если необходимо довавить данные для обработчика, просто добавить их в объект `event`:

    event = new Event("click")
    event.button = 1

Замечу, что при этом все созданные таким образом события являются _обычными_ событиями, те `new Event("click")` создаст не **MouseEvent**, а просто **Event**.

Второй класс для создания событий - это `CustomEvent`. Использование этого класса отличается только тем, что и инициализационный объект можно передать свойство **detail**

    event = new CustomEvent("somecustom:event", {bubbles : true, cancelable : false, detail : {"some data" : 123}})

Конструкторы `Event` и `CustomEvent` реализованы во всех современных браузерах (уже больше года). _[Polyfill](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L1662) для старых браузеров._

Также, идёт обсуждение, для специальных событий клавиатуры, Drag&Drop и мыши. Но пока они не реализованы ни одним браузером.
(Замечание: _Opera 12.10 поддерживает конструктор для `KeyboardEvent`, но он работает не по [спецификации](http://www.w3.org/TR/DOM-Level-3-Events/#events-keyboardevents))_

## HTMLLabelElement.prototype.control и HTML*Element.prototype.labels для элементов формы

[Спецификация для control](http://www.w3.org/TR/html5/the-label-element.html#dom-label-control)

Свойство **control** содержит ссылку на _элемент формы_ с которым связан данный `<label>` элемент:

    label.control.value = "123"

[Спецификация для labels](http://www.w3.org/TR/html5/the-label-element.html#dom-lfe-labels)

Свойство **labels**, наоборот, содержит массив элементов `<label>` для элемента формы. Массив потому, что по спецификации
у одного _элемента формы_ может быть несколько связанных `<label>` элементов.

    console.log(input.labels.length)

Свойства реализованы в большинстве браузеров. _[Polyfill для старых браузеров](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L2460)._
    
## HTMLOlElement.prototype.revert

[Спецификация](http://www.whatwg.org/specs/web-apps/current-work/multipage/grouping-content.html#the-ol-element)

Свойство **revent** у нумерованного списка, заставляет нумерацию идти в обратную сторону. При это учитывается свойство **start**. Вот простые примеры, которые должны сказать больше слов:

    <ol reversed>  
        <li>Элемент списка 1</li>
        <li>Элемент списка 2</li>  
        <li>Элемент списка 3</li>
        <li>Элемент списка 4</li>  
        <li>Элемент списка 5</li>  
    </ol>

Будет интерпретировано браузером как:
    
    5. Элемент списка 1
    4. Элемент списка 2
    3. Элемент списка 3
    2. Элемент списка 4
    1. Элемент списка 5

А такая разметка:

    <ol reversed start=100>  
        <li>Элемент списка 1</li>  
        <li>Элемент списка 2</li>  
        <li>Элемент списка 3</li>  
        <li>Элемент списка 4</li>  
        <li>Элемент списка 5</li>  
    </ol>

будет интерпретирована браузером как:
    
    100. Элемент списка 1
    99. Элемент списка 2
    98. Элемент списка 3
    97. Элемент списка 4
    96. Элемент списка 5

Поддержка свойства **revert** есть во всех современных браузерах кроме Opera'ы. _Для неё есть [Polyfill](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L2570)._

## Event.prototype.stopImmediatePropagation

[Спецификация](http://www.w3.org/TR/DOM-Level-3-Events/#events-event-type-stopImmediatePropagation)

Метод работает аналогично такому же [методу из jQuery](http://api.jquery.com/event.stopImmediatePropagation/).

Вкрадце, он останавливает обработку события _сразу же_, а не только всплытие события.

Только Opera до версии 12.10, не поддерживает этот метод. _[Polyfill](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.js#L1745) для неё._

## Поддержка IE < 9

В IE8 есть прототипы DOM-объектов, поэтому добавить туда поддержку не составит труда. Я вынес polyfill'ы для IE8 в [отдельный файл](https://github.com/termi/ES5-DOM-SHIM/blob/0.7/__SRC/a.ie8.js) и подключаю его через _Conditional Comments_.

Для IE6/7 всё сложнее, нужно либо отказаться от этих браузеров полностью или полностью реализовывать DOM API для них, что я и сделал, о чём рассказывал в своей статье [DOM-shim для всех браузеров включая IE < 8](http://habrahabr.ru/post/133328/).


_Код стать выложен на [github](https://github.com/termi/CreativeWork/tree/NEW_DOM_ES/RU_ru/New%20in%20DOM%20API). Если вы хотите помочь или нашли ошибку, сделайте pull request или напишите мне в личку. Также напишите в комментариях, если какаю-то тему нужно рассмотреть подробнее или нужно больше примеров._
