# Отладка в браузере: Chrome DevTools

Отлаживать код с помощью `console.log` — как искать иголку в стоге сена с закрытыми глазами. Chrome DevTools — это полноценная IDE для исследования и исправления проблем прямо в браузере. Знание этих инструментов отличает разработчика, который тратит час на поиск бага, от того, кто находит его за пять минут.

---

## Уровень 1: Основные панели DevTools

Открыть DevTools: `F12` или `Cmd+Option+I` на Mac, `Ctrl+Shift+I` на Windows/Linux. Или правая кнопка мыши → «Inspect».

---

### Elements: инспектирование DOM и CSS

Панель Elements позволяет видеть и редактировать HTML-структуру страницы и применённые CSS-стили в реальном времени.

**Что можно делать:**
- Кликнуть на элемент в браузере и сразу увидеть его в дереве DOM (иконка прицела в левом верхнем углу DevTools)
- Редактировать HTML прямо в панели — двойной клик на теге или атрибуте
- Во вкладке Styles справа видеть все применённые CSS-правила, включая унаследованные
- Включать/выключать CSS-свойства чекбоксами — удобно для экспериментов
- Во вкладке Computed смотреть итоговые вычисленные стили (что реально применилось после каскада)
- Смотреть Box Model — визуальное представление margin, border, padding, content

```javascript
// Полезный трюк: выделить элемент в Elements и получить ссылку на него в консоли
// После выбора элемента в Elements, в Console пишем:
$0 // последний выделенный элемент
$1 // предпоследний
```

**Force State** — принудительно применить псевдокласс к элементу. Правая кнопка на элементе → Force State → `:hover`, `:focus`, `:active`. Незаменимо при отладке hover-эффектов.

---

### Console: вывод, ошибки, интерактив

Console — это не только `console.log`. Это интерактивная среда выполнения JavaScript и многофункциональный инструмент для логирования.

```javascript
// Базовый вывод
console.log('Привет, мир');
console.log('Переменная:', { name: 'Ivan', age: 30 });

// Ошибки и предупреждения (разные иконки и цвета)
console.error('Что-то пошло не так:', error);
console.warn('Это предупреждение');
console.info('Информационное сообщение');

// Таблица — удобно для массивов объектов
const users = [
  { name: 'Ivan', role: 'admin' },
  { name: 'Maria', role: 'user' },
];
console.table(users);
// Выведет красивую таблицу с колонками name и role

// Группировка — убирает шум при множестве логов
console.group('Инициализация приложения');
console.log('Загрузка конфига...');
console.log('Конфиг загружен');
console.groupEnd();

// Свёрнутая группа
console.groupCollapsed('HTTP запросы');
console.log('GET /api/users');
console.log('POST /api/auth');
console.groupEnd();

// Замер времени
console.time('fetchUsers');
await fetchUsers();
console.timeEnd('fetchUsers');
// fetchUsers: 234ms

// Счётчик вызовов
console.count('buttonClick'); // buttonClick: 1
console.count('buttonClick'); // buttonClick: 2
```

В Console можно выполнять JavaScript-код напрямую: получать доступ к переменным, вызывать функции, манипулировать DOM.

---

### Network: анализ запросов

Панель Network показывает все сетевые запросы: HTML, CSS, JS, изображения, API-запросы. Открывать нужно ДО загрузки страницы, иначе ранние запросы не попадут в список.

**Ключевые колонки:**
- **Name** — URL запроса
- **Status** — HTTP статус (200, 404, 500...)
- **Type** — тип ресурса (fetch, xhr, script, stylesheet...)
- **Size** — размер ответа
- **Time** — время загрузки

**Waterfall** — визуализация времени загрузки каждого ресурса. Помогает найти узкие места: что загружается долго, что блокирует рендеринг.

**Детали конкретного запроса** (кликнуть на строку):
- **Headers** — заголовки запроса и ответа, метод, URL, статус
- **Payload** — тело запроса (для POST/PUT)
- **Response** — тело ответа сервера
- **Preview** — отформатированный ответ (для JSON — дерево)
- **Timing** — детальная разбивка времени: DNS Lookup, SSL, TTFB...

