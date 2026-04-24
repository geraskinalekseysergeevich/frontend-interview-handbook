# React — Base

[[_TOC_]]

## Уровни навыка
### Уровень 1
* JSX
  * Особенности JSX (camelCase в атрибутах, название компонентов с прописных букв и т.п.)
  * Virtual DOM — что такое, зачем нужен
* Компоненты
  * Жизненный цикл
  * state и props
  * классовые и функциональные
* props.chidren, что можно передавать, как использовать
  * React.Children 
* чистые функции
  * почему метод render должен быть чистой функцией
* Хуки
  * Базовые — useState, useReducer, useEffect
* Распространённые паттерны
  * HOC
  * Render Props
  * Custom hooks
* Рефы
  * хук useRef
  * React.forwardRef
* Context API

### Уровень 2
* Продвинутые возможности React
  * Portals
  * Error Boundaries
  * React.lazy
* Reconciliation в React
  * Что это такое, как работает
  * Для чего нужен key
* Чистые компоненты
  * Что это такое, как работают?
  * React.memo
  * хуки useMemo и useCallback
* События в React
  * что такое SyntheticEvent
  * особенности работы с нативным addEventListener
* Владение отладкой
* React DevTools

## Метод оценки

## Как прокачать
* [React Documentation](https://reactjs.org/docs/getting-started.html)
* [React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime/)
* [Index as a key is an anti-pattern](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)
* [React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture) - здесь неплохо написано про reconciliation в целом, часть про детали реализации (fiber) опциональна.
* [React events in depth w/ Kent C. Dodds, Ben Alpert, & Dan Abramov](https://www.youtube.com/watch?v=dRo_egw7tBc)
* [Getting to know React DOM's event handling system inside out](https://medium.com/the-guild/getting-to-know-react-doms-event-handling-system-inside-out-378c44d2a5d0)
* [Новый контекст React в деталях](https://blog.csssr.ru/2018/04/06/new-react-context)