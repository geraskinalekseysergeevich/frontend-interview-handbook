# Тестирование: Jest и пирамида тестов

Написать код — это полдела. Убедиться, что он работает правильно сейчас и не сломается через месяц после очередного рефакторинга — вот настоящая задача. Автоматические тесты дают уверенность в коде и превращают страшный «а вдруг что-то сломается?» в спокойное «тесты покажут».

---

## Зачем вообще писать тесты

**Уверенность при изменениях.** Вы добавляете новую функцию и хотите убедиться, что старое не сломалось. Без тестов — ручная проверка всего приложения. С тестами — `npm test`, и через секунды видите, всё ли в порядке.

**Облегчение рефакторинга.** Хотите переписать функцию, чтобы она была чище? С тестами можно смело менять реализацию, если тесты проходят — поведение сохранено.

**Живая документация.** Тесты показывают, как должен вести себя код в разных ситуациях. Это лучше, чем комментарии, потому что тесты нельзя забыть обновить (иначе они сломаются).

**Раннее обнаружение багов.** Баг, найденный до деплоя, дешевле бага в продакшене в 10-100 раз.

---

## Пирамида тестирования

```
        /\
       /E2E\         ← мало, медленные, дорогие
      /------\
     /Integr- \      ← средне
    / ation    \
   /------------\
  /   Unit Tests \   ← много, быстрые, дешёвые
 /________________\
```

**Unit-тесты** — проверяют одну изолированную функцию или компонент. Зависимости заменяются моками. Работают быстро (миллисекунды), их должно быть большинство.

```javascript
// Пример: тест чистой функции
function add(a, b) { return a + b; }
test('add(2, 3) returns 5', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Интеграционные тесты** — проверяют взаимодействие нескольких модулей. Например, форма + валидация + отправка запроса. Медленнее unit, но ближе к реальности.

**E2E тесты (End-to-End)** — проверяют всё приложение целиком через браузер (Cypress, Playwright). Самые медленные и дорогие в поддержке, но дают максимальную уверенность.

**Правило пирамиды:** большинство тестов — unit, меньше интеграционных, совсем немного E2E. Если перевернуть пирамиду (много E2E, мало unit) — получите медленный и хрупкий тестовый сьют.

---

## Jest: основы

Jest — самый популярный фреймворк для тестирования JavaScript. Разработан Facebook, отлично работает с React, TypeScript, Node.js.

### Установка

```bash
npm install --save-dev jest
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

### Структура теста: describe, it, test

```javascript
// describe — группирует связанные тесты
describe('Функция calculateTotal', () => {

  // it и test — синонимы, используйте что удобнее
  it('считает сумму товаров', () => {
    const items = [{ price: 100 }, { price: 200 }];
    expect(calculateTotal(items)).toBe(300);
  });

  test('возвращает 0 для пустого массива', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('игнорирует товары без цены', () => {
    const items = [{ price: 100 }, { name: 'broken item' }];
    expect(calculateTotal(items)).toBe(100);
  });
});
```

### Матчеры (Matchers)

Матчеры — это методы `expect()`, которые проверяют конкретное условие.

```javascript
// toBe — строгое равенство (===), для примитивов
expect(2 + 2).toBe(4);
expect('hello').toBe('hello');

// toEqual — глубокое сравнение, для объектов и массивов
expect({ a: 1, b: 2 }).toEqual({ a: 1, b: 2 });
expect([1, 2, 3]).toEqual([1, 2, 3]);

// Не путайте: toBe для объектов сравнивает ссылки, а не содержимое!
expect({ a: 1 }).toBe({ a: 1 });    // ❌ FAIL — разные объекты
expect({ a: 1 }).toEqual({ a: 1 }); // ✅ PASS — одинаковое содержимое

// Проверка истинности
expect(true).toBeTruthy();
expect(0).toBeFalsy();
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect('hello').toBeDefined();

// Числа
expect(5).toBeGreaterThan(4);
expect(5).toBeGreaterThanOrEqual(5);
expect(5).toBeLessThan(6);
expect(0.1 + 0.2).toBeCloseTo(0.3); // для float!

// Строки
expect('Hello, world').toMatch(/world/);
expect('Hello, world').toContain('world');

// Массивы
expect([1, 2, 3]).toContain(2);
expect([1, 2, 3]).toHaveLength(3);

// Исключения — функцию нужно обернуть!
expect(() => {
  throw new Error('Ошибка валидации');
}).toThrow('Ошибка валидации');

expect(() => JSON.parse('invalid json')).toThrow(SyntaxError);

// Отрицание через .not
expect(5).not.toBe(4);
expect([]).not.toContain(1);
```