```javascript
// Фильтрация запросов:
// Кнопки вверху: XHR/Fetch, JS, CSS, Img, Media, Font, Doc, WS
// Поле поиска принимает URL или -<текст> для исключения
```

**Полезные трюки:**
- Правая кнопка на запросе → Copy → Copy as fetch/curl — получить код для воспроизведения запроса
- Галочка «Preserve log» — не очищать логи при переходах между страницами
- «Disable cache» — отключить кеш при открытых DevTools (обязательно при разработке)
- Throttling — симуляция медленного соединения (3G, Offline)

---

### Application: хранилище данных

Панель Application — это всё, что браузер хранит для вашего сайта.

**Cookies:**
Видны все cookie для текущего домена: имя, значение, домен, путь, срок жизни, флаги HttpOnly/Secure.

```javascript
// В Console можно читать и записывать cookie (без HttpOnly флага)
document.cookie = "theme=dark; max-age=86400; path=/";
console.log(document.cookie);
```

**Local Storage и Session Storage:**
Оба хранят данные в формате ключ-значение. Разница: localStorage сохраняется между сессиями браузера, sessionStorage — только до закрытия вкладки.

```javascript
// LocalStorage
localStorage.setItem('token', 'abc123');
localStorage.getItem('token'); // 'abc123'
localStorage.removeItem('token');
localStorage.clear();

// SessionStorage — тот же API
sessionStorage.setItem('cart', JSON.stringify([{ id: 1 }]));
```

В панели Application можно редактировать и удалять записи вручную — очень удобно при отладке.

**Service Workers:**
Показывает зарегистрированные сервис-воркеры. Можно остановить, обновить, проверить, какие запросы они перехватывают. Кнопка «Update on reload» форсирует обновление воркера при каждой перезагрузке.

---

### Device Toolbar: эмуляция устройств

Иконка смартфона в левом верхнем углу DevTools (или `Cmd+Shift+M`) включает режим эмуляции устройств.

**Возможности:**
- Выбрать конкретное устройство из списка (iPhone 14, Pixel 7, iPad...)
- Установить произвольное разрешение
- Изменить device pixel ratio (retina vs обычный экран)
- Симулировать сенсорные события вместо мышиных
- Настроить ориентацию (portrait/landscape)

Важно понимать: это эмуляция, а не полная симуляция. Производительность, рендеринг шрифтов и поведение некоторых CSS-свойств могут отличаться от реального устройства. Для финальной проверки нужен реальный девайс.

---

## Уровень 2: Отладка кода

### debugger statement

Самый простой способ поставить точку останова — написать `debugger` прямо в коде:

```javascript
function processOrder(order) {
  const total = calculateTotal(order.items);
  debugger; // Выполнение остановится здесь, если DevTools открыты
  return applyDiscount(total, order.discount);
}
```

Когда DevTools открыты и выполнение доходит до `debugger`, браузер останавливается, и вы попадаете в панель Sources. Если DevTools закрыты — `debugger` игнорируется.

---

### Breakpoints: все типы

Панель Sources → выбрать файл → кликнуть на номер строки. Красная точка появится — это breakpoint.

**Line-of-code breakpoint** — самый распространённый. Выполнение остановится на этой строке.

**Conditional breakpoint** — правая кнопка на номере строки → «Add conditional breakpoint». Вводите условие на JavaScript. Выполнение остановится только если условие истинно:

```javascript
// Условие для conditional breakpoint:
user.role === 'admin' && order.total > 1000
// Или для отладки в цикле:
i === 42
```

**Logpoint** — правая кнопка → «Add logpoint». Выводит сообщение в Console без остановки. Удобная альтернатива `console.log`, не требует изменения кода:
```
Пользователь: {user.name}, роль: {user.role}
```

**XHR/Fetch breakpoints** — вкладка Sources → правая панель → «XHR/Fetch Breakpoints» → добавить фильтр URL. Выполнение остановится когда любой fetch/XHR запрос содержит указанную строку в URL:
```
/api/users  →  остановится на всех запросах к /api/users
```

