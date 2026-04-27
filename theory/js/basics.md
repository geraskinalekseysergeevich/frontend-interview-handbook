# JS: Основы

JavaScript — язык с динамической типизацией, прототипным наследованием и функциями как объектами первого класса. Это означает, что здесь нельзя просто "выучить синтаксис" и идти дальше: нужно понять, как язык думает изнутри. В этом файле разбираем фундамент, без которого на Junior+ уровне делать нечего.

---

## Типы данных

В JavaScript есть 8 типов данных. Семь из них — примитивы, один — объект.

**Примитивы:** `string`, `number`, `bigint`, `boolean`, `null`, `undefined`, `symbol`.

**Ссылочный тип:** `object` (сюда входят массивы, функции, даты и всё остальное).

```js
// Примитивы хранятся по значению
let a = 'hello';
let b = a;
b = 'world';
console.log(a); // 'hello' — a не изменилась

// Объекты хранятся по ссылке
let obj1 = { name: 'Вася' };
let obj2 = obj1;
obj2.name = 'Петя';
console.log(obj1.name); // 'Петя' — оба указывают на одно и то же
```

Понимание этого различия критично. Когда вы передаёте объект в функцию, вы передаёте ссылку. Функция может мутировать оригинал — это частый источник багов.

Оператор `typeof` помогает определить тип во время выполнения, но с одним известным сюрпризом:

```js
typeof null; // 'object' — историческая ошибка языка, так и живёт
typeof undefined; // 'undefined'
typeof function(){}; // 'function' — хотя функции это объекты
```

---

## Условия, операторы, циклы

Ничего экзотического, но несколько нюансов стоит держать в голове.

**Приведение типов и `==` vs `===`:**

```js
0 == false;  // true — JS приводит оба к числу
0 === false; // false — нет приведения, разные типы
null == undefined; // true — особое правило языка
null === undefined; // false
```

Всегда используйте `===`, если только вам явно не нужна нестрогая проверка.

**Тернарный оператор и nullish coalescing:**

```js
// Тернарный — для условного присваивания
const label = isAdmin ? 'Администратор' : 'Пользователь';

// ?? — только для null/undefined (не для 0 или '')
const port = userPort ?? 3000;

// || — для любого falsy (0, '', false тоже заменит)
const name = userName || 'Гость'; // осторожно: пустая строка тоже заменится
```

**Циклы — когда что использовать:**

```js
// for — когда нужен индекс или точное количество итераций
for (let i = 0; i < arr.length; i++) { ... }

// for...of — перебор значений итерируемого (массив, строка, Set, Map)
for (const item of arr) { ... }

// for...in — перебор ключей объекта (не для массивов!)
for (const key in obj) {
  if (obj.hasOwnProperty(key)) { ... } // защита от прототипных ключей
}

// while — когда количество итераций заранее неизвестно
while (condition) { ... }
```

---

## Замыкания и лексическое окружение

Замыкание — одна из самых важных концепций в JavaScript. Если понять её по-настоящему, многие "магические" вещи в языке станут очевидными.

**Лексическое окружение** — это невидимый объект, который создаётся при каждом запуске функции или блока кода `{}`. Он хранит две вещи:
1. Локальные переменные этого блока/функции
2. Ссылку на внешнее окружение (где эта функция была объявлена)

Когда вы обращаетесь к переменной, движок ищет её сначала в текущем окружении, потом идёт выше по цепочке — и так до глобального объекта.

**Замыкание** — это функция, которая запомнила своё лексическое окружение. Все функции в JS являются замыканиями, потому что у каждой функции есть скрытое свойство `[[Environment]]`, указывающее на окружение, в котором она была создана.

```js
function makeCounter() {
  let count = 0; // эта переменная живёт в окружении makeCounter

  return function() {
    return count++; // внутренняя функция замыкает count из внешнего окружения
  };
}

const counter = makeCounter();
console.log(counter()); // 0
console.log(counter()); // 1
console.log(counter()); // 2

// Каждый вызов makeCounter создаёт НОВОЕ окружение
const counter2 = makeCounter();
console.log(counter2()); // 0 — независимый счётчик
```

