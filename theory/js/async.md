# JS: Асинхронность

JavaScript — однопоточный язык. Это значит, что он может делать только одно дело за раз. Но браузер позволяет запускать сетевые запросы, таймеры и обработчики событий — и при этом не замораживаться. Всё это возможно благодаря асинхронности и Event Loop. Разберём от простого к сложному: колбэки, промисы, async/await и то, как всё это работает изнутри.

---

## Уровень 1: Callbacks

Колбэк — это просто функция, которую вы передаёте другой функции, чтобы та вызвала её когда-нибудь потом. Простейший пример — `setTimeout`:

```js
setTimeout(function() {
  console.log('Прошла секунда!');
}, 1000);
```

Колбэки отлично работают для одной-двух операций. Проблемы начинаются, когда нужно выполнить несколько асинхронных операций последовательно — каждая зависит от результата предыдущей.

**Callback Hell — пирамида обречённости:**

```js
loadScript('jquery.js', function(error, script) {
  if (error) {
    handleError(error);
  } else {
    loadScript('jquery-ui.js', function(error, script) {
      if (error) {
        handleError(error);
      } else {
        loadScript('app.js', function(error, script) {
          if (error) {
            handleError(error);
          } else {
            // наконец-то можно работать...
            initApp();
          }
        });
      }
    });
  }
});
```

Код "едет вправо" с каждым новым уровнем вложенности. Его сложно читать, сложно отлаживать, сложно рефакторить. При этом обработка ошибок размазана по всему дереву. Именно эта боль и стала причиной появления промисов.

---

## Уровень 1: Promise

Promise — это объект, представляющий результат асинхронной операции. Он может быть в трёх состояниях:
- `pending` — операция выполняется
- `fulfilled` — успешно завершена
- `rejected` — завершена с ошибкой

Состояние меняется ровно один раз и необратимо.

**Создание промиса:**

```js
const promise = new Promise(function(resolve, reject) {
  // здесь происходит асинхронная работа
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('данные загружены'); // переводим в fulfilled
    } else {
      reject(new Error('что-то пошло не так')); // переводим в rejected
    }
  }, 1000);
});
```

**Потребление результата через `.then`, `.catch`, `.finally`:**

```js
promise
  .then(result => {
    console.log('Успех:', result); // 'данные загружены'
    return result.toUpperCase(); // можно вернуть новое значение
  })
  .then(upper => {
    console.log('Модифицировано:', upper); // цепочка промисов
  })
  .catch(error => {
    console.error('Ошибка:', error.message); // перехватываем любую ошибку в цепочке
  })
  .finally(() => {
    console.log('Всегда выполняется'); // скрываем спиннер, освобождаем ресурсы
  });
```

Ключевая идея — цепочка `.then`. Каждый `.then` принимает результат предыдущего и возвращает новый промис. Так последовательные асинхронные операции записываются линейно, без вложенности:

```js
// Решение проблемы callback hell через промисы
loadScript('jquery.js')
  .then(() => loadScript('jquery-ui.js'))
  .then(() => loadScript('app.js'))
  .then(() => initApp())
  .catch(handleError); // единая обработка ошибок для всей цепочки
```

---

## Уровень 1: Async/Await

`async/await` — синтаксический сахар над промисами. Он позволяет писать асинхронный код так, как будто он синхронный. Никакой магии: под капотом это те же промисы.

**Ключевые правила:**
- `async` перед функцией — она всегда возвращает промис
- `await` можно использовать только внутри `async`-функции
- `await` приостанавливает выполнение функции до разрешения промиса

```js
async function loadUserData(userId) {
  const response = await fetch(`/api/users/${userId}`);
  const user = await response.json();
  return user; // обёрнется в Promise.resolve(user) автоматически
}

// Вызов async-функции возвращает промис
loadUserData(42).then(user => console.log(user));
```

**Обработка ошибок через `try/catch`:**