**DOM change breakpoints** — в панели Elements правая кнопка на элементе → «Break on»:
- Subtree modifications — любое изменение внутри элемента
- Attribute modifications — изменение атрибутов
- Node removal — удаление элемента

**Event listener breakpoints** — Sources → правая панель → «Event Listener Breakpoints». Разворачиваете категорию (Mouse, Keyboard, XHR...) и ставите галочку. Выполнение остановится при срабатывании этого события:
```
Mouse → click  →  остановится на любом click-хендлере
```

**Exception breakpoints** — иконка со стоп-знаком в правой панели Sources. Остановить при выброске исключения. «Pause on caught exceptions» — останавливает даже на try/catch.

---

### Call Stack, Scope, Watch

После остановки на breakpoint правая панель Sources показывает три ключевых секции:

**Call Stack** — стек вызовов функций. Показывает, как выполнение пришло к текущей строке:
```
processOrder  ← мы здесь
handleSubmit
onClick
```
Можно кликнуть на любой фрейм и увидеть состояние переменных в том контексте.

**Scope** — все переменные, доступные в текущей точке:
- **Local** — переменные текущей функции
- **Closure** — переменные из замыкания
- **Global** — глобальные переменные (window, document...)

Можно прямо в панели Scope редактировать значения переменных и продолжить выполнение с новыми значениями — очень мощная возможность.

**Watch** — добавить произвольное выражение для отслеживания. Нажать «+» и ввести любое JavaScript-выражение:
```javascript
user.profile.address.city
order.items.length
Date.now() - startTime
```
Значение пересчитывается при каждом шаге отладки.

---

### Управление выполнением: Step Over, Into, Out, Continue

Когда выполнение остановлено, вверху Sources (или в мини-панели) доступны четыре кнопки:

**Resume (F8 / ▶)** — продолжить выполнение до следующего breakpoint. Используйте, когда нашли нужное место и хотите продолжить.

**Step Over (F10 / →)** — выполнить текущую строку и перейти к следующей, не заходя внутрь вызываемых функций. Идеален для прохода по логике на текущем уровне:
```javascript
const result = calculateTotal(items);  // ← Step Over: вычислит и перейдёт дальше
return result;
```

**Step Into (F11 / ↓)** — войти внутрь вызываемой функции. Используйте, когда хотите понять, что происходит внутри:
```javascript
const result = calculateTotal(items);  // ← Step Into: зайдёт в calculateTotal
```

**Step Out (Shift+F11 / ↑)** — выйти из текущей функции и вернуться в вызывающий код. Используйте, когда зашли внутрь функции и хотите выйти обратно.

---

### React DevTools

React DevTools — отдельное расширение для Chrome (устанавливается из Chrome Web Store). Добавляет две новые вкладки в DevTools:

**Components** — дерево React-компонентов. Для каждого компонента показывает:
- **props** — все пропсы с текущими значениями
- **state** — состояние (для классовых компонентов и хуков)
- **hooks** — значения всех хуков (useState, useContext, useRef...)

Можно редактировать props и state прямо в панели — изменения применятся немедленно. Удобно для тестирования пограничных случаев.

**Profiler** — запись и анализ производительности рендеринга:
1. Нажать «Record»
2. Взаимодействовать с приложением
3. Нажать «Stop»
4. Увидеть, какие компоненты рендерились и сколько времени заняло

Flamegraph показывает каждый рендер: серый — компонент не рендерился, цветной — рендерился (чем ярче, тем дольше).

```javascript
// Для удобства отладки называйте функциональные компоненты
// Анонимные компоненты в DevTools показываются как "Anonymous"
const UserCard = ({ user }) => <div>{user.name}</div>;  // ✅ Видно как "UserCard"
export default ({ user }) => <div>{user.name}</div>;    // ⚠️  Видно как "Anonymous"
```

---

## Источники
- [Chrome DevTools — Debugging JavaScript](https://developer.chrome.com/docs/devtools/javascript)
- [Chrome DevTools — Breakpoints](https://developer.chrome.com/docs/devtools/javascript/breakpoints)
