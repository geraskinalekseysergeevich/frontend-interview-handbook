# HTML и CSS: основы

HTML и CSS — фундамент веба. HTML отвечает за структуру и смысл контента, CSS — за внешний вид и расположение элементов. На собеседованиях Junior-уровня эти темы проверяют досконально: знание семантики, модели специфичности, Flexbox и блочной модели отделяет кандидата, который "делал вёрстку", от того, кто понимает, почему она работает именно так.

---

## HTML: семантика

**Семантика** в HTML — это выбор тегов, которые передают смысл контента, а не просто его внешний вид. Браузеру, поисковикам и скринридерам важно понимать, что перед ними: навигация, статья, заголовок или форма.

```html
<!-- Плохо: структура из div'ов без смысла -->
<div class="header">
  <div class="nav">
    <div class="nav-item"><a href="/">Главная</a></div>
  </div>
</div>
<div class="main">
  <div class="post">
    <div class="post-title">Заголовок статьи</div>
    <div class="post-text">Текст...</div>
  </div>
</div>

<!-- Хорошо: семантическая разметка -->
<header>
  <nav>
    <a href="/">Главная</a>
  </nav>
</header>
<main>
  <article>
    <h1>Заголовок статьи</h1>
    <p>Текст...</p>
  </article>
</main>
```

Ключевые семантические элементы HTML5:

| Тег | Назначение |
|-----|-----------|
| `<header>` | Шапка страницы или раздела |
| `<nav>` | Навигационные ссылки |
| `<main>` | Основной контент страницы (один на страницу) |
| `<article>` | Самостоятельная единица контента (статья, пост, комментарий) |
| `<section>` | Тематическая секция с заголовком |
| `<aside>` | Боковой контент, косвенно связанный с основным |
| `<footer>` | Подвал страницы или раздела |
| `<figure>` + `<figcaption>` | Иллюстрация с подписью |
| `<time>` | Дата и время |
| `<mark>` | Выделенный текст |
| `<address>` | Контактная информация |

Заголовки `<h1>`–`<h6>` образуют иерархию документа. На странице должен быть один `<h1>`, далее уровни идут по порядку без пропусков — это влияет на доступность и SEO.

---

## Блочные и строчные элементы

Классически элементы делятся на два типа:

**Блочные (block-level)** — занимают всю доступную ширину родителя, начинаются с новой строки:
- `<div>`, `<p>`, `<h1>`–`<h6>`, `<ul>`, `<ol>`, `<li>`, `<table>`, `<header>`, `<section>` и другие семантические.

**Строчные (inline)** — занимают ровно столько места, сколько нужно содержимому, не начинают новую строку:
- `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`, `<input>`, `<label>`, `<code>`.

```html
<div>Я занимаю всю строку</div>
<div>И я тоже</div>

<span>Мы</span> <span>в одной</span> <span>строке</span>
```

В современном CSS эта граница условная: любой элемент можно сделать блочным или строчным через `display`. Но понимание исходного поведения элементов важно для предсказания результата.

---

## Meta-теги

Мета-теги живут в `<head>` и передают метаданные браузеру, поисковикам и социальным сетям.

```html
<head>
  <!-- Кодировка — всегда первым элементом в <head> -->
  <meta charset="UTF-8">

  <!-- Адаптивность под мобильные устройства — обязательно -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- SEO: описание страницы для поисковиков -->
  <meta name="description" content="Справочник по frontend-разработке">

  <!-- Запрет на кеширование -->
  <meta http-equiv="cache-control" content="no-cache">

  <!-- Open Graph — превью при расшаривании в соцсетях -->
  <meta property="og:title" content="Frontend Handbook">
  <meta property="og:description" content="Теория и практика">
  <meta property="og:image" content="https://example.com/og.png">
  <meta property="og:url" content="https://example.com">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">

  <!-- Заголовок вкладки и закладки -->
  <title>Frontend Handbook</title>

  <!-- Фавиконка -->
  <link rel="icon" href="/favicon.ico">
</head>
```

---

## Атрибуты HTML

Атрибуты задают дополнительные свойства элементов. Некоторые универсальны (global attributes), другие специфичны для конкретных тегов.

