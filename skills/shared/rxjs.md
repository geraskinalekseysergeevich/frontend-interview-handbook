# RxJS

## Уровни навыка
### Уровень 1
* Интерфейсы Observable и Observer в Rx
* Разница между Observable и Promise.
* Знает, что такое Subject и какие бывают подвиды.
* Знает базовые операторы вида: map(), filter(), take(), forkJoin(), zip(), combineLatest() и прочие.

### Уровень 2
* Паттерны Observable, Observer, Event Emitter
* Горячие и холодные Observable, Multicasting.
* Может создать observable через конструктор new Observable().
* Higher Order Observables: операторы, особенности подписки, обработка ошибок.
* Конечные и бесконечные Observable, состояния потока, обработка ошибок.
* Объект Subscription: назначение, возможные варианты отписки от потока и их особенности.
* Есть практический опыт широкого спектра операторов, что подтверждается знанием специфики таких операторов, как scan и reduce, distict'ы.
* Тестирование observables
* Marble diagrams

### Уровень 3
* Синхронность и асинхронность в RxJS.
* Знает, как реализовать кастомные операторы.
* Schedulers в RxJS.
* Хорошо знаком со спецификацией основных операторов. Учитывает эти особенности в работе системы. Примеры:
  * shareReplay - утечки памяти, возникающие при отписке от незавершенного потока.
* Руководствуется практиками ФП при формировании пайплайнов обработки событий в потоке. Аккуратно работает с side effects в потоке.

## Как прокачать
* [Learn RxJS](https://www.learnrxjs.io/)
* [Egghead Rxjs Cources](https://egghead.io/search?topic=rxjs)
* [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
* [Strongbrew: Примеры решения практических задач](https://blog.strongbrew.io/tag/RxJS/)
* [Hot vs Cold Observables](https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339#.8x9uam5rg)
* [RxJS Subjects](https://aalexeev239.github.io/rxjs-subjects/)
* [RxJS: Higher Order Observables](https://aalexeev239.github.io/rxjs-hoo/)
* [Как выбрать правильный оператор?](https://rxjs-dev.firebaseapp.com/operator-decision-tree)
* [Сценарии использования RxJS](https://www.learnrxjs.io/learn-rxjs/recipes)
* <https://rxviz.com/>
* [[Github][r.sedov & a.inkin] RxJS Challenge](https://github.com/AngularWave/rxjs-challenge)