```js
async function loadUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Не удалось загрузить пользователя:', error.message);
    throw error; // пробрасываем ошибку выше, если нужно
  }
}
```

Если не использовать `try/catch`, ошибку можно поймать через `.catch()` на вызове функции:

```js
loadUserData(42)
  .then(user => renderUser(user))
  .catch(error => showErrorMessage(error));
```

**Параллельное выполнение с await:**

Важный нюанс — `await` блокирует. Если запускать операции последовательно, они не будут параллельными:

```js
// МЕДЛЕННО — операции выполняются одна за другой (3 сек)
async function slow() {
  const user = await fetchUser(); // 1 сек
  const posts = await fetchPosts(); // 2 сек — ждёт завершения user
}

// БЫСТРО — параллельно (2 сек, по времени самой долгой)
async function fast() {
  const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
}
```

---

## Уровень 1: setTimeout и setInterval

```js
// setTimeout — выполнить один раз через delay миллисекунд
const timerId = setTimeout(() => {
  console.log('раз');
}, 2000);

// Отменить до срабатывания
clearTimeout(timerId);

// setInterval — повторять каждые delay миллисекунд
const intervalId = setInterval(() => {
  console.log('тик');
}, 1000);

// Обязательно чистим, иначе утечка памяти
clearInterval(intervalId);
```

Важная деталь: `setTimeout(fn, 0)` не гарантирует мгновенное выполнение. Колбэк попадает в очередь задач и выполнится только после завершения текущего синхронного кода — и всех микрозадач.

---

## Уровень 2: Event Loop в деталях

Теперь разберём, как всё это работает на самом деле. Event Loop — это бесконечный цикл, в котором движок JS ожидает задачи, выполняет их и снова ждёт.

**Call Stack (стек вызовов)** — место, где выполняется код. Синхронные функции помещаются в стек и убираются оттуда по завершению. Стек — это LIFO (последний пришёл — первый ушёл).

**Task Queue (очередь макрозадач)** — сюда попадают колбэки `setTimeout`, `setInterval`, события пользователя, I/O-операции.

**Microtask Queue (очередь микрозадач)** — сюда попадают обработчики `.then/.catch/.finally` промисов и `queueMicrotask()`.

**Ключевое правило:** после завершения каждой макрозадачи движок полностью опустошает очередь микрозадач, прежде чем взять следующую макрозадачу или отрисовать страницу.

```js
console.log('1 — синхронный код');

setTimeout(() => console.log('2 — макрозадача'), 0);

Promise.resolve().then(() => console.log('3 — микрозадача'));
Promise.resolve().then(() => console.log('4 — микрозадача'));

console.log('5 — синхронный код');

// Вывод: 1, 5, 3, 4, 2
```

**Разбор:**
1. Выполняется весь синхронный код: `1`, `5`
2. Стек пуст — обрабатываем все микрозадачи: `3`, `4`
3. Берём следующую макрозадачу из очереди: `2`

Более сложный пример:

```js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve()
  .then(() => console.log(3))
  .then(() => setTimeout(() => console.log(4), 0)); // добавит макрозадачу

Promise.resolve().then(() => console.log(5));

setTimeout(() => console.log(6), 0);

console.log(7);

// Вывод: 1, 7, 3, 5, 6, 2, 4
// Нет, подождите: 1, 7, 3, 5, 2, 6, 4
```

Последовательность: `1`, `7` — синхронный код → `3`, `5` — микрозадачи (третий then добавляет макрозадачу `4` в конец очереди) → `2`, `6` — макрозадачи из первых setTimeout → `4` — макрозадача, добавленная из микрозадачи.

**Алгоритм Event Loop полностью:**

```
1. Взять старейшую задачу из Task Queue (макрозадача)
2. Выполнить её в Call Stack
3. Опустошить Microtask Queue (все новые микрозадачи тоже выполнить)
4. Если нужно — отрисовать страницу (requestAnimationFrame здесь)
5. Вернуться к шагу 1
```