```html
<!-- Global attributes — работают на любом элементе -->
<div id="unique-id" class="box active" data-user-id="42"
     hidden tabindex="0" title="Подсказка">Элемент</div>

<!-- role и aria — доступность -->
<div role="button" aria-pressed="false" aria-label="Закрыть">✕</div>

<!-- Специфичные -->
<img src="photo.jpg" alt="Описание для скринридеров и SEO" loading="lazy" width="800" height="600">
<a href="/about" target="_blank" rel="noopener noreferrer">Открыть в новой вкладке</a>
<input type="email" name="email" required autocomplete="email" placeholder="your@email.com">
```

Атрибут `alt` у картинок — не опциональный. Пустой `alt=""` говорит скринридеру "это декоративное изображение, пропусти". Отсутствующий `alt` — ошибка доступности.

`rel="noopener noreferrer"` у ссылок с `target="_blank"` — мера безопасности: без `noopener` открытая страница может получить доступ к `window.opener`.

---

## CSS: способы подключения

CSS можно подключить тремя способами:

```html
<!-- 1. Внешний файл — предпочтительно для большинства стилей -->
<link rel="stylesheet" href="styles.css">

<!-- 2. Инлайн в теге style — для критических стилей или изоляции -->
<style>
  body { margin: 0; }
</style>

<!-- 3. Атрибут style — только для динамических стилей из JS, избегать в HTML -->
<p style="color: red; font-size: 16px;">Текст</p>
```

Внешние файлы кешируются браузером. Инлайн-стили в `<style>` можно использовать для **критического CSS** — минимального набора стилей, необходимого для первого рендера (выше fold), это сокращает CRP.

---

## CSS: специфичность

Специфичность (specificity) — алгоритм, определяющий, какое из конкурирующих CSS-правил применится к элементу. Вычисляется как три числа: **[ID] — [CLASS] — [TYPE]**.

| Тип селектора | Вес |
|--------------|-----|
| Инлайн-стиль (`style=""`) | 1-0-0-0 (особый уровень) |
| ID (`#id`) | 1-0-0 |
| Класс (`.class`), псевдокласс (`:hover`), атрибут (`[type]`) | 0-1-0 |
| Тег (`div`, `p`), псевдоэлемент (`::before`) | 0-0-1 |
| Универсальный (`*`), комбинаторы (`>`, `+`, `~`) | 0-0-0 |

Числа сравниваются слева направо, как числа в разных разрядах:

```css
#header .nav a        /* 1-1-1 */
.nav-list .nav-item a /* 0-2-1 */
/* Побеждает первый: 1 > 0 в колонке ID, остальные не важны */

.card .card__title   /* 0-2-0 */
.card h2             /* 0-1-1 */
/* Побеждает первый: 2 > 1 в колонке CLASS */
```

`!important` выбивает элемент из нормального каскада и применяется в последнюю очередь. Это антипаттерн — при злоупотреблении ломает читаемость стилей. Оправдан только для переопределения стилей сторонних библиотек.

```css
/* Инструмент последней надежды */
.third-party-widget {
  background: white !important;
}
```

---

## Псевдоэлементы и псевдоклассы

**Псевдоклассы** (`:`) — стилизуют элемент в определённом состоянии:

```css
a:hover    { color: blue; }
a:visited  { color: purple; }
a:focus    { outline: 2px solid blue; }
a:active   { color: red; }

input:focus-visible { outline: 3px solid orange; } /* только при клавиатурной навигации */
input:disabled  { opacity: 0.5; }
input:checked   { accent-color: green; }

li:first-child  { font-weight: bold; }
li:last-child   { border-bottom: none; }
li:nth-child(2n) { background: #f5f5f5; } /* чётные */
li:nth-child(3n+1) { color: red; }        /* 1-й, 4-й, 7-й... */
p:not(.special) { color: grey; }
```

**Псевдоэлементы** (`::`) — создают виртуальные элементы внутри или рядом с элементом:

