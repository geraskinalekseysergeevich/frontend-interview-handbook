# TypeScript

TypeScript — это надстройка над JavaScript, которая добавляет статическую типизацию. Грубо говоря: вы пишете JS, но при этом говорите компилятору, что переменная может быть только строкой, а функция принимает объект с конкретными полями. Если что-то не так — ошибка ещё на этапе разработки, а не в продакшне в три ночи.

---

## Виды типизации: разбираемся в терминах

Прежде чем говорить о TypeScript, важно понять, в чём разница между разными подходами к типизации.

**Статическая vs Динамическая:**

При **статической** типизации типы переменных известны до запуска — на этапе компиляции. Компилятор не даст передать строку туда, где ожидается число. Примеры: TypeScript, Java, C#.

При **динамической** типизации тип переменной определяется во время выполнения. Одна переменная может быть числом, потом строкой, потом объектом. JavaScript — динамически типизированный язык.

```js
// JavaScript — динамическая типизация
let x = 42;
x = 'теперь строка'; // ок, без вопросов
x = { prop: true };   // ок, никаких претензий
```

```ts
// TypeScript — статическая типизация
let x: number = 42;
x = 'теперь строка'; // Ошибка компиляции!
```

**Сильная vs Слабая:**

**Сильная** типизация — язык не выполняет неявных преобразований типов. Нельзя сложить число и строку без явного приведения. Пример: Python.

**Слабая** типизация — язык автоматически приводит типы при необходимости. JavaScript — слабо типизирован:

```js
'5' + 3;    // '53' — число превратилось в строку
'5' - 3;    // 2   — строка превратилась в число
true + 1;   // 2   — true стал 1
[] + {};    // '[object Object]' — добро пожаловать в JS
```

TypeScript **не** делает язык сильно типизированным на уровне выполнения — он компилируется в обычный JS. Но он добавляет **явную и статическую** типизацию, которая ловит многие ошибки до запуска.

**Явная vs Неявная:**

Явная — вы сами пишете типы. Неявная (вывод типов, type inference) — компилятор определяет тип сам по контексту. TypeScript умеет и то, и другое:

```ts
let name: string = 'Вася'; // явная аннотация типа
let age = 25;               // неявная — TypeScript сам понял: number
```

---

## Зачем TypeScript в JS-проекте

Честный ответ: не всегда нужен. На маленьком скрипте TypeScript — это оверхед. Но на реальном проекте с командой из нескольких человек, тысячами строк кода и месяцами разработки — это другой разговор.

**Что даёт TypeScript:**

1. **Ошибки до запуска.** Передали `null` вместо объекта — узнаете при написании кода, а не когда пользователь увидит белый экран.

2. **Автодополнение и навигация.** IDE знает структуру объектов и подсказывает свойства и методы. Это не мелочь — это огромная часть комфорта разработки.

3. **Рефакторинг без страха.** Переименовали функцию или изменили сигнатуру — TypeScript укажет все места, которые нужно обновить.

4. **Документация через типы.** Тип функции — это уже документация. `function createUser(name: string, age: number): Promise<User>` понятнее любого комментария.

5. **Межкомандное взаимодействие.** Когда один разработчик делает API, а другой — UI, типы служат контрактом между ними.

---

## Базовые типы

```ts
// Примитивы — строчными буквами (не String, Number, Boolean!)
let name: string = 'Вася';
let age: number = 25;
let isAdmin: boolean = true;

// Массивы — два эквивалентных способа
let scores: number[] = [1, 2, 3];
let tags: Array<string> = ['ts', 'js'];

// Кортеж — массив с фиксированными типами на каждой позиции
let pair: [string, number] = ['hello', 42];
```

**`any` — отключает проверку типов:**

```ts
let x: any = 'строка';
x = 42;           // ок
x = { foo: 1 };  // ок
x.nonExistent();  // ок — TypeScript не проверяет
```

`any` — это эвакуационный люк. Полезен при постепенной миграции с JS на TS, но злоупотребление им лишает смысла всю типизацию.

**`unknown` — безопасная альтернатива `any`:**

```ts
let value: unknown = 'строка';
value.toUpperCase(); // Ошибка! Нельзя использовать без проверки типа

// Нужно сначала проверить тип
if (typeof value === 'string') {
  value.toUpperCase(); // теперь ок — TypeScript знает, что это строка
}
```

`unknown` говорит: "я не знаю тип, но ты обязан проверить его прежде чем использовать". Это гораздо безопаснее `any`.

**`never` — значение, которого никогда не будет:**

```ts
// Функция, которая никогда не возвращает (всегда бросает ошибку или бесконечный цикл)
function throwError(message: string): never {
  throw new Error(message);
}

// Полезно в exhaustive checks — проверке, что обработали все случаи
type Shape = 'circle' | 'square' | 'triangle';

function getArea(shape: Shape): number {
  switch (shape) {
    case 'circle': return Math.PI;
    case 'square': return 1;
    case 'triangle': return 0.5;
    default:
      const _exhaustive: never = shape; // Ошибка, если добавили новый Shape и забыли обработать
      throw new Error(`Неизвестная фигура: ${shape}`);
  }
}
```

**`void` — функция ничего не возвращает:**

```ts
function logMessage(msg: string): void {
  console.log(msg);
  // return; — можно вернуть undefined, нельзя вернуть что-то другое
}
```

---

## Interfaces vs Type Aliases

Это один из самых частых вопросов. Оба позволяют описать форму данных, но ведут себя по-разному.

**Interface:**

```ts
interface User {
  id: number;
  name: string;
  email?: string; // опциональное поле
  readonly createdAt: Date; // нельзя изменить после создания
}

// Interface можно расширить в любом месте
interface User {
  role: 'admin' | 'user'; // декларация merging — добавляем поле
}

// Наследование через extends
interface AdminUser extends User {
  permissions: string[];
}
```

