# Браузерный рендер

## Уровни навыка
### Уровень 1
* Представляет как браузер преобразовывает байты, полученные по сети в Dom.
* Этапы процесса отрисовки от полученного html/css до изображения в браузере.
* Инструменты:
  * setTimeout
  * setInterval

### Уровень 2
* Влияние загрузки скриптов и стилей на процесс отрисовки web-страницы
* Атрибуты async и defer на скриптах
* Процесс перерисовки изображения web-страницы.
* Процессы reflow и repaint, их отличие, а также какие изменения или действия пользователя к этому приводят
* Влияние исполнения JS на рендеринг web-страницы.
* Инструменты:
  * queueMicrotask
  * requestAnimationFrame
  * requestIdleCallback

Знает, почему для внесения визуальных изменений стоит использовать requestAnimationFrame вместо setTimeout.

Может решить такие проблемы, как "дергающийся макет", подтормаживание визуальных изменений.

### Уровень 3
* Слои и compositor thread и для чего они используются в браузере.
* Оптимизации, которые используют браузеры при перерисовке web-страницы.
* Инструменты оптимизации производительности процесса ререндеринга.

## Как прокачать
* [[html5rocks] How Browsers Work: Behind the scenes of modern web browsers](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)
* [[Web fundamentals] Rendering performance](https://developers.google.com/web/fundamentals/performance/rendering)
* [[Web fundamentals] Critical rendering path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
* [Css triggers](https://csstriggers.com/)
* [Gecko Reflow Visualization](https://www.youtube.com/watch?v=dndeRnzkJDU)
* [Browser Rendering Optimization Free Course](https://www.udacity.com/course/browser-rendering-optimization--ud860)