```css
/* ::before и ::after — самые частые, требуют content */
.card::before {
  content: '';           /* обязательно, даже если пустое */
  display: block;
  width: 4px;
  height: 100%;
  background: blue;
}

.price::after {
  content: ' ₽';
}

/* ::first-line — первая строка текста */
p::first-line { font-weight: bold; }

/* ::first-letter — первая буква (drop cap) */
p::first-letter {
  font-size: 3em;
  float: left;
}

/* ::placeholder — стиль подсказки в input */
input::placeholder { color: #999; font-style: italic; }

/* ::selection — выделенный текст */
::selection { background: #ffeb3b; }
```

---

## Блочная модель (Box Model)

Каждый элемент в CSS — прямоугольник, состоящий из четырёх областей (снаружи внутрь): **margin → border → padding → content**.

```css
.box {
  width: 200px;       /* ширина области content */
  height: 100px;
  padding: 16px;      /* внутренний отступ */
  border: 2px solid;  /* граница */
  margin: 24px;       /* внешний отступ */
}
```

По умолчанию (`box-sizing: content-box`) `width` и `height` задают размер только **content-области**. Реальная ширина элемента: `width + padding-left + padding-right + border-left + border-right`.

```css
/* С content-box: реальная ширина = 200 + 32 + 4 = 236px */
.content-box {
  box-sizing: content-box; /* по умолчанию */
  width: 200px;
  padding: 16px;
  border: 2px solid black;
}

/* С border-box: реальная ширина = ровно 200px, padding и border входят внутрь */
.border-box {
  box-sizing: border-box;
  width: 200px;
  padding: 16px;
  border: 2px solid black;
}

/* Современная практика: применять border-box глобально */
*, *::before, *::after {
  box-sizing: border-box;
}
```

`border-box` — де-факто стандарт в современных проектах. Он делает верстку предсказуемой: указал `width: 100%` — и элемент занимает ровно 100% родителя, независимо от padding и border.

---

## Позиционирование

Свойство `position` управляет тем, как элемент располагается в потоке документа.

```css
/* static — по умолчанию, элемент в нормальном потоке */
.default { position: static; }

/* relative — смещение относительно нормальной позиции, место в потоке сохраняется */
.shifted {
  position: relative;
  top: 10px;
  left: 20px;
}

/* absolute — вырывается из потока, позиционируется относительно ближайшего
   предка с position != static (или relative к <body> если такого нет) */
.tooltip {
  position: absolute;
  top: 100%;
  right: 0;
}

/* fixed — относительно viewport, не скроллится вместе со страницей */
.sticky-header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
}

/* sticky — relative до достижения порога, затем фиксируется */
.table-header {
  position: sticky;
  top: 0;    /* фиксируется когда достигает top: 0 при скролле */
  z-index: 1;
}
```

Типичный паттерн: родитель `position: relative` + дочерний `position: absolute`. Это позволяет абсолютно позиционировать элемент (выпадашку, тултип, значок) внутри конкретного контейнера.

```css
.card {
  position: relative; /* контейнер для абсолютного потомка */
}

.card__badge {
  position: absolute;
  top: 8px;
  right: 8px;
}
```

---

## Flexbox

Flexbox — одномерная система расположения элементов (в строку или в колонку). Включается на **контейнере**:

```css
.container {
  display: flex;
  /* или display: inline-flex — если контейнер должен быть строчным */
}
```

### Ось и направление

```css
.container {
  flex-direction: row;            /* → горизонтально (по умолчанию) */
  flex-direction: column;         /* ↓ вертикально */
  flex-direction: row-reverse;    /* ← горизонтально, справа налево */
  flex-direction: column-reverse; /* ↑ вертикально, снизу вверх */
}
```

### Выравнивание

```css
.container {
  /* По главной оси (justify — в направлении flex-direction) */
  justify-content: flex-start;    /* по умолчанию */
  justify-content: flex-end;
  justify-content: center;
  justify-content: space-between; /* промежутки между элементами */
  justify-content: space-around;  /* промежутки вокруг каждого */
  justify-content: space-evenly;  /* равные промежутки везде */

  /* По поперечной оси */
  align-items: stretch;      /* по умолчанию — растянуть по высоте */
  align-items: flex-start;
  align-items: flex-end;
  align-items: center;
  align-items: baseline;     /* по базовой линии текста */
}
```

### Перенос строк