Переменная `count` не исчезает после выхода из `makeCounter`, потому что на неё держит ссылку возвращённая функция. Сборщик мусора не тронет объект, пока на него есть живые ссылки.

**Практический пример — модульный паттерн:**

```js
const userService = (function() {
  let _users = []; // приватная переменная — недоступна снаружи

  return {
    addUser(user) {
      _users.push(user);
    },
    getCount() {
      return _users.length;
    }
  };
})();

userService.addUser({ name: 'Вася' });
console.log(userService.getCount()); // 1
console.log(userService._users); // undefined — снаружи не видно
```

---

## Передача контекста: this, call, apply, bind

`this` — одна из главных источников боли в JS. Значение `this` определяется не тем, где функция объявлена, а тем, как она вызвана.

```js
const user = {
  name: 'Вася',
  greet() {
    console.log(`Привет, я ${this.name}`);
  }
};

user.greet(); // 'Привет, я Вася' — this = user

const fn = user.greet;
fn(); // 'Привет, я undefined' — this потерялся, в strict mode это undefined
```

Когда вы вырываете метод из объекта и передаёте его как колбэк, `this` теряется. Это классическая проблема.

**Три способа явно задать `this`:**

```js
// call — вызывает функцию с заданным this и аргументами через запятую
user.greet.call({ name: 'Петя' }); // 'Привет, я Петя'

// apply — то же самое, но аргументы передаются массивом
function sum(a, b) { return a + b; }
sum.apply(null, [1, 2]); // 3

// bind — создаёт новую функцию с жёстко зафиксированным this
const greetVasya = user.greet.bind(user);
setTimeout(greetVasya, 1000); // 'Привет, я Вася' — this не потеряется
```

`bind` особенно полезен при передаче методов как колбэков:

```js
class Button {
  constructor(label) {
    this.label = label;
    this.handleClick = this.handleClick.bind(this); // фиксируем this в конструкторе
  }

  handleClick() {
    console.log(`Нажата кнопка: ${this.label}`);
  }
}

const btn = new Button('Отправить');
document.addEventListener('click', btn.handleClick); // this корректный
```

**Стрелочные функции** — отдельная история. Они не имеют собственного `this` и захватывают его из внешнего лексического окружения:

```js
const user = {
  name: 'Вася',
  greet() {
    // Обычная функция — this потеряется в колбэке
    setTimeout(function() {
      console.log(this.name); // undefined
    }, 100);

    // Стрелочная — захватывает this из greet()
    setTimeout(() => {
      console.log(this.name); // 'Вася' — работает!
    }, 100);
  }
};
```

---

## Классы под капотом: прототипное наследование

Классы в JS — это синтаксический сахар над прототипным наследованием. Под капотом никаких классов нет, есть только объекты и ссылки между ними.

**Как работает цепочка прототипов:**

```js
// Когда вы делаете это:
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} издаёт звук`;
  }
}

class Dog extends Animal {
  speak() {
    return `${this.name} лает`;
  }
}

const dog = new Dog('Бобик');
dog.speak(); // 'Бобик лает'
```

Под капотом это выглядит примерно так:

```js
// Эквивалент через прототипы
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} издаёт звук`;
};

function Dog(name) {
  Animal.call(this, name); // вызываем родительский конструктор
}
Dog.prototype = Object.create(Animal.prototype); // наследуем прототип
Dog.prototype.constructor = Dog; // восстанавливаем конструктор
Dog.prototype.speak = function() {
  return `${this.name} лает`;
};
```

Когда вы обращаетесь к `dog.speak()`, движок ищет метод сначала у самого объекта `dog`, потом в `Dog.prototype`, потом в `Animal.prototype`, потом в `Object.prototype`, и дальше `null` — конец цепочки.

**Современные методы работы с прототипами:**

