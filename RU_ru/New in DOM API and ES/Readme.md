# Новинки DOM API и EcmaScript

## Преамбула

В данной статье я расскажу о новинка в DOM API и EcmaScript, которые мы можем использовать уже сейчас. Тут не будет говорится о таких
вкусных технологиях как [Proxy](http://wiki.ecmascript.org/doku.php?id=harmony:proxies) или [Modules](http://wiki.ecmascript.org/doku.php?id=harmony:modules),
т.к. их невозможно [реализовать](http://remysharp.com/2010/10/08/what-is-a-polyfill/) на plain javascript.
  
## DOM4 Mutation methods

У `Element.prototype` и `document` появился ряд очень интересных методов, которые будут знакомы любителям jQuery, однако работают несколько по-другому.  

* `append(...(Node|string))`
* `preppend(...(Node|string))`
* `before(...(Node|string))`
* `after(...(Node|string))`
    
    Данные метод вставляет в текущую ноду n-нное колличество нод. Ноды передаются как аргументы функции (Например: node.append(otherNode1, otherNode2).
    При это, вы можете передать вместо ноды текст, и он автоматически преобразуется в TextNode, как если бы вы вызвали document.createTextNode(text).
    Функция `append` вставляет ноды в конец своего списка нод, `preppend` - в начало, функции `before` и `after` - перед и после текущей ноды, соответственно.
     
    @param {(Node|string)} nodesAndText
* `delete`
    Метод удаляет текущую ноду из родителя.
* `replace(...(Node|string))`
    Метод заменяет тукущую ноду, одной или несколькими нодами, которые указываются в качестве параметров метода.
    
Все эти методы не имеют возвращяемого значения.
    
## DOM4 Selector API: NodeRef и :scope

    * `document.querySelector(string, NodeRef{(Node|NodeList|Array.<Node>)})`
    * `document.querySelectorAll(string, NodeRef{(Node|NodeList|Array.<Node>)})`
    * `document.find(string, NodeRef{(Node|NodeList|Array.<Node>)})`
    * `document.findAll(string, NodeRef{(Node|NodeList|Array.<Node>)})`
    
Старые добыре методы querySelector\[All\] обзавелись вторым параметром, но только в реализации для `document`, а также добавились новые методы find\[All\].

Второй параметр позволяет указать контекст, в котором мы будет искать ноды по селектору. Например:

    document.findAll("li", [ulElement1, ulElement2])
    
Найдёт все элементы `<li>` в двух элементах _ulElement1_ и _ulElement2_.

Псевдо класс _:scope_ позволяет делать действительно сласные вещи. Это ещё один способ указать контекст поиска - он указывает на текущий элемент, в которорм  мы проводим поиск по селекторы.
Он позволяет искать по ранее невалидным селекторам ">.class" или "~tagname". Просто укажите _:scope_ вначале и данные селекторы станут валидными. Вся мощь _:scope_ видна, когда
мы применяем его вместе с NodeRef:

    document.findAll(":scope > p", [divElement1, divElement2])

Найдёт всех _непосредственных_ детей с тэгом _p_ у _divElement1_ и _divElement2_!

## Element.prototype.matches

## [classList](https://developer.mozilla.org/en-US/docs/DOM/element.classList)

DOM API для работы с CSS-классами элемента.

Методы:

* `add(class{string})` Добавляет _class_ в **element.className**
* `remove(class{string})` Удаляет _class_ из **element.className**
* `toggle(boolean: class{string})` Удаляет _class_ в случеи его наличия из **element.className** и возвращяет **false**. Добавляет в случае его отсутсвие в **element.className** и возвращяет **true**.
* `contains(boolean: class{string})` Проверяет _class_ на его наличие в **element.className**. Возвращяет **true** или **false** соответственно.

## Events Constructor

В новом стандарте DOM API определены конструкторы для событий. Теперь мы можем забыть про мучения с

    event = document.createEvent("click")
    event.initMouseEvent( /*тут что-то, я даже сам не помню что*/ )

А просто написать:

    event = new Event("click")

В конструктор мы можем передать любое текстовое значение:

    event = new Event("some:event")
    event = new Event("dbclick1")

А необходимые данные для обработчика просто добавить в объект `event`:

    event = new Event("click")
    event.button = 1

Замечу, что при этом все созданные таким образом события являются _обычными_ событиями, те `new Event("click")` создаст не **MouseEvent**, а просто **Event**.

CustomEvent
KeyboardEvent

## DOM Keyboard Events Level 3

## Array.prototype

Ещё в редакции ES5 в Array.prototype были добавлены несколько полезных методов. Наверняка многие уже вовсю их используют, поэтому я просто дам список и покажу
наиболее ??? методы использования

    object = {a:1, b:2};
    newObject = Object.keys(object).reduce(function(val, key) { val[key] = this[key]; return val }.bind(object), {c:3, d:4})

## Array.from/Array.of

## Object.keys

## Объект в качестве event handler в addEventListener

Да-да, вы не ослышались в `addEventListener` можно передать объект, и он даже будет обрабатывать события при наличии у него метода 'handleEvent`. Что самое интересное,
метод 'handleEvent` может быть в прототипе этого объекта, что даёт нам большой выигрышь по использованию памяти.

Также это позволяет делать что-то наподобии event namespace из jQuery. Например, каждый объект может обрабатывать свою пачку событий.

    objMouseHandler = {
        handleEvent : function(e) {
            switch(e.type) {
                "click":
                    // do something
                break;
                "mousedown":
                    // do something
                break;
                "mouseup":
                    // do something
                break;
            }
        }
    }
    element.addEventListener("click", objMouseHandler)
    element.addEventListener("mousedown", objMouseHandler)
    element.addEventListener("mouseup", objMouseHandler)

    dndInterface = {
        handleEvent : function(e) {
            switch(e.type) {
                "dragstart":
                    // do something
                break;
                "dragover":
                    // do something
                break;
                "drop":
                    // do something
                break;
            }
        }
    }
    element.addEventListener("dragstart", dndInterface)
    element.addEventListener("dragover", dndInterface)
    element.addEventListener("drop", dndInterface)

На самом деле, это всё можно обернуть в паттерн, от чего данный метод становиться ещё более простой и гибкий в использовании. Пример:

## HTMLLabelElement.prototype.control

## HTML*Element.prototype.labels для элементов формы

## HTMLOlElement.prototype.revert

## Event.prototype.stopImmidiatePropagation

## String.prototype

    repeat
    startsWith
    endsWith
    contains
    toArray
    reverse

## Number.prototype

    isFinite
    isInteger
    isNaN
    toInteger
