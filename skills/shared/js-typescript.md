# JS — TypeScript

[[_TOC_]]

## Уровни навыка
### Уровень 1
*Знание:*

* Разные виды типизации, их отличия:
  * статическая/динамическая;
  * явная/неявная;
* Какая типизация используется в JS и TS.
* Что такое интерфейсы и type-алиасы, чем отличаются друг от друга и для чего используются.
* Основные функции TS:
  * Ключевые слова static, private, etc.
  * Синтаксис .?, !

*Использование:* Способен писать и понимать код с типами (система типизации не важна). Понимает точки кода, где необходима runtime-типизация

### Уровень 2
*Знание:*

* Прочие основные функции TS:
  * Наследование интерфейсов и классов, множественная имплементация
  * Абстрактные классы
  * Тайпгарды
  * Юнионы и intersections типы
  * Const assertion и Enum
* Встроенные утилитарные типы TS
  * Partial<T>
  * Readonly<T>
  * Required<T>
  * Record<T, U>
  * Pick<T, U>
  * Omit<T,K>
  * Exclude<T, U>
  * Extract<T, U>
  * NonNullable<T>
  * ReturnType<T>
  * Parameters<T>
  * InstanceType<T>
  * ThisType<T>
 
*Использование:* Способен покрыть типами внешний код.

### Уровень 3
*Знание:*

* Strict-mode
* Перегрузка функций
* Рекурсивные типы
* Conditional types
* Выводимость типов
* Может писать свои mapped types
  * и свои типы с дженериками

*Использование:* Думает о типах как об эффективном инструменте описания доменной области. Ставит типы в основу процесса проектирования, даже если не использует системы статической типизации. Пишет в строгом режиме (или лоббирует его включение в текущем проекте). Использует все выразительные возможности системы типизации, знает ее слабые места, использует кодогенерацию для обхода слаботипизируемых мест, перепроектирует код так, что максимальное количество кода покрывается минимальными возможностями системы типизации. Типы составляют неотъемлимую часть контракта системы

### Уровень 4
*Знание:*

* Type soundness и Type safety
* Разные виды типизации, из различия:
  * Nominal и Structural Typing
  * слабая/сильная 
* Инвариантность, ковариантность, контравариантность и бивариантность
* Полиморфизм в TS

## Метод оценки

## Как прокачать
### Уровень 1-2:
* [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html) --- basic
* [TypeScript FAQ](https://github.com/microsoft/TypeScript/wiki/FAQ)
* [Ликбез по типизации в языках программирования](https://habr.com/ru/post/161205/)
* [Зачем использовать статические типы в JavaScript?](https://habr.com/ru/post/326304/)
* <https://github.com/typescript-cheatsheets/react>
* <https://github.com/piotrwitek/react-redux-typescript-guide>

### Уровень 3:
* [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/advanced-types.html) --- advanced
* [How 2 TypeScript: Serious business with TypeScript's infer keyword](https://dev.to/miracleblue/how-2-typescript-serious-business-with-typescripts-infer-keyword-40i5)
* [Conditional Types in TypeScript](https://mariusschulz.com/blog/conditional-types-in-typescript)
* [Recursive Type Aliases](https://dev.to/busypeoples/notes-on-typescript-recursive-types-and-immutability-5ck1)
* <https://www.typescript-weekly.com/>
* чтение и понимание исходников <https://github.com/piotrwitek/utility-types>
* вторая статья про Conditional Types: <https://artsy.github.io/blog/2018/11/21/conditional-types-in-typescript>
* never <https://habr.com/ru/post/471026/>
* <https://www.typescriptlang.org/docs/handbook/2/types-from-types.html>
* <https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#keyof-and-lookup-types>
* <https://dev.to/busypeoples/notes-on-typescript-mapped-types-and-lookup-types-i36>

### Уровень 4:
* [What is Type Soundness?](http://jschuster.org/blog/2017/03/21/what-is-type-soundness/)
* [Is there a difference between type safety and type soundness?](https://cs.stackexchange.com/questions/82155/is-there-a-difference-between-type-safety-and-type-soundness)
* [How can a statically typed language support duck typing?](https://softwareengineering.stackexchange.com/questions/252984/how-can-a-statically-typed-language-support-duck-typing)
* [Flavoring: Flexible Nominal Typing for TypeScript](https://spin.atomicobject.com/2018/01/15/typescript-flexible-nominal-typing/)
* [Номинативная типизация в TypeScript или как защитить свой интерфейс от чужих идентификаторов](https://habr.com/ru/post/446768/)
* [Programming TypeScript by Boris Cherny. Chapter 6.](https://learning.oreilly.com/library/view/programming-typescript/9781492037644/ch06.html)
* [What are covariance and contravariance?](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)
* [TS FAQ. Why are function parameters bivariant?](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-function-parameters-bivariant)
* [TS Handbook. Function Parameter Bivariance](https://www.typescriptlang.org/docs/handbook/type-compatibility.html#function-parameter-bivariance)
* [What the heck is polymorphism? (только главы до Row polymorphism включительно)](https://dev.to/jvanbruegge/what-the-heck-is-polymorphism-nmh)
* [Discussion: Typed, modular macros for OCaml, thread about row polymorphism](https://news.ycombinator.com/item?id=13046210)
* <https://habr.com/ru/post/477448/>
* <https://hackernoon.com/learn-advanced-typescript-4yl727e6>

### Дополнительные материалы
* [12 советов по внедрению TypeScript в React-приложениях](https://habr.com/ru/company/tinkoff/blog/505488/)
* [Простые TypeScript-хитрости, которые позволят масштабировать ваши приложения бесконечно](https://habr.com/ru/company/tinkoff/blog/521262/)
* [Тренажер по написанию типов](https://github.com/type-challenges/type-challenges)
