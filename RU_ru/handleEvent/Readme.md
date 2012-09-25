## handleEvent. Взглянем на обработку событий по-новому

Мало кто знает, но уже в спецификации DOM Level 2 была введена возможность передать [объект в качестве обработчика событий](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-EventListener)
в **addEventListener**.

Да-да, вы не ослышались. Такой код вполне валиден:

    document.addEventListener("click", [])

Но для того, чтобы собыет обработать нужна функция. Код выше не сдалает ничего, но если мы добавиим в объект специальный
метод **handleEvent**, то браузер найдёт и вызовет его в качестве обработчика события. Например:

    document.addEventListener("click", { handleEvent : console.log.bind(null, 1) })

Для **handleEvent** не нужно _закреплять_ **this** (например, с помошью **Function.prototypy.bind**), т.к. при вызове
**handleEvent** браузер не переопределяет **this** и он будет указывать на объект, который был передан в **addEventListener**.


## IE ниже 9й версии

Вы можете просто отказаться от поддержки IE < 8, как это [уже делают крупнейшие интернет-сервисы](http://habrahabr.ru/post/151536/),
или вы можете использовать обёртку или polyfill, например такую:

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