---

## Моки: jest.fn()

Мок — это замена реальной зависимости на «пустышку», которая ведёт себя предсказуемо и позволяет отслеживать вызовы.

```javascript
// Создать mock-функцию
const mockFn = jest.fn();

// Вызвать её
mockFn('hello', 42);
mockFn('world');

// Проверить вызовы
expect(mockFn).toHaveBeenCalled();              // вызвана хотя бы раз
expect(mockFn).toHaveBeenCalledTimes(2);        // вызвана ровно 2 раза
expect(mockFn).toHaveBeenCalledWith('hello', 42); // вызвана с этими аргументами
expect(mockFn).toHaveBeenLastCalledWith('world'); // последний вызов с этим аргументом

// Настроить возвращаемое значение
const fetchUser = jest.fn().mockReturnValue({ id: 1, name: 'Ivan' });
const fetchUserAsync = jest.fn().mockResolvedValue({ id: 1, name: 'Ivan' });

// Разные значения при последовательных вызовах
const mock = jest.fn()
  .mockReturnValueOnce('first')
  .mockReturnValueOnce('second')
  .mockReturnValue('default');

mock(); // 'first'
mock(); // 'second'
mock(); // 'default'
mock(); // 'default'
```

---

## Примеры тестов

### Тест JS-функции

```javascript
// src/utils/formatPrice.js
export function formatPrice(amount, currency = 'RUB') {
  if (typeof amount !== 'number' || isNaN(amount)) {
    throw new Error('amount must be a number');
  }
  return new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency,
  }).format(amount);
}

// src/utils/formatPrice.test.js
import { formatPrice } from './formatPrice';

describe('formatPrice', () => {
  it('форматирует цену в рублях', () => {
    expect(formatPrice(1000)).toBe('1 000,00 ₽');
  });

  it('форматирует цену в другой валюте', () => {
    expect(formatPrice(50, 'USD')).toContain('$');
  });

  it('выбрасывает ошибку для нечисловых значений', () => {
    expect(() => formatPrice('abc')).toThrow('amount must be a number');
    expect(() => formatPrice(null)).toThrow();
    expect(() => formatPrice(NaN)).toThrow();
  });

  it('корректно работает с нулём', () => {
    expect(formatPrice(0)).toBeDefined();
  });
});
```

### Тест React-компонента

```javascript
// src/components/Button.jsx
export function Button({ label, onClick, disabled = false }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// src/components/Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('рендерит текст кнопки', () => {
    render(<Button label="Отправить" onClick={() => {}} />);
    expect(screen.getByText('Отправить')).toBeInTheDocument();
  });

  it('вызывает onClick при клике', () => {
    const handleClick = jest.fn();
    render(<Button label="Кликни" onClick={handleClick} />);
    fireEvent.click(screen.getByText('Кликни'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('не вызывает onClick когда disabled', () => {
    const handleClick = jest.fn();
    render(<Button label="Кнопка" onClick={handleClick} disabled />);
    fireEvent.click(screen.getByText('Кнопка'));
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

---

## Источники
- [Jest — Getting Started](https://jestjs.io/docs/getting-started)
- [Jest — Using Matchers](https://jestjs.io/docs/using-matchers)
- [Jest — Mock Functions](https://jestjs.io/docs/mock-functions)
