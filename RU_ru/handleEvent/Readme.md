## Объект в качестве обработчика в addEventListener. Взглянем на обработку событий по-новому

Мало кто знает, но уже в спецификации DOM Level 2 была введена возможность передать [объект в качестве обработчика событий](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-EventListener) в **addEventListener**.

Да-да, вы не ослышались. Такой код вполне валиден:

    document.addEventListener("click", [])

Но для того, чтобы событие обработать нужна функция. Код выше не сдалает ничего, но если мы добавиим в объект специальный метод **handleEvent**, то браузер его найдёт и вызовет в качестве обработчика события. Например:

    document.addEventListener("click", { test : "тест", handleEvent : function(){ console.log(this.test) } })

Для **handleEvent** не нужно _закреплять_ **this** (например, с помошью **Function.prototype.bind**), т.к. при вызове **handleEvent** браузер не переопределяет **this** и он будет указывать на объект, который был передан в **addEventListener**.

Использование **handleEvent** позволяет делать что-то наподобии event namespace из jQuery. Например, каждый объект может обрабатывать свою пачку событий.

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

На самом деле, это всё можно обернуть в паттерн, от чего данный метод становиться ещё более простой и гибкий в использовании:

    /**
     * @constructor
     * @param {Node} node */
    function Component(node) {
        this.node = node;
    }
    Component.prototype.handlerEvent = function(e) {
        var handlers = this["$" + e.type]
            , handler = (Array.isArray(handlers) ? handlers : (handlers = [handlers]))[0]
            , i = 0
        ;

        do {
            handler.call(this, e);
        }
        while(handler = handlers[++i])
    }
    Component.prototype.init = function() {
        for(var handler in this) if(this.hasOwnProperty(handler) && handler[0] == "$") {
            this.node.addEventListener(handler.substr(1), this);
        }
    }

    testComponent = new Component(document)
    testComponent.$DOMContentLoaded = function() {
        // do something
    }
    testComponent.$click = [
        function(e) {
            // do something
        }
        , function(e) {
            // do something
        }
    ]
    testComponent.init()

Как вы могли заметить, **handlerEvent** берётся из прототипа объекта и это позволяет существенно сократить потребление памяти.

## Плюсы

*


## IE ниже 9й версии

Вы можете просто отказаться от поддержки IE < 8, как это [уже делают крупнейшие интернет-сервисы](http://habrahabr.ru/post/151536/), или вы можете использовать обёртку или polyfill, например такую:

    // аргумент fn может быть либо функцией, либо объектом, благодаря обработчику handleEvent
    function on(el, evt, fn, bubble) {
        if(!("addEventListener" in el)) {
            // check if the callback is an object and contains handleEvent
            if(typeof fn == "object" && fn.handleEvent) {
                el.attachEvent("on" + evt, function(){
                    // Bind fn as this
                    fn.handleEvent.call(fn);
                });
            } else {
                el.attachEvent("on" + evt, fn);
            }
        } else  {
            el.addEventListener(evt, fn, bubble);
        }
    }

_(Код взят из статьи[addEventListener, handleEvent and passing objects](http://www.thecssninja.com/javascript/handleevent) и отредактирован. Исключён polyfill для BlackBerry OS 6.0)_

##
