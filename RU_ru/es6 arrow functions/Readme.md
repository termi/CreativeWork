Подробнее про Стрелочные функции (Arrow functions) в ECMAScript 6

Компиляция из вольного перевода статьи [Understanding ECMAScript 6 arrow functions](http://www.nczonline.net/blog/2013/09/10/understanding-ecmascript-6-arrow-functions/) и чтения последнего черновика [спецификации](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-arrow-function-definitions) (January 20, 2014 Draft Rev 22).

Одной из самых интересный частей нового стандарта ECMAScript 6 являются стрелочные функции.
Стрелочные функции, как и понятно по названию определяются новым синтаксисом, который использует стрелку `=>`.
Однако стрелочные функции отличаются от традиционных функций в некоторых моментах:
* Лексическое связывание - значения _специальных переменных_ **this**, **super** и **arguments**  определяются не тем,
как стрелочные функции были вызваны, а тем, как они были созданы.
* Стрелочные функции не могут быть использованы как конструктор и кидают ошибку при использовании с ключевым словом **new**
* Неизменяемые **this**, **super** и **arguments** - Значения этих переменных внутри стрелочные функции остаются неизменными
на протяжении всего жизненного цикла функции

Было несколько причин для введения этих отличий. Первоочередная - это то, что связывание (binding) используется довольно
часто в javascript. Очень легко потерять нужное значение **this** при использовании традиционных функций, что может
привести к непредсказуемым последствиям. Другая причина, это то, что JS-движки смогут легко оптимизировать выполнение
стрелочных функций, за счет этих ограничений (в противоположность традиционным функциям, которые могут быть использованы
в качестве конструктора и которые свободны для модификации _специальных переменных_).

## Синтаксис

Синтаксис стрелочных функций может быть различен, в зависимости от того, как вы объявляете функцию.
Объявление всегда начинается со списка аргументов, далее следует стрелка и тело функции. И список аргументов и тело
функции могут иметь различную форму, в зависимости от того, что вы пишете. Например, вот так объявляется стрелочная
функция которая принимает один аргумент и просто возвращает его:
```javascript
var reflect = value => value;
// эквивалент
var reflect = function(value) { return value; }
```
Когда у стрелочной функции только один аргумент, он может быть объявлен без скобок. Следующее после стрелки тело функции
также может быть без фигурных скобок и может не содержать ключевого слова **return**.

Но если вы хотите объявить более одного параметра, вы должны обрамить список параметров в круглые скобки. Например:
```javascript
var sum = (num1, num2) => num1 + num2;
// эквивалент
var sum = function(num1, num2) { return num1 + num2; };
```
Функция `sum` просто суммирует два аргумента. Единственное отличие от предыдущего примера в наличии круглых скобок и
запятой (прямо как в традиционных функциях). Аналогично, функция безо всяких аргументов, должна иметь пустой список
параметров обрамлённый в круглые скобки:
```javascript
var sum = () => 1 + 2;
// эквивалент
var sum = function() { return 1 + 2; };
```
Также, вы можете воспользоваться синтаксисом традиционных функций для тела стрелочной функции, когда оно содержит более
одного выражения - т.е. обернуть функцию в фигурные скобки и добавить ключевое слово **return**:
```javascript
var sum = (num1, num2) => { return num1 + num2; }
// эквивалент
var sum = function(num1, num2) { return num1 + num2; };
```
Тело функции будет обработано точно также, как и для классических функций, за исключением того, что значения
_специальных переменных_ **this**, **super** и **arguments** будут вычисляться по-другому.

Отдельно надо упомянуть, что тело функции которое не содержит фигурных скобок и просто возвращает литерал объекта,
должно быть забрано в круглые скобки. Например:
```javascript
var getTempItem = id => ({ id: id, name: "Temp" });
// эквивалент
var getTempItem = function(id) { return { id: id, name: "Temp" } };
```
Забор литерала объекта в круглые скобки означает, указывает парсеру, что фигурные скобки это не начало традиционного
синтаксиса для тела функции, а начало литерала.

Т.к. "собственный" объект **arguments** не доступен внутри стрелочной функции, для стрелочных функций с переменным числом
параметров нужно использовать **rest** паттерн из _шаблонов деструктуризации_. Пример:
```javascript
var getTempItems = (...rest) => rest;
// эквивалент
var getTempItems = function() { return [].slice.apply(rest) };
```
Как видите, несмотря на то, что у стрелочной функции всего один аргумент, всё равно необходимо использовать круглые
скобки с _шаблонов деструктуризации_. Примеры с другими _шаблонами_:
```javascript
var a = ({a}) => a;
var b = ([b]) => b;
```

## Использование

Одним из частых сценариев в JavaScript является установка правильного значения **this** внутри функции (связывание).
Поскольку значение **this** может быть изменено, в зависимости от контекста исполнения функции, возможно ошибочно
действовать на один объект, когда вы имели ввиду совершенно другой. Посмотрите на следующий пример:
```javascript
var PageHandler = {
    id: "123456"
    , init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // ошибка
        });
    }
    , doSomething: function(type) { console.log("Handling " + type  + " for " + this.id) }
};
```
В приведённом коже, объект `PageHandler` должен обрабатывать сообщения на странице. Метод `init()` навешивает обработчик
на нужное событие, который внутри себя вызывает `this.doSomething()`. Однако, код сработает неправильно. Ссылка на
`this.doSomething()` не валидна, поскольку **this** указывает на объект `document` внутри обработчика события вместо
планируемого `PageHandler`. При попытке выполнить этот код, вы получите ошибку, поскольку объект `document` не имеет
метода `doSomething`.

Вы можете завязать значение **this** на объекте `PageHandler` используя `handleEvent` или вызвав у функции стандартный
метод `bind()`:
```javascript
var PageHandler = {
    id: "123456"
    , init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // error
        }).bind(this));
    }
    , doSomething: function(type) { console.log("Handling " + type  + " for " + this.id) }
};
```
Теперь код будет работать так, как и задумывалось, но выглядит более громоздко. Кроме того, вызывая `bind(this)`
вы каждый раз создаёте новую функцию, значение **this** которой завязано на значении `PageHandler`, но зато код работает
так, как вы задумывали.

Стрелочные функции решают проблему более элегантным способом, поскольку используют лексическое связывание значения
**this** (также **super** и **arguments**) и его значение определяется значением **this** в том месте, где стрелочная функция
была создана. Например:
```javascript
var PageHandler = {
    id: "123456"
    , init: function() {
        document.addEventListener("click", event => this.doSomething(event.type));
    }
    , doSomething: function(type) { console.log("Handling " + type  + " for " + this.id) }
};
```
В этом примере, обработчик это стрелочная функция в которой вызывается `this.doSomething()`. Значение **this** будет тем
же, что и в функции `init()`, и код в данном примере отработает правильно, аналогично тому, который использовал `bind()`.
В не зависимости от того, возвращает вызов `this.doSomething()` значение или нет, выражение внутри тела стрелочной
функции не нужно обрамлять в фигурные скобки.

Кроме того, пример выше ещё и эффективнее вызова `bind()`, потому что для браузера он аналогичен
```javascript
var PageHandler = {
    id: "123456"
    , init: function() {
		var self = this;
        document.addEventListener("click", function(event) {
			return self.doSomething(event.type)
		});
    }
    , doSomething: function(type) { console.log("Handling " + type  + " for " + this.id) }
};
```
Т.е. не происходит создание новой функции, как в случае с вызовом `bind()`.

Более того, вы можете вкладывать одну стрелочную функцию в другую, "прокидывая" значение **this** через них. Пример:
```javascript
var obj = {
	arr1: [1, 2, 3]
	, arr2: ['a', 'b', 'c']
	, concatenate: function(a, b){ return a + "|" + b }
	, intersection: function() {
		return this.arr1.reduce( (sum, v1) => 
			this.arr2.reduce( (sum, v2) => (sum.push( this.concatenate( v1, v2 ) ),sum), sum )
			, []
		)
	}
};
var arrSum = obj.intersection();
```
Короткий синтаксис стрелочной функции делает их идеальными кандидатами на передачу в качестве аргументов в вызов других
функций. Например, если вы хотите отсортировать массив, вы обычно пишите что-то типа такого:
```javascript
var result = values.sort(function(a, b) { return a - b });
```
Довольно многословно для простой операции. Сравните с короткой записью стрелочной функции:
```javascript
var result = values.sort((a, b) => a - b);
```
Использование таких методов как массивные `sort()`, `map()`, `reduce()` и т.д., может быть упрошено с использование
короткого синтаксиса стрелочной функции.

## Особенности стрелочных функций

Несмотря на то, что стрелочные функции отличаются от традиционных функций, у них есть общие черты:
* оператор `typeof` вернёт "function" для стрелочной функции
* стрелочная функция также экземпляр "класса" Function, поэтому `instanceof` сработает также как и с традиционной функцией
* Вы всё ещё можете использовать методы `call()`, `apply()`, и `bind()`, однако помните, что они не будут влиять на
значение **this**
* Вы можете использовать метод `toMethod()`, однако он не будет менять значение **super**

Существенным отличием от традиционных функций является то, что попытка вызвать стрелочную функцию с указанием оператора
**new** вызовет ошибку исполнения.

## Итог

Стрелочные функции это одна из интереснейших нововведений в ECMAScript 6, которая имея краткий синтаксис определения
упростит передачу функций в качестве значения параметра другой функции. А лексическое связывание закроет один из самых
больших источников боли и разочарования для разработчиков, а так же улучшит производительность за счёт оптимизации на
уровне движка.

Если вы хотите попробовать стрелочные функции, вы можете выполнить примеры выше в консоли Firefox, который на данный
момент (02.2014 FF28) почти полноценно поддерживает стрелочные функции (FF28 неправильно вычисляет значение **arguments**).