**Type alias:**

```ts
type User = {
  id: number;
  name: string;
};

// Type нельзя "переоткрыть" — это ошибка
type User = { role: string }; // Ошибка: Duplicate identifier 'User'

// Но type может описать union, tuple, примитив
type ID = string | number;
type Pair = [string, number];
type Name = string; // просто псевдоним

// Расширение через intersection
type AdminUser = User & { permissions: string[] };
```

**Когда что использовать:**

- `interface` — для описания объектов и классов. Предпочтительнее, если нужно расширять или реализовывать через `implements`.
- `type` — для всего остального: union types, tuple, примитивов, сложных вычисленных типов.

На практике в большинстве команд используют `interface` для объектов и `type` для всего остального — и это разумное соглашение.

---

## Generics — параметризованные типы

Generics — это способ написать код, который работает с любым типом, сохраняя при этом типовую безопасность. Аналог шаблонов в C++ или дженериков в Java.

**Зачем нужны:**

```ts
// Без generics — пришлось бы писать для каждого типа или использовать any
function identity(value: any): any {
  return value;
}

// С generics — сохраняем информацию о типе
function identity<T>(value: T): T {
  return value;
}

identity<string>('hello'); // возвращает string
identity<number>(42);      // возвращает number
identity('hello');         // T выводится автоматически как string
```

**Практические примеры:**

```ts
// Generic интерфейс — типизированный ответ от API
interface ApiResponse<T> {
  data: T;
  status: number;
  error?: string;
}

interface User {
  id: number;
  name: string;
}

// Теперь TypeScript знает точную структуру data
async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

const result = await fetchUser(1);
result.data.name; // TypeScript знает, что это string
```

**Ограничения generics через `extends`:**

```ts
// T должен иметь свойство length
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength('hello');    // 5 — строка имеет length
getLength([1, 2, 3]); // 3 — массив имеет length
getLength(42);         // Ошибка! число не имеет length
```

---

## Модификаторы доступа в классах

TypeScript добавляет модификаторы доступа, которых нет в обычном JS (хотя в современном JS есть приватные поля через `#`).

```ts
class UserService {
  public name: string;        // доступен везде (по умолчанию)
  protected role: string;     // доступен в классе и наследниках
  private _password: string;  // доступен только внутри класса
  readonly id: number;        // можно задать только в конструкторе
  static instanceCount = 0;   // свойство класса, не экземпляра

  constructor(name: string, password: string) {
    this.name = name;
    this._password = password;
    this.id = Math.random();
    UserService.instanceCount++;
  }

  // Краткая запись — то же самое что присвоение в constructor
  // constructor(public name: string, private _password: string) {}

  public checkPassword(input: string): boolean {
    return this._password === input;
  }
}

const service = new UserService('Вася', 'secret');
service.name;           // ок
service._password;      // Ошибка TS! — приватное
service.id = 999;       // Ошибка TS! — readonly
UserService.instanceCount; // ок — статическое свойство
```

**Важно:** `private`, `public`, `protected` — это только TypeScript-проверки. После компиляции их нет в JS. Для настоящей приватности в рантайме используйте `#`.

---

## Опциональная цепочка `?.` и ненулевое утверждение `!`

**Опциональная цепочка `?.`:**

Позволяет безопасно обращаться к свойствам объекта, который может быть `null` или `undefined`:

```ts
interface User {
  name: string;
  address?: {
    city?: string;
  };
}

const user: User | null = getUser();

// Без ?. — нужно проверять каждый уровень
const city1 = user !== null && user.address !== undefined
  ? user.address.city
  : undefined;

// С ?. — коротко и безопасно
const city2 = user?.address?.city;

// Работает и с методами
const upper = user?.name?.toUpperCase();

// И с массивами
const firstItem = arr?.[0];
```

Если любое звено цепочки `null` или `undefined` — выражение вернёт `undefined` без ошибки.

**Ненулевое утверждение `!`:**

Говорит TypeScript: "я знаю, что это не null/undefined, доверяй мне":

```ts
function processUser(user: User | null) {
  // TypeScript требует проверку перед использованием
  user.name; // Ошибка! user может быть null

  // Явная проверка — правильный путь
  if (user !== null) {
    user.name; // ок, TypeScript сузил тип до User
  }

  // ! — если вы точно уверены
  user!.name; // убираем null из типа; ошибка рантайма если user действительно null
}
```

`!` нужно использовать осторожно. Это, по сути, обещание компилятору: "здесь точно не null". Если обманули — получите TypeError в рантайме. Хороший сигнал к рефакторингу, а не к использованию `!`.

---

## Union Types и Type Narrowing

**Union Types — когда значение может быть нескольких типов:**

```ts
type ID = string | number;

function printId(id: ID) {
  // Нельзя сразу вызвать toUpperCase() — вдруг это number
  console.log(id.toUpperCase()); // Ошибка!

  // Нужно сузить тип
  if (typeof id === 'string') {
    console.log(id.toUpperCase()); // ок — TypeScript знает: это string
  } else {
    console.log(id.toFixed(2)); // ок — TypeScript знает: это number
  }
}
```

**Type Narrowing — автоматическое сужение типа:**

```ts
type Cat = { type: 'cat'; meow(): void };
type Dog = { type: 'dog'; bark(): void };
type Animal = Cat | Dog;

function makeSound(animal: Animal) {
  if (animal.type === 'cat') {
    animal.meow(); // TypeScript знает: это Cat
  } else {
    animal.bark(); // TypeScript знает: это Dog
  }
}
```

---

## Источники

- [Everyday Types — TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [Object Types — TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [Введение в типизацию — Habr](https://habr.com/ru/post/161205/)
