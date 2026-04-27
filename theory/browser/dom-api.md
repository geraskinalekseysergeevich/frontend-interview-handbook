# DOM API

DOM (Document Object Model) — это программный интерфейс для HTML-документов. Браузер, получив HTML, строит из него дерево объектов, и именно с этим деревом работает JavaScript. Понимание DOM API — базовый навык фронтендера, без которого невозможна ни работа с фреймворками, ни отладка нативного кода.

---

## DOM: что такое, дерево узлов

DOM представляет документ в виде иерархии **узлов (nodes)**. Каждый тег — узел, каждый текст внутри тега — тоже узел. Корень дерева — объект `document`.

Существует несколько типов узлов:

| Тип | Константа | Пример |
|-----|-----------|--------|
| Element | `Node.ELEMENT_NODE` (1) | `<p>`, `<div>` |
| Text | `Node.TEXT_NODE` (3) | текст внутри тега |
| Comment | `Node.COMMENT_NODE` (8) | `<!-- комментарий -->` |
| Document | `Node.DOCUMENT_NODE` (9) | сам объект `document` |

```html
<body>
  <p>Текст <!-- комментарий --></p>
</body>
```

В этом примере `<body>` содержит узел `<p>`, который в свою очередь содержит текстовый узел "Текст" и узел-комментарий. Пробелы и переносы строк — тоже текстовые узлы, это частая причина путаницы при навигации по дереву.

---

## Поиск элементов

Браузер предоставляет несколько методов для поиска узлов. Современный стандарт — `querySelector` и `querySelectorAll`.

```js
// Возвращает первый подходящий элемент или null
const title = document.querySelector('h1');
const btn = document.querySelector('.btn-primary');
const input = document.querySelector('input[type="email"]');

// Возвращает статичный NodeList всех подходящих
const items = document.querySelectorAll('.list-item');
items.forEach(item => console.log(item.textContent));

// Классические методы — быстрее querySelector, но менее гибкие
const header = document.getElementById('header');           // один элемент
const links = document.getElementsByClassName('nav-link'); // живая HTMLCollection
const divs = document.getElementsByTagName('div');          // живая HTMLCollection
```

Важная разница: `querySelectorAll` возвращает **статичный** NodeList (снимок на момент вызова), а `getElementsByClassName` / `getElementsByTagName` возвращают **живую** HTMLCollection, которая автоматически обновляется при изменении DOM.

```js
const live = document.getElementsByClassName('item');
const static_ = document.querySelectorAll('.item');

// Добавляем новый элемент
document.body.innerHTML += '<div class="item">Новый</div>';

console.log(live.length);    // увеличилось — живая коллекция
console.log(static_.length); // не изменилось — статичный снимок
```

---

## Навигация по дереву

Имея ссылку на узел, можно перемещаться по дереву с помощью свойств:

```js
const el = document.querySelector('.parent');

// Дочерние элементы
el.children;         // HTMLCollection только элементов (без текстовых узлов)
el.childNodes;       // NodeList всех дочерних узлов, включая текстовые
el.firstElementChild;
el.lastElementChild;

// Родитель и соседи
el.parentElement;
el.parentNode;       // может быть и Document, а не только Element
el.nextElementSibling;
el.previousElementSibling;

// Утилиты
el.closest('.container'); // ближайший предок, подходящий под селектор (включая сам элемент)
el.matches('.active');    // true/false — подходит ли элемент под селектор
```

Метод `closest()` особенно полезен при делегировании событий — он позволяет найти ближайший "интересный" предок вне зависимости от глубины вложенности клика.

---

## Создание и вставка элементов. DocumentFragment

Для создания нового элемента используют `document.createElement()`. До вставки в документ элемент существует только в памяти и не отображается на странице.

```js
// Создание
const card = document.createElement('div');
card.className = 'card';
card.textContent = 'Привет';

// Вставка
document.body.append(card);         // в конец body
document.body.prepend(card);        // в начало body
someEl.before(card);                // перед someEl
someEl.after(card);                 // после someEl
someEl.replaceWith(card);           // заменить someEl

// insertAdjacentHTML — вставка HTML-строки
el.insertAdjacentHTML('beforeend', '<span class="badge">Новый</span>');
// Позиции: beforebegin | afterbegin | beforeend | afterend

// Удаление
card.remove();

// Клонирование
const clone = card.cloneNode(true);  // true — глубокое клонирование с потомками
```

**DocumentFragment** — это лёгкий контейнер-буфер без родителя. Его смысл: собрать несколько элементов в памяти, а затем вставить в DOM одной операцией. При вставке фрагмент "растворяется" — в DOM попадает только его содержимое.

```js
// Без фрагмента: 100 операций с DOM — медленно
const list = document.getElementById('list');
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Пункт ${i}`;
  list.append(li); // каждый раз триггерит возможный reflow
}

// С фрагментом: одна операция вставки
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Пункт ${i}`;
  fragment.append(li);
}
list.append(fragment); // один reflow
```

---

## Атрибуты vs свойства: разница и расхождения