```js
// Object.create — создать объект с нужным прототипом
const proto = {
  greet() {
    return `Привет, я ${this.name}`;
  }
};

const obj = Object.create(proto);
obj.name = 'Вася';
obj.greet(); // 'Привет, я Вася'

// Проверить прототип
Object.getPrototypeOf(obj) === proto; // true

// Объект без прототипа — чистый словарь
const dict = Object.create(null);
dict['__proto__'] = 'что угодно'; // никаких конфликтов с прототипом
```

---

## API объектов и массивов

### Методы массивов

Три кита функциональной работы с массивами: `map`, `filter`, `reduce`. Они не мутируют исходный массив и возвращают новый результат.

```js
const users = [
  { id: 1, name: 'Вася', age: 25, active: true },
  { id: 2, name: 'Петя', age: 17, active: false },
  { id: 3, name: 'Маша', age: 30, active: true },
];

// map — трансформировать каждый элемент
const names = users.map(u => u.name);
// ['Вася', 'Петя', 'Маша']

// filter — отобрать по условию
const adults = users.filter(u => u.age >= 18 && u.active);
// [{ id: 1, name: 'Вася', ... }, { id: 3, name: 'Маша', ... }]

// reduce — свернуть в одно значение
const totalAge = users.reduce((sum, u) => sum + u.age, 0);
// 72

// Цепочки — можно комбинировать
const activeNames = users
  .filter(u => u.active)
  .map(u => u.name.toUpperCase());
// ['ВАСЯ', 'МАША']
```

Чуть реже, но не менее важны:

```js
// find — первый подходящий элемент (или undefined)
const vася = users.find(u => u.name === 'Вася');

// some — хоть один элемент подходит?
const hasMinors = users.some(u => u.age < 18); // true

// every — все элементы подходят?
const allActive = users.every(u => u.active); // false

// flat и flatMap — для работы с вложенными массивами
const nested = [[1, 2], [3, 4]];
nested.flat(); // [1, 2, 3, 4]
```

### Методы объектов

```js
const config = { host: 'localhost', port: 3000, debug: true };

// Перебор ключей, значений, пар
Object.keys(config);    // ['host', 'port', 'debug']
Object.values(config);  // ['localhost', 3000, true]
Object.entries(config); // [['host', 'localhost'], ['port', 3000], ...]

// Удобно для трансформации объекта
const doubled = Object.fromEntries(
  Object.entries(config).map(([key, val]) => [key, val])
);

// Object.assign — поверхностное слияние
const defaults = { port: 3000, debug: false };
const merged = Object.assign({}, defaults, { port: 8080 });
// { port: 8080, debug: false } — defaults не мутируется

// Spread — то же самое, но читабельнее
const merged2 = { ...defaults, port: 8080 };
```

### Деструктуризация

Мощная возможность, упрощающая извлечение данных:

```js
// Деструктуризация объекта
const { name, age, city = 'Москва' } = user; // city со значением по умолчанию

// Переименование при деструктуризации
const { name: userName, age: userAge } = user;

// Деструктуризация массива
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

// Деструктуризация в параметрах функции
function renderUser({ name, age, role = 'user' }) {
  return `${name} (${age}), роль: ${role}`;
}

// Вложенная деструктуризация
const { address: { city, street } } = user;
```

Spread-оператор работает и с массивами, и с объектами:

```js
// Копирование массива (поверхностное)
const copy = [...originalArray];

// Объединение массивов
const combined = [...arr1, ...arr2, ...arr3];

// Spread в вызове функции
Math.max(...[1, 2, 3]); // 3
```

---

## Источники

- [Замыкания — learn.javascript.ru](https://learn.javascript.ru/closure)
- [Функции bind, call, apply — learn.javascript.ru](https://learn.javascript.ru/bind)
- [Методы прототипов — learn.javascript.ru](https://learn.javascript.ru/prototype-methods)
- [Методы массивов — learn.javascript.ru](https://learn.javascript.ru/array-methods)
- [Методы объектов и this — learn.javascript.ru](https://learn.javascript.ru/object-methods)
