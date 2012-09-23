# Web Components Explained

Статус: `Эксперементальный драфт`

Авторы:

 * Dominic Cooney, Google, dominicc@google.com
 * Dimitri Glazkov, Google, dglazkov@chromium.org

## Введение

Компонентная модель для Web'а (или Web Components) состоит из четырёх модулей, которые, будучи использованными вместе, позволят разработчикам web-приложений создавать виджеты с богатыми визуальными возможностями, легкие в разработке и повторном использовании, что в данный момент невозможно с использованием только CSS и JS-библиотек.

Модули:

 * шаблоны ([templates](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#template-section)): определяюют часть неактивной разметки, которая может быть использована в дальнейшем;
 * декораторы ([decorators](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#decorator-section)): позволяют с помощью CSS управлять визуальными и поведенческими изменениями в шаблонах;
 * пользовательские элементы ([custom elements](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#custom-element-section)): позволяют авторам определить их собственные _элементы_ (включая представление и API этих элементов), которые могут быть использованы в основном HTML-документе;
 * скрытый DOM ([shadow DOM](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#shadow-dom-section)): определяет, как представление и поведение _декораторов_ и _пользовательские элементы_ сочетаются друг с другом в дереве DOM.

Вместе _декораторы_ и _пользовательские элементы_ называются **компонентами**

## Шаблоны

Элемент `<template>` содержит разметку, предназначенную для использования позже с помощью скрипта или другого модуля, который может использовать шаблон (например, `<decorator>` и `<element>`, которые описаны ниже).

Содержимое элемента `<template>` анализируется парсером, но оно неактивно: скрипты не запускаются, изображения не загружаются и т. д. Элемент `<template>` не выводится.

В скрипте такой элемент имеет специальное свойство **content**, которое содержит статическую DOM-структуру, определённую в шаблоне.

Например, разработчику может понадобиться определить DOM-структуру, которая создается несколько раз в документе, а затем создать его экземпляр, когда это будет необходимо.

    <template id="commentTemplate">
        <div>
            <img src="">
            <div class="comment"></div>
            …
        </div>
    </template>

    var t = document.querySelector("#commentTemplate");
    // Populate content and img[src] values in the template.
    // Заполнить содержание и значения img[src] в шаблоне.
    // …
    someElement.appendChild(t.content.cloneNode());

Добавление статического DOM-узла в документ делает его "живым", как если бы этот DOM-узел был получен через свойство **innerHTML**.

## Декораторы

Декоратор это нечто, что улучшает или переопределяет представление существующего элемента. Как и все аспекты представлений, поведение декораторов контролируется с помощью CSS. Однако, возможность определять дополнительные аспекты представления, используя разметку - уникальная черта декораторов.

Элемент `<decorator>` содержит элемент `<template>`, который определяет разметку используемую для рендеринга декоратора.

    <decorator id="fade-to-white">
        <template>
            <div style="position: relative;">
                <style scoped>
                    #fog {
                        position: absolute;
                        left: 0;
                        bottom: 0;
                        right: 0;
                        height: 5em;
                        background: linear-gradient(
                        bottom, white 0%, rgba(255, 255, 255, 0) 100%);
                    }
                </style>
                <content></content>
                <div id="fog"></div>
            </div>
        </template>
    </decorator>

Элемент [`<content>`](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#content-element) указывает на место, куда декоратор (точнее, его содержимое) должен быть вставлен.

Декоратор применяется, используя CSS-свойство **decorator**:

    .poem {
        decorator: url(#fade-to-white);
        font-variant: small-caps;
    }

Декоратор и CSS, описанные выше, создадут такую разметку:

    <div class="poem">
        Two roads diverged in a yellow wood,<br>
        …
    </div>

Но её рендер будет как у этой разметки (стили user agent опущены для краткости):

    <div class="poem" style="font-variant: small-caps;">
        <div style="position: relative;">
            Two roads diverged in a yellow wood,<br>
            …
            <div style="position: absolute; left: 0; …"></div>
        </div>
    </div>

Если документ изменился так, что CSS-селектор, где был объявлен декоратор, более не действителен - обычно, когда селектор со свойством **decorator** более не применяется к элементу или правило с декоратором было изменено в атрибуте **style**, декоратор более не применяется, возвращая рендеринг элемента в первоначальное состояние.

Даже несмотря на то, что CSS-свойство **decorator** может указывать на любой ресурс в сети, декоратор не будет применяться, пока его определение загружается в текущий документ.

Разметка, которая генерируется представлениями, ограничивается чисто презентационным применением: она никогда не может запустить скрипт (в том числе встроенные обработчики событий), и она не может быть доступна для [редактирования](http://www.whatwg.org/specs/web-apps/current-work/multipage/editing.html#editing-0).

### События в декораторах

Декораторы также могут навешивать обработчики событий для реализации интерактивности. Поскольку декораторы _преходящи_,  не эффективно навешивать обработчики событий на ноды в шаблоне или полагаться на любое состояние шаблона, так как ноды в шаблоне _пересобираются_ каждый раз как декоратор применяется или снимается с элемента.

Вместо этого, декораторы регистрируют обработчики событий у контроллера событий. Чтобы зарегистрировать обработчик событий, шаблон включает в себя элемент `<script>`. Скрипт запускается один раз, когда декоратор парсится или вставляется в документ, или загружается как часть внешнего документа.

Рис. Регистрация обработчиков событий

![Регистрация обработчиков событий](http://www.habrastorage.com/images/eventhfvf.png "Регистрация обработчиков событий")

Контроллер событий будет передан в скрипт в качестве значения **this**.

    <decorator id="decorator-event-demo">
        <script>
            function h(event) {
                alert(event.target);
            }
            this.listen({selector: "#b", type: "click", handler: h});
        </script>
        <template>
            <content></content>
            <button id="b">Bar</button>
        </template>
    </decorator>

Вызов функции **lisnen** означает, что когда кнопка будет нажата, сработает обработчик события.

Контроллер событий перенаправит событие, наступившее в на любой ноде, на которой декоратор был применён, в обработчик события.

Рис. Обработка и переназначение событий

![Обработка и переназначение событий](http://www.habrastorage.com/images/eventrpcp.png "Обработка и переназначение событий")

Когда слушатель событий вызывается, значением **target** события является нода, на которую декоратор был применен, а не содержимое его шаблона. Например, если декоратор, указанный выше, такого содержания:

    <span style="decorator: url(#decorator-event-demo);">Foo</span>

Рендерится в:

    Foo[Bar]

Клик по кнопке покажет сообщение с `[object HTMLSpanElement]`.

Переопределение свойства **target** необходимо, тк декоратор определяет отображение; он не влияет на структуру документа. Пока декоратор применён, свойство **target** переопределяется на ноду, на которую он применён.

Также, если скрипт меняет контент шаблона, изменения игнорируются, точно также, как установка **textContent** элемента `<script>` не влечет за собой  выполнение скрипта ещё раз.

Декоратор не может никак изменить свой шаблон и повлиять на отображение самого себя на элемент, он может только переопределить декоратор на другой.

### Примеры декораторов

Пример, как декоратор может быть применён для создания простого варианта элемента **detail**:

    details {
        decorator: url(#details-closed);
    }
    details[open] {
        decorator: url(#details-open);
    }

    <decorator id="details-closed">
        <script>
            this.listen({
                selector: "#summary", type: "click",
                handler: function (event) {
                    event.currentTarget.open = true;
                }
            });
        </script>
        <template>
            <a id="summary">
                &gt; <content select="summary:first-of-type"></content>
            </a>
        </template>
    </decorator>

    <decorator id="details-open">
        <script>
            this.listen({
                selector: "#summary", type: "click",
                handler: function (event) {
                    event.currentTarget.open = false;
                }
            });
        </script>
        <template>
            <a id="summary">
                V <content select="summary:first-of-type"></content>
            </a>
            <content></content>
        </template>
    </decorator>

Для этого понадобилось два декоратора. Один представляет **detail** элемент в закрытом виде, другой в открытом. Каждый декоратор использует обработчик события клика мыши, для изменения состояния открыт/закрыт. Атрибут **select** элемента `<element>` будет рассмотрен подробнее ниже.

## Пользовательские элементы

Пользовательские элементы - новый тип DOM-элементов, которые могут быть определены автором.

Пользовательские элементы могут определять вид отображения через декораторы. В отличии от декораторов, которые могут быть применены или сняты на нужный элемент, тип пользовательского элемента фиксирован. Но пользовательские элементы могут определять совершенно новое отображение и поведение, которые нельзя определить через декораторы, из-за элементарной природы последних.

Элемент `<element>` определяет пользовательский элемент.

    <element extends="button" name="x-fancybutton">
        …
    </element>

Атрибут **extends** определяется элемент, функционал которого мы хотим расширить. Каждый экземпляр пользовательского элемента будет иметь **tagName** определённый в атрибуте **extends**.

Атрибут **name** определяет пользовательский элемент, который будет связан с этой разметкой. Пространство имён у атрибута **name** такое же, как у имён тэгов стандартных элементов, поэтому для устранения коллизий, используется префикс x-.

Разные браузеры, определяют HTML элементы по-разному, однако все их интерпретации руководствуются семантикой HTML.

Тк не все браузеры поддерживают пользовательские элементы, авторы должны расширять HTML элементы, которые имеют наиболее близкое значение для нового пользовательского элемента. Например, если мы определяем пользовательский элемент, который является интерактивным и реагирует на клики, выполняя некоторые действия, мы должны расширять кнопку (`<button>`).

Когда нету HTML элемента, семантически близкого к нужному, автор должен расширять нейтральный элемент, такой как `<span>`.


### Представление

Пользовательский элемент может содержать шаблон:

    <element extends="button" name="x-fancybutton">
        <template>
            <style scoped>
                ::bound-element { display: transparent; }
                div.fancy {
                    …
                }
            </style>
            <div class="fancy">
                <content></content>
                <div id="t"></div>
                <div id="l"></div>
                <div id="b"></div>
                <div id="r"></div>
            </div>
        </template>
    </element>

Если пользовательский элемент содержит шаблон, копия этого шаблона будет вставлена в теневой DOM элемента конструктором пользовательских элементов.

Теневой DOM будет описан ниже.

### Использование пользовательских элементов в разметке

Т.к. пользовательские элементы использую существующие HTML тэги (div, button, option и тд), нам нужен атрибут для определения когда мы хотим использовать пользовательский элемент. Таким атрибутом является **is**, а значением его является название пользовательского элемента. Например:

    <element extends="button" name="x-fancybutton">  <!-- definition -->
        …
    </element>

    <button is="x-fancybutton" onclick="showTimeClicked(event);">  <!-- use -->
        Show time
    </button>


### Использование пользовательских элементов в скриптах

Вы можете создать пользовательский элемент из скрипта, используя стандартный метод **document.createElement**:

    var b = document.createElement("x-fancybutton");
    alert(b.outerHTML); // will display '<button is="x-fancybutton"></button>'

Также, вы можете установить атрибут **constructor** у элемента `<element>`, чтобы явно указать название конструктора элемента, которое будет экспортировано в объект **window**. Этот конструктор может быть использован для создания пользовательского элемента:

    <element extends="button" name="x-fancybutton" constructor="FancyButton">
        …
    </element>

    …

    var b = new FancyButton();
    document.body.appendChild(b);

Пользовательский элемент может объявлять методы API добавляя их в свой **prototype**, в элементе `<script>`, находящимся внутри элемента `<element>`:

    <element extends="button" name="x-fancybutton" constructor="FancyButton">
        …
        <script>
        FancyButton.prototype.razzle = function () {
            …
        };
        FancyButton.prototype.dazzle = function () {
            …
        };
        </script>
    </element>

    …

    <script>
        var b = new FancyButton();
        b.textContent = "Show time";
        document.body.appendChild(b);
        b.addEventListener("click", function (event) {
            event.target.dazzle();
        });
        b.razzle();
    </script>

Для того, чтобы обеспечить простую деградацию, **this** внутри элемента `<script>` указывает на родительский элемент типа **HTMLElementElement**:

    <element extends="table" name="x-chart" constructor="Chart">
        <script>
            if (this === window)
                // Use polyfills to emulate custom elements.
                // …
            else {
                // …
            }
        </script>
    </element>

Технически, скрипт внутри элемента `<script>`, когда он является вложенным в `<element>` или `<decorator>`, выполняется идентично такому вызову:

    (function() {
        // code goes here.
    }).call(parentInstance);

В ситуациях, когда название конструктора в объекте **window** неизвестно, автор пользовательского компонента может воспользоваться свойством **generatedConstructor** у **HTMLElementElement**:

    <element extends="table" name="x-chart">
        <script>
            // …
            var Chart = this.generatedConstructor;
            Chart.prototype.shizzle = function() { /* … */ };
            // …
        </script>
    </element>

Нельзя создать пользовательский элемент указывая **is** атрибут у существующего DOM-элемента. Выполнение следующего кода ничего не даст:

    var div = document.createElement("div");
    div.setAttribute("is", "foo");
    alert(div.is); // displays null
    alert(div.outerHTML); // displays <div></div>

### Обновление элемента

Когда объявление пользовательского элемента будет загружено, каждый элемент с атрибутом **is** установленном в имя пользовательского элемента будет обновлён до пользовательского элемента. Обновление должно быть идентично удалению элемента и замене его на пользовательский элемент.

Когда каждый элемент заменяется, не всплывающее (non-bubbling), неотменяемое (non-cancellable) событие возникает на удаляющемся элементе. Скрипт, который хочет задержать взаимодействие с остальной часть документа до тех пор, пока пользовательский элемент не загрузиться может подписаться на специальное событие:

    function showTimeClicked(event) {
        // event.target may be an HTMLButtonElement or a FancyButton

        if (!event.target.razzle) {
            // razzle, part of the FancyButton API, is missing
            // so upgrade has not happened yet
            event.target.addEventListener('upgrade', function (upgradeEvent) {
                showTime(upgradeEvent.replacement);
            });
            return;
        }

        showTime(event.target);
    }

    function showTime(b) {
        // b is FancyButton
    }

Авторы, которые хотят избежать показа нестилизованного контента, могут использовать CSS для изменения отображения не заменённого, обычного элемента, пока пользовательский элемент не загрузиться.

### Методы жизненного цикла

Пользовательский элемент может подписаться на четыре метода жизненного цикла:

 * created - вызов конструктора, создание экземпляра **ShadowRoot** из элемента `<template>`. При отсутствии `<template>`, **ShadowRoot** устанавливается в *null*.
 * attributeChanged - вызывается всякий раз, когда атрибут элемента изменяется. Аргументы: имя, старое значение, новое значение.
 * inserted - вызывается, после того, как пользовательский компонент вставлен в DOM-дерево. В данном обработчике можно подгрузить ресурсы для пользовательского конпонента или запустить таймеры.
 * removed - вызывается после того, как пользовательский элемент удалён из DOM-дерева. Тут можно остановить таймеры, которые не нужны, когда пользовательский элемент не находится в DOM-дереве.

Обработчики вызываются с **this** указывающим на элемент.

Обработчики inserted и removed могут быть вызваны несколько раз, каждый раз, когда пользовательский элемент вставляется и удаляется.

Подписаться на эти обработчики можно вызвав метод **HTMLElementElement.lifecycle**:

    <element extends="time" name="x-clock">
    <script>
    // …

            this.lifecycle({
                inserted: function() { this.startUpdatingClock(); },
                removed: function() { this.stopUpdatingClock(); }
            });

    // …
    </script>
    </element>

### Расширение пользовательских элементов

В дополнении к HTML-элементам, можно расширить пользовательский элемент указанием имени пользовательского элемента в атрибуте **extends** элемента `<element>`:

    <element extends="x-clock" name="x-grandfatherclock">
        …
    </element>

### Обработка ошибок

Есть несколько возможностей для обработки ошибок при рендеринге пользовательских элементов:

 * **tagName** элемента не соответствует значению атрибута **extends** пользовательского элемента, например: `<div is="x-fancybutton">`, но `<element name="x-fancybutton" extends="button">`. В этом случае, атрибут **is** отбрасывается во время парсинга.
 * Значение атрибута **is** не соответствует никакому существующему элементу. Эта ситуация рассматривается как если бы определение пользовательского элемента ещё не было загружено, и элемент будет обновлён​​, как только определение загрузится.
 * Значение атрибута **extends** не соответствует никакому существующему элементу. В этом случае обработка определения пользовательского элемента будет удерживаться до тех пор, пока элемент обозначенный в атрибуте **extends** не загрузится.
 * Значение атрибута **is** не начинается с _x-_. В данном случае, атрибут **is** игнорируется.
 * Значение атрибута **name** не начинается с _x-_. В данном случае, определение пользовательского элемента считается невалидным и игнорируется.

## Shadow DOM

Shadow DOM is a tree of DOM nodes. Shadow DOM can be applied to a custom element declaratively, by including a template, or to any element imperatively using JavaScript. 

When an element has shadow DOM, the element's children are not rendered; the content of the shadow DOM is rendered instead. 
Content and Shadow Elements

Any shadow DOM subtree can specify insertion points for element's children by using a <content> element: 

    <div>
        <div class="stuff">
            <content><!-- all children will appear here --></content>
        </div>
    </div>

The `<content>` element acts as an insertion point for rendering only—it does not change where the elements appear in DOM. When rendering, the element's children just appear in place of the <content> element. And yes, you can have more than one <content> in the shadow DOM subtree! The select attribute gives you a handy way to choose which children appear at which insertion point. If there is more than one <content> element, the document order dictates when each of these elements takes a pass at selecting the children. Once the child is selected to be rendered at one insertion point, it can't be claimed by another one.

    <div>
        <content select="h1.cool"><!-- all h1.cool children will
            appear here --></content>
        <div class="cool">
            <content select=".cool"><!-- all .cool children
                (except the ones that are h1.cool) appear here --></content>
        </div>
        <div class="stuff">
            <content><!-- all remaining children will
                appear here --></content>
        </div>
    </div>

Multiple Shadow Subtrees

Any element can have more than one shadow DOM subtree. Don't look so puzzled! Suppose you are extending a DOM element that already has a shadow DOM subtree. What happens to that poor old tree? We could just ditch it for the new arboreal hotness, but what if you don't want to? What if you want reuse it? 

For this noble purpose, we have a special <shadow> element, which acts as an insertion point for the, you guessed it, the previously applied shadow DOM subtree of this element. For instance, you can take existing range slider (that's <input type="range">) and extend it, adding your extra shadow schmutz to it: 

    <div>
        <shadow><!-- range slider renders here --></shadow>
        <div class="ticks"> … </div>
    </div>

Since an element can have multiple shadows, we need to understand how these shadows interact with each other and what effect these interactions have on rendering of the element's children. 

First, the order in which the shadow DOM subtrees are applied is important. Because you cannot remove a shadow DOM, the order is typically like this: 
element shadow DOM (I got here first, nyah!) 
shadow DOM of the element sub-class 
shadow DOM of the element sub-sub-class 
… 
some shadow DOM added using script 
decorator shadow (applied and removed with CSS rules—not technically a shadow DOM, but can, too, have <content> and <shadow> elements) 

Next, we take this list and traverse it backwards, starting with the last-applied shadow subtree. Each <content> instance, encountered in document order, grabs the element's children that it needs as usual. 

This is where things get interesting. Once we're done shuffling the children into their right places to render, we check and see if we have a <shadow> element. If we don't, we're done. 

If we do, we plop the next shadow subtree in our list in place of the <shadow> element, and rinse-repeat first replacing <content> insertion points, then <shadow> until we reach the start of the list. 

And then—ta-da!—we have our wondrous shadow DOM Yggdrasil, ready for rendering. Here's an easy way to remember how this works: 
The most recently applied shadow DOM subtree has the best shot of getting fresh children for its <content> insertion points (unfair, I know!) 
Once it's had its way, the next most recent shadow DOM subtree—if even allowed to—can rummage through the remaining children. 
Cycle repeats until either the current shadow DOM subtree has no <shadow> element, or we've processed the oldest DOM subtree for this element. 
CSS and Shadow DOM

When building a custom element, you often want to prevent document styles from interfering with what you've painstakingly crafted. 

Lucky for you, the shadow DOM subtree is surrounded by an invisible boundary, only letting the user agent styles apply by default. The inheritance still works as usual. 

You can relax this boundary with the apply-author-styles attribute on the <template> element. With the attribute present, document's author styles start applying in the shadow DOM subtree. 

There is a similar boundary between the <content> element and element's children, rendered in its place. The styles from the shadow DOM subtree do not apply to the element's children. 

The magic style boundary also prevents selectors from reaching into the shadow DOM subtree, skipping it altogether. For example, if <div id="foo"> had this shadow DOM subtree: 

    <a href="http://example.com">shadow link</a>
    <content></content>

and a child 

    <a href="http://example.org">light link</a>
,

both div#foo>a and div#foo a will select the child (light link), not the anchor in the shadow DOM (shadow link). 

If the component authors wish to allow some styling of the shadow DOM subtree by the component users, they can do so with CSS variables. The variable definitions specified on the shadow DOM host element are also resolved inside of the shadow DOM subtree. 
Events

To ensure that the elements of the shadow DOM subtree are not exposed outside of the subtree, there's a fair bit of work that happens while dispatching an event from inside of the subtree. 

First, some events (like mutation and selectstart events) are just plain prevented from escaping out of the shadow DOM subtree—they are never heard from the outside. 

Those events that do cross the shadow DOM boundary are retargeted—their target and relatedTarget values are modified to point to the element that hosts the shadow DOM subtree. 

In some cases, like DOMFocusIn, mouseover, mouseout events have to be given extra attention: if you're moving a mouse between two elements inside of the shadow subtree, you don't want to be spamming the document with these events, since they will appear as non-sensical babbling after the retargeting (what? the element just reported that the mouse just left it to itself?!) 
Making More Shadows

As implied before, you can always add a shadow DOM subtree to any DOM element in one line of code: 

    var shadow = new ShadowRoot(anyElement);

This does two things: 
Creates a brand-spanking-new ShadowRoot instance, which you should imagine as this shadowy, invisible other side of your element. It's like a tiny document that lives inside of our custom element, and serves as the root of your shadow DOM subtree. 
Sets anyElement as the host of this newly created shadow DOM subtree. 

The important thing to remember is that while you can add new shadow DOM subtrees this way, you can never remove them: they just keep stacking up. 
Decorators vs. Custom Elements

Decorator shadows are never accessible with DOM. When the decorator is created, the shadow DOM subtree is placed in a cold, dark place—for storage. Once the decorator is applied to an element, this shadow DOM subtree is used to produce the rendering, but its nodes are never accessible from script. You can, however listen to events that happen inside the decorator shadow using EventController. 

An easy way to think of the decorator shadow is as a projection. You can see it, but never can actually touch it. 

On the other hand, the custom element shadow (or any shadow DOM subtree created using script) is a real DOM tree that you as the custom element author can change as you please. 

## External Custom Elements and Decorators

Пользовательские элементы и декораторы могут быть загружены из внешних файлов используя тэг `<link>`:

    <link rel="components" href="goodies.html">

Только элементы `<decorator>`, `<element>` и `<script>` парсятся браузером.
DOM текущего элемента не доступен в скриптах загруженных таким образом. _Origin_ внедряющего документа используется как _origin_ внедряемого документа.
Скрипты запускаются так, как будто у них есть атрибут **defer**. Объявления пользовательских элементов и декораторов недоступны, пока скрипты не выполнятся.
Документы полученные кросс-доменно, используют [**CORs**](http://www.w3.org/TR/cors/) для определения полномочий.

## Изоляция, ограничения и инкапсуляция

Конпонентную изоляцию можно рассматривать как комбинацию из двух частей:

 * ограничения, когда документ ограничивает компонент в доступе к информации содержащийся в документе, и
 * инкапчуляция, когда компонент ограничивает документ в доступе к состояниям конпонента

Инкапсуляция компонента частично навязывается использованием **shadow DOM**, гарантируя что ноды **shadow DOM** никогда не появятся в возникающих событиях элемента, в range _(Прим. переводчика: возможно имеется ввиду [Document Object Model Range](http://www.w3.org/TR/DOM-Level-2-Traversal-Range/ranges.html))_, focus-related DOM accessors _(Прим. переводчика: возможно имеется ввиду изменения состояния элемента связанные с получением фокуса)_ или в любых _API_ которые выбирают элементы. Только автор компонента определяет, будет ли внутреннее состояние конпонента влиять на состояние элемента.

Ограничение компонента сложнее, но возможно использование существующих примитивов, таких как `<iframe>` или **Worker**. Лучше всего, если компонент будет восприниматься как небезопастный но ответственный за элемент пользовательского интерфейса. Компонент, в данном случае, может загрузить **Worker** (или `<iframe>`) и общятся с ним используя *postMessage*:

    <!--- located at http://example.com/adorable/button -->
    <element name="x-adore-button" extends="button">
        <script>
            var service = new Worker("http://example.com/adorable/service/worker.js");
            service.onmessage = function(event) {
                // process responses from the service.
                …
            }
            // set up UI.
            …
            // start interacting with the worker.
            service.postMessage("hello");
        </script>
        <template>I adore this</template>
    </element>

## Примеры

Предположим, что мы хотим показывать кликабельную карту тайм-зон вместо элемента `<select>`. Вот как это можно сделать используя декораторы:

    <decorator id="timezone-map">
        <script>
            var timezones = ["PST", … ];

            function createHandler(timezone)
            {
                this.listen({
                    selector: "#" + timezone,
                    type: "click",
                    handler: function() {
                        // Decorators are late-bound, we must check for type.
                        if (this instanceof HTMLSelectElement)
                            this.selectedIndex = this[timezone].index;
                    }
                });
            }

            timezones.forEach(createHandler, this);
        </script>
        <template>
            <svg …>
                <g id="worldShapes"> … </g>
                <g id="timezoneShapes">
                    <path id="PST" …>
                    <path id="CST" …>
                </g>
                …
            </svg>
        </template>
    </decorator>

Вы можете применить этот декоратов так:

    select.timezones {
        decorator: url(#timezone-map);
    }

provided that there's the following markup in the document: 

    <select class="timezones">
        <option>PST</option>
        …
    </select>

при условии, что есть следующая разметка в документе:

    <element extends="select" name="x-timezone-map" constructor="MapSelector">
        <script>
            this.lifecycle({
                created: function(shadowRoot) {
                    var select = shadowRoot.shadowHost;
                    var timezones = shadowRoot.querySelector("#timezoneShapes");
                    timezones.addEventHandler("click", function() {
                        if (event.target != this)
                            select.selectedIndex = select[event.target.id].index;
                    });
                }
            });

            MapSelector.prototype.isDaylightSaving = function() { /* … */ }
        </script>
        <template>
            <svg>
                <g id="worldShapes"> … </g>
                <g id="timezoneShapes">
                    <path id="PST" …>
                    <path id="CST" …>
                </g>
                …
            </svg>
        </template>
    </element>

Вы можете использовать этот пользовательский элемент примерно так:

    <select is="x-timezone-map">
        <option>PST</option>
        …
    </select>

Больше примеров по ссылке [Web Components Samples](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/samples/index.html).

Приложение "A". Интерфейсы

    partial interface HTMLElement {
        readonly attribute DOMString is;
    }

    interface HTMLTemplateElement : HTMLElement {
        readonly attribute DocumentFragment content;
    }

    interface HTMLDecoratorElement : HTMLElement {
        attribute bool applyAuthorStyles;
        void listen(ListenInit listenInitDict);
    }
    dictionary ListenInit {
        DOMString type;
        DOMString selector;
        EventHandler handler;
    }

    interface HTMLContentElement : HTMLElement {
        attribute DOMString select;
    }

    interface HTMLElementElement : HTMLElement {
        attribute DOMString extends;
        attribute DOMString name;
        attribute bool applyAuthorStyles;
        readonly attribute Function generatedConstructor;
        void lifecycle(LifecycleInit lifecycleInitDict);
        void reflectAttribute(DOMString attributeName,
        DOMString propertyName, ChangeCallback changeCallback);
        void setupEvent(DOMString eventName);
    }
    dictionary LifecycleInit {
        Function attributeChanged(
                DOMString name,
                DOMString newValue,
                DOMString oldValue);
        Function created(ShadowRoot shadowRoot);
        Function inserted();
        Function removed();
    }
    [Callback=FunctionOnly, NoInterfaceObject]
    interface ChangeCallback {
        void handleEvent(DOMString name, DOMString newValue, DOMString oldValue);
    }

    interface HTMLShadowElement : HTMLElement {
    }

    [Constructor(in HTMLElement element) raises (DOMException)]
    interface ShadowRoot : Node {
        Element getElementById(in DOMString elementId);
        NodeList getElementsByClassName(in DOMString tagName);
        NodeList getElementsByTagName(in DOMString className);
        NodeList getElementsByTagNameNS(DOMString namespace, DOMString localName);
        attribute bool applyAuthorStyles;
        readonly attribute Element host;
    }
    ShadowRoot implements NodeSelector;

## Appendix B. HTML Elements
`<content>`

> Определяет [точку вставки](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-insertion-point) в [shadow DOM subtree](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-shadow-dom-subtree). Точка вставки будет заменена детьми элемента в момент рендеринга. Элемент `<content>` сам никогда не рендерится.

> Контекст: Там, где ожидается содержимое потока.

> Дети: любой элемент (_fallback_ контент).

> Атрибуты:

> select - список [_фрагментрых_ CSS-селекторов](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-selector-fragment) выбирающих непосредственных детей элемента.

`<decorator>`

> Определяет новый декоратор. В большенстве случаев, вы также определите для него атрибут **id** для указания на него из CSS.

> Контекст: В любом месте документа.

> Дети: `<template>` and `<script>`

> Атрибуты:

> apply-author-styles - Если _true_, то стили документа будут применятся на [shadow DOM subtree](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-shadow-dom-subtree) элемента. Если _false_ (по-умолчанию), стили документа не применяются.

`<element>`

> Определяет новый пользовательский элемент.

> Контекст: В любом месте документа.

> Дети: `<template>` and `<script>`

> Атрибуты:

>> constructor - имя функции порождаемого конструктора. Если указано, по этому ключу в объекте **window** будет доступна функция-порождённый конструктор элемента.

>> extends - **tagName** элемента или название конструктора пользовательского элемента, который будет расширять новый пользовательский элемент.

>> name - имя конструктора пользовательского элемента, порождаемого данным определением _(Прим. переводчика: возможно имеется ввиду, имя которое будет использоваться в document.createElement)_

>> apply-author-styles - Если _true_, то стили документа будут применятся на [shadow DOM subtree](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-shadow-dom-subtree) элемента. Если _false_ (по-умолчанию), стили документа не применяются.

`<shadow>`

> Определяет [теневую точку вставки](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-shadow-insertion-point), где предыдущее [shadow DOM subtree](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-shadow-dom-subtree) в [shadow DOM дереве](http://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#dfn-tree-stack) элемента будет рендерится. Элемент `<shadow>` сам никогда не рендерится.

> Контекст: Там, где ожидается содержимое потока.

> Дети: любой элемент (_fallback_ контент).

`<template>`

> Определяет инертную DOM-структуру, которая в дальнеёшем может быть использована для создания DOM-объектов в неизвестном контексте.

> Контекст: В любом месте документа.

> Дети: любой элемент.

## Примечание "C". Свойства конструктора пользовательских элементов

Объяснение некоторых других свойств порождённого конструктора и экземпляров **FancyButton**:

    typeof FancyButton ⇒ "function"
    FancyButton.call({}) ⇒ throws
    Object.getPrototypeOf(FancyButton.prototype) ===
       HTMLButtonElement.prototype ? true

    document.body.lastChild === b ⇒ true
    b.tagName ⇒ "button"
    b.outerHTML ⇒ "<button is="FancyButton"></button.html>"
    b instanceof FancyButton ⇒ true
    b instanceof HTMLButtonElement ⇒ true

## Благодарности

Thanks to Alex Komoroske, Alex Russell, Darin Fisher, Dirk Pranke, Divya Manian, Erik Arvidsson, Hayato Ito, Hajime Morita, Ian Hickson, Jonas Sicking, Rafael Weinstein, Roland Steiner, and Tab Atkins for their comments and contributions to this document. 
See a problem? Select text and  or view bugs filed.