Это один из самых частых вопросов на собеседовании. Атрибуты и свойства — разные вещи.

**Атрибуты** — это то, что написано в HTML-разметке. Они всегда строки.

**Свойства** — это поля JavaScript-объекта, соответствующего элементу. Они могут быть любого типа.

При загрузке страницы браузер создаёт DOM-узлы и **инициализирует свойства на основе атрибутов**. После этого они живут независимо.

```html
<input type="text" value="начальное значение" checked>
```

```js
const input = document.querySelector('input');

// Атрибут — начальное значение из HTML, всегда строка
input.getAttribute('value'); // "начальное значение"

// Свойство — текущее значение, изменяется при вводе пользователя
input.value; // "начальное значение" → но после ввода: "что ввёл пользователь"

// После ввода пользователя атрибут НЕ меняется:
input.getAttribute('value'); // по-прежнему "начальное значение"
input.value;                 // "что ввёл пользователь"
```

Другое классическое расхождение — атрибут `href`:

```html
<a id="link" href="/about">О нас</a>
```

```js
const a = document.querySelector('#link');

a.getAttribute('href'); // "/about" — строго то, что в HTML
a.href;                 // "https://example.com/about" — абсолютный URL
```

Булевы атрибуты — ещё одна ловушка:

```js
const checkbox = document.querySelector('input[type="checkbox"]');

checkbox.getAttribute('checked'); // "checked" или "" (строка) или null
checkbox.checked;                 // true или false (булево)

// Установка атрибута не работает так, как ожидается для live-состояния:
checkbox.setAttribute('checked', '');  // задаёт дефолт, но не текущее состояние
checkbox.checked = true;              // меняет текущее состояние
```

Управление атрибутами через API:

```js
el.getAttribute('data-id');       // читать
el.setAttribute('data-id', '42'); // установить
el.removeAttribute('data-id');    // удалить
el.hasAttribute('data-id');       // проверить наличие

// Специальный API для data-атрибутов
el.dataset.id;       // читает data-id
el.dataset.userId;   // читает data-user-id (camelCase)
el.dataset.id = '5'; // устанавливает data-id
```

---

## Управление CSS через JS

### Инлайн-стили через style

```js
const el = document.querySelector('.box');

el.style.backgroundColor = '#f00';  // camelCase вместо kebab-case
el.style.fontSize = '16px';         // значение всегда строка с единицей
el.style.display = 'none';

// Читать можно только явно установленные инлайн-стили
el.style.color; // "" — если цвет задан через класс, а не инлайн
```

### classList

Предпочтительный способ управления стилями — через классы:

```js
el.classList.add('active');
el.classList.remove('hidden');
el.classList.toggle('open');          // добавляет если нет, удаляет если есть
el.classList.toggle('open', true);    // принудительно добавить
el.classList.toggle('open', false);   // принудительно удалить
el.classList.contains('active');      // true/false
el.classList.replace('old', 'new');   // заменить один класс другим
```

### getComputedStyle

Для чтения реально применённых стилей (с учётом каскада, наследования и медиа-запросов):

```js
const styles = getComputedStyle(el);

styles.color;           // "rgb(255, 0, 0)"
styles.fontSize;        // "16px"
styles.display;         // "block"

// Для псевдоэлементов:
const before = getComputedStyle(el, '::before');
before.content; // '"★"'
```

`getComputedStyle` возвращает только для чтения — записать через него нельзя.

---

## События: addEventListener, фазы capture/bubble

Для подписки на события используют `addEventListener`:

```js
const btn = document.querySelector('#myBtn');

btn.addEventListener('click', (event) => {
  console.log('Клик!', event.target);
});

// Удаление обработчика — функция должна быть именованной
function handleClick(e) {
  console.log('Клик');
}
btn.addEventListener('click', handleClick);
btn.removeEventListener('click', handleClick);
```

### Три фазы события

Когда пользователь кликает по элементу, событие проходит **три фазы**:

1. **Capture (перехват)** — событие движется вниз от `window` до целевого элемента.
2. **Target** — событие находится на целевом элементе.
3. **Bubble (всплытие)** — событие движется вверх от целевого элемента к `window`.

```js
// Третий аргумент: false (по умолчанию) — обработчик на фазе bubble
document.body.addEventListener('click', (e) => {
  console.log('body bubble:', e.target.tagName);
}, false);

// true — обработчик на фазе capture (срабатывает раньше)
document.body.addEventListener('click', (e) => {
  console.log('body capture:', e.target.tagName);
}, true);

// При клике на <button> внутри body:
// → "body capture: BUTTON"   (capture, сверху вниз)
// → "body bubble: BUTTON"    (bubble, снизу вверх)
```

`event.target` — элемент, на котором произошло событие (не меняется).
`event.currentTarget` — элемент, на котором сейчас выполняется обработчик.

```js
// Остановка всплытия
btn.addEventListener('click', (e) => {
  e.stopPropagation();    // событие не пойдёт дальше вверх
  e.preventDefault();     // отменить действие браузера по умолчанию (например, переход по ссылке)
});
```

