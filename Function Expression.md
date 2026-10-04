# JS: Function Expression

## 📌 Что это

**Function Expression** — создание функции **внутри выражения** (например, справа от `=`). В отличие от Function Declaration, здесь функция — это **значение**, которое присваивается переменной.

```js
let sayHi = function() {
    console.log("Привет");
};
```

⚠️ **После `function` нет имени** — функция анонимная.

---

## 🔧 Два синтаксиса

**Function Declaration:**
```js
function sayHi() {
    console.log("Привет");
}
```
Отдельная инструкция. Имя обязательно.

**Function Expression:**
```js
let sayHi = function() {
    console.log("Привет");
};
```
Внутри выражения (`= ...`). Имя не обязательно.

**Оба создают функцию и кладут её в переменную `sayHi`.**

---

## 🎯 Функция — это значение

Функция — **такое же значение**, как число или строка. С ней можно работать как с любым другим значением.

```js
function sayHi() {
    console.log("Привет");
}

console.log(sayHi);   // выведет КОД функции (не вызов!)
```

⚠️ **Без `()` — не вызывается.** `sayHi` — это ссылка на функцию. `sayHi()` — вызов.

**Можно скопировать:**
```js
function sayHi() { console.log("Привет"); }

let func = sayHi;    // ← без скобок! копируем функцию
func();              // Привет
sayHi();             // Привет
```

⚠️ `let func = sayHi()` (со скобками) — **вызовет** функцию и сохранит **результат** (`undefined`). Не то.

---

## 📝 Точка с запятой

**Function Declaration** — `;` не нужна:
```js
function sayHi() {
    // ...
}
```

**Function Expression** — `;` нужна:
```js
let sayHi = function() {
    // ...
};
```

**Почему:** `Function Expression` — часть **выражения присваивания**. Как `let x = 5;` — точка с запятой завершает инструкцию.

---

## 🔄 Функции-колбэки

**Колбэк** — функция, которую передают **как аргумент** другой функции.

```js
function ask(question, yes, no) {
    if (confirm(question)) yes();
    else no();
}

function showOk()     { alert("Согласны"); }
function showCancel() { alert("Отменено"); }

ask("Вы согласны?", showOk, showCancel);
```

⚠️ **Без скобок** при передаче: `showOk`, а не `showOk()`. Иначе функция вызовется **сразу**.

**Анонимные функции прямо в вызове:**
```js
ask(
    "Вы согласны?",
    function() { alert("Согласны"); },
    function() { alert("Отменено"); }
);
```
Здесь функции **без имени** — не доступны снаружи, и это ок.

---

## ⚖️ Declaration vs Expression: ключевые отличия

### 1. Когда создаётся функция

**Function Declaration** — **до** выполнения кода (hoisting):
```js
sayHi("Вася");   // ✅ Работает

function sayHi(name) {
    console.log("Привет, " + name);
}
```

**Function Expression** — **когда доходит** до строки:
```js
sayHi("Вася");   // ❌ Ошибка: sayHi is not defined

let sayHi = function(name) {
    console.log("Привет, " + name);
};
```

---

### 2. Блочная область видимости

**Function Declaration внутри `if`** — видна **только внутри блока**:

```js
let age = prompt("Возраст?", 18);

if (age < 18) {
    function welcome() { alert("Привет!"); }
} else {
    function welcome() { alert("Здравствуйте!"); }
}

welcome();   // ❌ Ошибка: welcome is not defined
```

**Решение** — объявить переменную снаружи и присвоить через `Function Expression`:

```js
let age = prompt("Возраст?", 18);
let welcome;

if (age < 18) {
    welcome = function() { alert("Привет!"); };
} else {
    welcome = function() { alert("Здравствуйте!"); };
}

welcome();   // ✅ Работает
```

**Или через тернарник:**
```js
let welcome = (age < 18) ?
    function() { alert("Привет!"); } :
    function() { alert("Здравствуйте!"); };

welcome();
```

---

## 📊 Сводная таблица

| Критерий | Function Declaration | Function Expression |
|----------|---------------------|---------------------|
| Синтаксис | `function name() {}` | `let name = function() {}` |
| Имя | Обязательно | Не обязательно |
| Когда создаётся | **До** выполнения (hoisting) | **Когда доходит** поток |
| Вызов до объявления | ✅ Работает | ❌ Ошибка |
| Область видимости | Внутри блока `{}` | Снаружи (если переменная снаружи) |
| `;` в конце | Не нужна | **Нужна** |
| Использование | По умолчанию | Когда Declaration не подходит |

---

## 🎯 Когда что использовать

**По умолчанию — Function Declaration:**
- Видна до объявления.
- Легче «ловится глазами».
- Больше свободы в организации кода.

**Function Expression — когда Declaration не подходит:**
- **Условное объявление** (внутри `if`).
- **Анонимная функция** (одноразовая, в колбэке).
- **Функция как значение** (передача в аргумент).

---

## 🎯 Связь с пентестом

- **Payload'ы — это Function Expression.** Пример:
  ```js
  let payload = function() { alert(document.cookie); };
  ```
- **Колбэки в `fetch`:** `fetch(...).then(function(r) { ... })`.
- **IIFE (Immediately Invoked Function Expression)** — частый приём в обфускации:
  ```js
  (function() { alert(1); })();
  ```
  Функция создаётся **и сразу вызывается**. Используется для изоляции кода в payload'ах.
- **Передача функции как значения** — основа XSS через `postMessage`, `setTimeout(function(){}, 0)`.
- **Анонимные функции** — в обфусцированном коде трудно отследить, где какая функция.
- **Hoisting** — используется в эксплуатации: можно вызвать функцию **раньше** её объявления в чужом коде.

---

## ✅ Чек-лист

- [✅] Понимаю разницу Declaration и Expression
- [✅] Знаю, что функция — это значение
- [✅] Умею копировать функцию в переменную (без скобок)
- [✅] Понимаю, когда нужна `;`
- [✅] Знаю, что такое колбэк
- [✅] Помню про hoisting (Declaration — до, Expression — после)
- [✅] Знаю про блочную область видимости
- [✅] Понимаю, когда использовать Expression вместо Declaration

---

## 📚 Источник

- [learn.javascript.ru → Function Expression](https://learn.javascript.ru/function-expressions)

---

*Дата: 2026-10-04*
