# Производительность, оптимизация

## Уровни навыка
### Уровень 1
Понимание, как браузер подгружает ресурсы на страницу, что следует оптимизировать ассеты

* порядок подгрузка ресурсов на вебстраницу, загрузка скриптов, defer + async
* инструменты оптимизация графики/видео
* использование SVG для плоской графики
* актуальные форматы веб-шрифтов

### Уровень 2
Понимание популярных техник создания производительного приложения

* <picture>, подгрузке адаптивной графики/видео в зависимости от размеров и плотности экрана
* анимируемые CSS-свойства, will-change
* lazy-load картинок, IntersectionObserver
* использование CDN
* анализ бандла, использование code-splitting, tree-shaking, scope hoisting
* понимание работы SSR, регидрации
* requestIdleCallback
* знакомство с инструментами мониторинга: PageSpeed Insights, LightHouse, SpeedCurve, etc
* умение профилировать приложение и искать узкие места
* Кэширующие заголовки HTTP: expires, cache-control, max-age
* Gzip

### Уровень 3
Системный подход к работе над метриками, мониторинг, готовность доносить ценность метрик бизнесу, использование еще более сложных техник

* понимание важности перформанса, влияние скорости на бизнес-метрики (в частности на конверсию)
* метрики: TTI (Time to Interactive), FID (First Input Delay), LCP (Largest Contentful Paint), TBT (Total Blocking Time), CLS (Cumulative Layout Shift), Speed Index
* Core Web Vitals: LCP, FID, CLS
* бюджетирование перформанса

* RAIL: response, animation, idle, load и соответствующие цели по перформансу

* critical HTML/CSS/JavaScript
* Brotli/Gzip
* форматы графики и видео, WebP, AVIF, GIF -> mp4, прогрессивная загрузка изображений, использование фонов/скелетонов, content-visibility
* шрифты: сабсетинг шрифтов, стратегии загрузки, Font Loading API
* modern сборка для ES2017+
* удаление неиспользуемого CSS / JS
* оптимизация JS библиотек, установка более точечных зависимостей
* работа с thirdparty: self-hosting, CSP (Content Security Policy), sandbox
* rendering / decoding тяжелых изображений: content-visibility: auto, contain-intrinsic-size, \<img decoding="async">
* Кэширующий service worker
* HTTP/2, HPACK compression 
* resource hints: dns-lookup, preconnect, prefetch, preload, prerender

### Уровень 4
Системный подход к работе над перформансом, готовность перестраивания архитектуры для улучшения производительности

* выявление средних пользователей и составление чистых / кастомных профилей тестирования (тротлинг сети, тротлинг CPU, установка популярных браузерных расширений). Synthetic testing tools / Real User Monitoring. 
* улучшение производительности API, REST -> graphql
* устранение бутылочных горлышек в js через WebWorkers/WebAssembly
* эвристики для предварительной подгрузки чанков (Guess.js)
* Streaming Server-Side Rendering With Progressive Hydration, Trisomorphic Rendering
* прогрев соединений (dns-prefetch)
* Сертификаты: OCSP, SSL (DV vs EV)
* Сеть: IPv6, TCP BBR, QUIC, HTTP/3 
* Тестирование в прокси- и легаси- браузерах

## Метод оценки

## Как прокачать
### Must See
* <https://www.smashingmagazine.com/2021/01/front-end-performance-2021-free-pdf-checklist/> - основной туториал по всем темам
* <https://web.dev/fast/>

### CSS
* <https://csstriggers.com/> - что вызывает перерисовку