---

## Делегирование событий

Вместо навешивания обработчика на каждый дочерний элемент — назначить один обработчик на родителя и определять цель через `event.target`.

```html
<ul id="menu">
  <li data-action="open">Открыть</li>
  <li data-action="save">Сохранить</li>
  <li data-action="delete">Удалить</li>
</ul>
```

```js
const menu = document.getElementById('menu');

menu.addEventListener('click', (e) => {
  // closest ищет ближайший li по иерархии вверх (включая сам элемент)
  const item = e.target.closest('li');
  if (!item) return; // клик мимо li

  const action = item.dataset.action;

  if (action === 'open') openFile();
  if (action === 'save') saveFile();
  if (action === 'delete') deleteFile();
});
```

Преимущества:
- **Меньше памяти** — один обработчик вместо N.
- **Работает с динамическими элементами** — не нужно перевешивать обработчики при добавлении новых `li`.
- **Меньше кода** в инициализации.

Ограничение: не все события всплывают. Например, `focus` и `blur` — нет. Для них есть аналоги `focusin` и `focusout`, которые всплывают.

---

## Кастомные события: CustomEvent

Можно создавать и диспатчить собственные события — это основа event-driven архитектуры в чистом JS.

```js
// Создание события с данными
const event = new CustomEvent('user:login', {
  bubbles: true,     // всплывает по умолчанию false
  cancelable: true,
  detail: {
    userId: 42,
    name: 'Алексей'
  }
});

// Диспатч с нужного элемента
document.querySelector('#app').dispatchEvent(event);

// Подписка в любом месте выше по дереву
document.addEventListener('user:login', (e) => {
  console.log('Пользователь вошёл:', e.detail.name);
  console.log('ID:', e.detail.userId);
});
```

Соглашение: называть кастомные события в формате `namespace:action` — это помогает отличать их от нативных.

```js
// Паттерн: компонент сообщает о своём состоянии
class Accordion {
  constructor(el) {
    this.el = el;
    this.el.addEventListener('click', this.toggle.bind(this));
  }

  toggle() {
    this.el.classList.toggle('open');
    this.el.dispatchEvent(new CustomEvent('accordion:toggled', {
      bubbles: true,
      detail: { isOpen: this.el.classList.contains('open') }
    }));
  }
}
```

---

## HTML5 валидация форм

Браузер предоставляет встроенную валидацию форм через атрибуты HTML5. Без единой строки JS форма не отправится, если поля не заполнены правильно.

```html
<form id="registration">
  <input type="email" required placeholder="Email">
  <input type="text" minlength="3" maxlength="20" required placeholder="Имя">
  <input type="number" min="18" max="120" placeholder="Возраст">
  <input type="url" placeholder="Сайт">
  <input type="text" pattern="[А-Яа-я ]+" title="Только кириллица" placeholder="Фамилия">
  <button type="submit">Зарегистрироваться</button>
</form>
```

Управление через JS:

```js
const form = document.getElementById('registration');
const emailInput = document.querySelector('input[type="email"]');

// Проверить валидность всей формы
form.checkValidity(); // true/false

// Проверить конкретное поле
emailInput.validity.valueMissing;  // true если required и пустое
emailInput.validity.typeMismatch;  // true если значение не email
emailInput.validity.tooShort;      // true если меньше minlength
emailInput.validity.patternMismatch; // true если не совпадает с pattern
emailInput.validity.valid;         // true если всё ок

// Кастомные сообщения об ошибке
emailInput.setCustomValidity('Этот email уже занят');
emailInput.setCustomValidity(''); // сбросить — поле снова валидно

// Отключить встроенные поп-апы браузера и делать своё
form.addEventListener('submit', (e) => {
  e.preventDefault();

  if (!form.checkValidity()) {
    // Показываем свои ошибки
    const firstInvalid = form.querySelector(':invalid');
    firstInvalid.focus();
    return;
  }

  // Отправка данных
  const data = new FormData(form);
  fetch('/api/register', { method: 'POST', body: data });
});

// Атрибут novalidate отключает встроенную валидацию полностью
// <form novalidate> — управляем всем сами
```

CSS-псевдоклассы для стилизации состояний валидации:

```css
input:valid   { border-color: green; }
input:invalid { border-color: red; }
input:required { background: #fffbe6; }

/* Только после попытки отправки или фокуса — используем :user-invalid */
input:user-invalid { border-color: red; }
```

---

## Источники

- [DOM-узлы — learn.javascript.ru](https://learn.javascript.ru/dom-nodes)
- [Навигация по DOM — learn.javascript.ru](https://learn.javascript.ru/dom-navigation)
- [Изменение документа — learn.javascript.ru](https://learn.javascript.ru/modifying-document)
- [Введение в браузерные события — learn.javascript.ru](https://learn.javascript.ru/introduction-browser-events)
- [Всплытие и погружение — learn.javascript.ru](https://learn.javascript.ru/event-bubbling)
- [Делегирование событий — learn.javascript.ru](https://learn.javascript.ru/event-delegation)