```css
.container {
  flex-wrap: nowrap;        /* по умолчанию — всё в одну строку */
  flex-wrap: wrap;          /* перенос на следующую строку */
  flex-wrap: wrap-reverse;
}
```

### Свойства элементов (flex items)

```css
.item {
  /* flex-grow: коэффициент роста при наличии свободного места */
  flex-grow: 0;   /* по умолчанию — не растёт */
  flex-grow: 1;   /* занимает всё свободное место */
  flex-grow: 2;   /* занимает в 2 раза больше, чем элемент с flex-grow: 1 */

  /* flex-shrink: коэффициент сжатия при нехватке места */
  flex-shrink: 1; /* по умолчанию — сжимается */
  flex-shrink: 0; /* не сжимается */

  /* flex-basis: базовый размер до распределения свободного места */
  flex-basis: auto;   /* по умолчанию — размер по содержимому */
  flex-basis: 200px;

  /* Шорткат: flex: grow shrink basis */
  flex: 1;         /* то же что flex: 1 1 0 */
  flex: auto;      /* то же что flex: 1 1 auto */
  flex: none;      /* то же что flex: 0 0 auto — жёсткий размер */

  /* Выравнивание конкретного элемента по поперечной оси */
  align-self: center;

  /* Порядок элементов без изменения HTML */
  order: 0;   /* по умолчанию */
  order: -1;  /* встанет первым */
  order: 1;   /* встанет после элементов с order: 0 */
}
```

Практический пример — типичный layout:

```css
/* Шапка: лого слева, навигация справа */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 24px;
}

/* Карточки: равномерная сетка с переносом */
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.card {
  flex: 1 1 280px; /* растут, сжимаются, минимум ~280px */
  max-width: 360px;
}

/* Вертикальное центрирование любого содержимого */
.centered {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
```

---

## CSS-переменные (Custom Properties)

CSS-переменные — это пользовательские свойства, объявленные через `--` и доступные через `var()`. Они каскадируются и наследуются как обычные CSS-свойства.

```css
/* Объявление — обычно в :root для глобального доступа */
:root {
  --color-primary: #3b82f6;
  --color-text: #1f2937;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --border-radius: 8px;
  --font-size-base: 16px;
}

/* Использование */
.btn {
  background: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius);
  color: white;
}

/* Второй аргумент var() — значение по умолчанию */
.card {
  color: var(--card-text-color, var(--color-text));
}

/* Переопределение в локальном контексте */
.btn--danger {
  --color-primary: #ef4444; /* переопределяет только для этого элемента и его потомков */
}

/* Тёмная тема через переопределение переменных */
:root {
  --bg: white;
  --text: #1f2937;
}

[data-theme="dark"] {
  --bg: #1f2937;
  --text: #f9fafb;
}

body {
  background: var(--bg);
  color: var(--text);
  /* Переключение темы — просто смена атрибута на <html> или <body> */
}
```

Управление из JavaScript:

```js
// Читать переменную
const root = document.documentElement;
getComputedStyle(root).getPropertyValue('--color-primary').trim();
// → "#3b82f6"

// Устанавливать переменную
root.style.setProperty('--color-primary', '#10b981');

// Динамическая тема
document.documentElement.setAttribute('data-theme', 'dark');
```

Преимущества CSS-переменных перед препроцессорными (SASS/LESS):
- Работают в браузере нативно, доступны в DevTools.
- Могут меняться из JS без перекомпиляции.
- Каскадируются и наследуются — можно переопределять в конкретном компоненте.
- Работают внутри медиа-запросов.

```css
/* Переменная не может так — это ошибка в SASS: */
/* $size: if($mobile, 14px, 16px) — нельзя в медиазапросе */

/* CSS-переменные — могут: */
:root { --font-size: 16px; }

@media (max-width: 768px) {
  :root { --font-size: 14px; }
}

p { font-size: var(--font-size); }
```

---

## Источники

- [Начало работы с HTML — MDN](https://developer.mozilla.org/ru/docs/Learn/HTML/Introduction_to_HTML/Getting_started)
- [Специфичность CSS — MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [Основы Flexbox — MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)
- [Стили и классы — learn.javascript.ru](https://learn.javascript.ru/styles-and-classes)