**Практическое применение — разбиение тяжёлых задач:**

Если нужно обработать большой массив и не заморозить UI — разбиваем работу через `setTimeout(fn, 0)`:

```js
function processHeavyData(data) {
  let index = 0;

  function processChunk() {
    const end = Math.min(index + 1000, data.length);

    for (; index < end; index++) {
      // обрабатываем data[index]
    }

    if (index < data.length) {
      setTimeout(processChunk, 0); // отдаём управление браузеру
    }
  }

  processChunk();
}
```

Каждый вызов `setTimeout` создаёт новую макрозадачу — между ними браузер успевает отрисовать экран и обработать ввод пользователя.

---

## Уровень 2: requestAnimationFrame и рендеринг

`requestAnimationFrame` (rAF) — специальный механизм для анимаций. Браузер вызывает переданный колбэк перед следующим шагом отрисовки, синхронизируя его с частотой обновления экрана (обычно 60 раз в секунду, то есть каждые ~16 мс).

```js
function animate() {
  // изменяем DOM/стили
  element.style.left = (parseFloat(element.style.left) + 1) + 'px';

  // запрашиваем следующий кадр
  requestAnimationFrame(animate);
}

requestAnimationFrame(animate); // запускаем
```

В отличие от `setTimeout(fn, 16)`, rAF:
- Не вызывается, если вкладка неактивна — экономия ресурсов
- Синхронизирован с реальным циклом отрисовки браузера
- Гарантирует, что изменения применятся до следующей отрисовки

В Event Loop rAF-колбэки выполняются после микрозадач, но до следующего рендеринга страницы.

---

## Уровень 2: Promise API

Когда нужно работать с несколькими промисами одновременно, помогает `Promise API`:

**`Promise.all` — все или ничего:**

```js
// Ждёт ВСЕ промисы. Если хоть один отклонён — всё упало
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id)
]);
// Если fetchPosts упадёт — user и comments тоже потеряются
```

**`Promise.allSettled` — всегда получим все результаты:**

```js
// Ждёт ВСЕ промисы, не останавливается при ошибке
const results = await Promise.allSettled([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id)
]);

results.forEach(result => {
  if (result.status === 'fulfilled') {
    console.log('Данные:', result.value);
  } else {
    console.error('Ошибка:', result.reason);
  }
});
// Используйте, когда нужно обработать все результаты независимо
```

**`Promise.race` — побеждает первый:**

```js
// Возвращает результат первого завершённого (успех или ошибка)
const result = await Promise.race([
  fetch('/api/data'),
  new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Timeout')), 5000)
  )
]);
// Удобно для реализации таймаута запроса
```

**`Promise.any` — первый успешный:**

```js
// Возвращает первый УСПЕШНЫЙ результат
// Ждёт, пока не найдёт успех. Если все упали — AggregateError
const fastMirror = await Promise.any([
  fetch('https://mirror1.example.com/data'),
  fetch('https://mirror2.example.com/data'),
  fetch('https://mirror3.example.com/data')
]);
// Удобно для параллельных запросов к нескольким серверам
```

**Сравнение методов:**

| Метод | Когда выполняется | При ошибке |
|-------|-------------------|------------|
| `all` | Все успешны | Сразу отклоняется |
| `allSettled` | Все завершены | Возвращает все результаты |
| `race` | Первый завершён (любой) | Берёт статус первого |
| `any` | Первый успешен | Ждёт успешного или `AggregateError` |

---

## Источники

- [Event Loop — learn.javascript.ru](https://learn.javascript.ru/event-loop)
- [Promise: основы — learn.javascript.ru](https://learn.javascript.ru/promise-basics)
- [Promise API — learn.javascript.ru](https://learn.javascript.ru/promise-api)
- [Async/Await — learn.javascript.ru](https://learn.javascript.ru/async-await)
- [Колбэки — learn.javascript.ru](https://learn.javascript.ru/callbacks)
