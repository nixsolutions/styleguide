# jQuery Style Guide
Если вы изучаете данный раздел, значит можно с уверенностью сказать, что раздел
[JavaScript Style Guide](javascript.md) вы уже усвоили.

## Модульность
Весь код должен быть инкапсулирован внутри модуля (все функции и переменные).
Один файл может содержать только один модуль.
```javascript
/**
 * @param jQuery $  - be sure `$` is `jQuery`
 * @param window w  - be sure `w` is `window`
 */
(function($, w){
  // your code here
  var foo = "bar";
  // ... 
})(jQuery, window);
```

## Именование переменных
Переменные содержащие jQuery объект должны начинаться со знака $
```javascript
var $header = $("#header");
var $articles = $("article");
```

Используйте правильно единственное (singular) и множественное (plural) числа
```javascript
var $sidebar = $("#sidebar");
var $articles = $(".article");
```

Правильно указывайте множественное число (plural) для слов исключений:
```javascript
var $child = $("#child");
var $children = $(".child");
```

## Именование функций
Имена функций-оберток над селекторами должны начинаться со знака $
```javascript
function $articles() {
  return $("body").find(".content > article");
}
```

Имена функций должны быть короткими и содержательными, не нужно без нужды баламутить воду:
```javascript
// bad
function makeGreen() {} // color? background? WAT?
function loadData() {}  // data? rly?
function error() {}     // error in your DNA
```
К перечисленным функциям возникает слишком много вопросов, сравните со следующим примером:
```javascript
// good
function changeColorToGreen() {}
function loadJSON() {}
function showErrorDialog() {}
```
## Именование CSS классов
При создании классов для работы с JavaScript и jQuery используйте префикс `js-*`

## Callback функции
Для упрощения чтения кода следует избегать использования анонимных функций, 
т.к. это сводит на нет повтороное использование такого кода, и затрудняет чтение:
```javascript
// bad
// on click handler + callback function
$("body").on("click.module", "p", function() {
  $(this).css("color", "green");
});
```
Конечно данный пример ещё можно отнести к читабельным вариантам, 
но вот повторное использование функции уже стало невозможным.
Сравните:
```javascript
// good
// on click handler
$("body").on("click.module", "p", changeColorToGreen);
// callback function
function changeColorToGreen() {
  $(this).css("color", "green");
}
```
Наглядней преимущество именования можно оценить в более комплексном примере:
```javascript
// on click handler
$("header").on("click.module", loadJSON);

// callback with AJAX call
function loadJSON(event) {
  $.ajax("ajax/example.json", {context: event.currentTarget})
    .done(changeColorToGreen)
    .fail(changeColorToRed)
  ;
}

// callback for success
function changeColorToGreen() {
  $(this).css("color", "green");
}

// callback for error
function changeColorToRed() {
  $(this).css("color", "red");
}
```
## Пространство имён
Все обработчики событий внутри модуля следует «вешать» внутри единого пространства имён,
таким образом можно избежать конфликтов при одновременной работе сразу нескольких модулей

```javascript
// file `module.js`
(function($, w){
  var $headers = $("header");
  $headers.on("click.module", function() {
    // ...
  });
  $headers.trigger("click.module");
})(jQuery, window);
```
## Кэширование
Если вы пропустили этот момент в учебнике [jQuery для начинающих](http://anton.shevchuk.name/jquery-book/),
то повторю ещё разок ПОИСК ЭЛЕМЕНТОВ НЕ КЭШИРУЕТСЯ, следовательно вам надо самостоятельно позаботиться
о кэшировании ваших выборок:
```javascript
// bad
$("#sidebar");
// ...
$("#sidebar").css("color", "#ff0");
// ...
$("#sidebar").hide();
```
Подобный код следует переписать:
```javascript
// good
var $sidebar = $("#sidebar");
// ...
$sidebar.css("color", "#ff0");
// ...
$sidebar.hide();
```
Как вариант можете использовать следующую функцию, возможно она вам пригодится:
```javascript
var $function = function(selector) {
  var elements;
  return function() {
    if (!elements) {
      elements = $(selector);
    }
    return elements;
  }
};
```
С её помощью получается красивый КЭШИРУЕМЫЙ вариант для выборок элементов, которые не изменяются
```javascript
var $headers = $function("h2");
var $articles = $function("article");
$articles().on("click.module", e => false);
```
## Оптимизация
Стоит так же напомнить простое правило по оптимизации поиска элементов - указывайте 
[контекст поиска](https://jsperf.com/jquery-find-vs-context-sel/16):
```javascript
// slowest
$("article").find("p")
// slow too
$("p", "article")
```
Лучше так:
```javascript
// fastest
$(DOMElement).find("p")
// fast
$("p", DOMElement)
// fast too
$("p", $article[0])
```
Если данное правило вам не подходит, то посмотрите в сторону использования каскадных селекторов 
или селекторов вида `parent > child`:
```javascript
// not fast, not bad
$("article > p")
// slower, not bad
$("article p")
// slower
$article.find("p")
```
> В данных примерах используется `var`, если вам нет нужды поддерживать старые браузеры, то смело используйте
 `let` и `const` в соответствии со стандартами от Airbnb

> Так же следует быть осторожным, когда используете функции-стрелки, недопускайте ситуации, когда читаемость кода становится жертвой погони за синтаксическим сахаром, соблюдайте баланс
