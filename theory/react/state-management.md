# React: Управление состоянием

Управление состоянием — одна из тех тем, где большинство ошибок совершается не из незнания синтаксиса, а из-за неправильной архитектуры. Положить лишнее в state, продублировать данные в двух местах, не поднять state вовремя — это не баги компилятора, это баги мышления. В этом файле — принципы, которые помогают думать о state правильно.

---

## Локальный state (useState): когда использовать

`useState` подходит, когда:

- Данные нужны только этому компоненту (или ближайшим детям через props)
- Изменение значения должно вызывать перерисовку
- Данные не нужно шарить с несвязанными частями приложения

```jsx
// Правильное использование useState — локальная видимость формы
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await api.login({ email, password });
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button disabled={isSubmitting}>
        {isSubmitting ? 'Вход...' : 'Войти'}
      </button>
    </form>
  );
}
```

**Когда useState недостаточно:**
- Данные нужны далеко в дереве компонентов → используйте Context или поднятие state
- Сложная логика переходов между состояниями → рассмотрите `useReducer`
- Данные нужны в нескольких несвязанных частях приложения → внешний стор (Zustand, Redux)

---

## useRef vs useState: разница в назначении

Оба сохраняют значение между рендерами. Разница в одном: `useState` при изменении вызывает ре-рендер, `useRef` — нет.

| | useState | useRef |
|---|---|---|
| Вызывает ре-рендер | Да | Нет |
| Используется в JSX | Да | Нет (обычно) |
| Изменение | Через setter | Прямая мутация `ref.current` |
| Читать во время рендера | Да | Не рекомендуется |

**Используйте `useRef`, когда значение нужно компоненту, но не влияет на отображение:**

```jsx
function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  // intervalId не нужен для рендеринга — он нужен только чтобы его остановить
  const intervalRef = useRef(null);

  function handleStart() {
    const start = Date.now();
    setStartTime(start);
    setNow(start);

    intervalRef.current = setInterval(() => {
      setNow(Date.now()); // вот здесь нужен state — экран должен обновляться
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current); // используем сохранённый ID
  }

  const elapsed = startTime && now ? (now - startTime) / 1000 : 0;

  return (
    <div>
      <p>{elapsed.toFixed(2)}с</p>
      <button onClick={handleStart}>Старт</button>
      <button onClick={handleStop}>Стоп</button>
    </div>
  );
}
```

**Классическая ошибка** — попытаться использовать ref вместо state для отображаемых данных:

```jsx
// Сломанный вариант — счётчик никогда не обновится на экране
function BrokenCounter() {
  const countRef = useRef(0);

  return (
    <button onClick={() => countRef.current++}>
      Нажато: {countRef.current} {/* Всегда 0 — ре-рендера нет */}
    </button>
  );
}

// Правильный вариант
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Нажато: {count}
    </button>
  );
}
```

**Другой частый случай для `useRef`** — предыдущее значение state:

```jsx
function PriceDisplay({ price }) {
  const prevPriceRef = useRef(price);

  useEffect(() => {
    prevPriceRef.current = price; // обновляем после каждого рендера
  });

  const prevPrice = prevPriceRef.current;
  const direction = price > prevPrice ? '↑' : price < prevPrice ? '↓' : '→';

  return (
    <span>
      {price} {direction}
    </span>
  );
}
```

---

## Single Source of Truth (SSOT)

**Single Source of Truth** — один из базовых принципов React: каждый кусок данных должен жить в одном-единственном месте. Если одни и те же данные хранятся в двух местах одновременно, они рано или поздно расходятся.

**Нарушение SSOT — дублирование данных:**

```jsx
// ПЛОХО: fullName — дубликат firstName + lastName
function NameForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState(''); // лишний state!

  return (
    <div>
      <input
        value={firstName}
        onChange={(e) => {
          setFirstName(e.target.value);
          setFullName(e.target.value + ' ' + lastName); // синхронизируем вручную
        }}
      />
      <input
        value={lastName}
        onChange={(e) => {
          setLastName(e.target.value);
          setFullName(firstName + ' ' + e.target.value); // и снова вручную
        }}
      />
      <p>Полное имя: {fullName}</p>
    </div>
  );
}

// ХОРОШО: fullName вычисляется, а не хранится
function NameForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName; // вычисляем при каждом рендере

  return (
    <div>
      <input value={firstName} onChange={(e) => setFirstName(e.target.value)} />
      <input value={lastName} onChange={(e) => setLastName(e.target.value)} />
      <p>Полное имя: {fullName}</p>
    </div>
  );
}
```

**Нарушение SSOT — зеркалирование props в state:**

```jsx
// ПЛОХО: color "застынет" при первом рендере и не обновится при изменении пропа
function ColorBadge({ color }) {
  const [badgeColor, setBadgeColor] = useState(color); // зеркало пропа — ошибка

  return <span style={{ backgroundColor: badgeColor }}>badge</span>;
}

// ХОРОШО: используем проп напрямую
function ColorBadge({ color }) {
  return <span style={{ backgroundColor: color }}>badge</span>;
}

// ИСКЛЮЧЕНИЕ: если вам нужно начальное значение, которое потом управляется независимо
// Тогда называйте проп явно "initial" чтобы это было понятно
function EditableColor({ initialColor }) {
  const [color, setColor] = useState(initialColor); // окей, это начальное значение

  return (
    <input
      type="color"
      value={color}
      onChange={(e) => setColor(e.target.value)}
    />
  );
}
```

**Нарушение SSOT — дублирование объектов:**

```jsx
// ПЛОХО: selectedItem дублирует объект из массива items
function ItemList({ items }) {
  const [selectedItem, setSelectedItem] = useState(items[0]);
  // Если items обновится извне — selectedItem устареет и не обновится

  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          className={item.id === selectedItem.id ? 'selected' : ''}
          onClick={() => setSelectedItem(item)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// ХОРОШО: храним только ID, сам объект берём из массива
function ItemList({ items }) {
  const [selectedId, setSelectedId] = useState(items[0]?.id);
  const selectedItem = items.find(item => item.id === selectedId); // вычисляем

  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          className={item.id === selectedId ? 'selected' : ''}
          onClick={() => setSelectedId(item.id)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## Производные данные: не хранить то, что вычисляется

Производные данные — это то, что можно получить из уже существующего state или props. Хранить их в отдельном state — это нарушение SSOT и источник трудноуловимых багов.

**Правило**: если значение можно вычислить из state или props в момент рендера — вычисляйте его там, а не кладите в state.

```jsx
// ПЛОХО: filteredTodos — производное от todos и filter
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  const [filteredTodos, setFilteredTodos] = useState([]); // лишний state

  // Приходится синхронизировать в useEffect — это антипаттерн
  useEffect(() => {
    if (filter === 'all') setFilteredTodos(todos);
    else if (filter === 'done') setFilteredTodos(todos.filter(t => t.done));
    else setFilteredTodos(todos.filter(t => !t.done));
  }, [todos, filter]);

  return (/* ... */);
}

// ХОРОШО: вычисляем напрямую при рендере
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  // Вычисляется синхронно, не нужен дополнительный state и useEffect
  const filteredTodos = todos.filter(todo => {
    if (filter === 'all') return true;
    if (filter === 'done') return todo.done;
    return !todo.done;
  });

  return (
    <div>
      <div>
        {['all', 'done', 'active'].map(f => (
          <button key={f} onClick={() => setFilter(f)}>{f}</button>
        ))}
      </div>
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Когда вычисление дорогое** — используйте `useMemo`, чтобы не пересчитывать при каждом рендере:

```jsx
import { useMemo } from 'react';

function ExpensiveList({ items, searchQuery }) {
  // Пересчитывается только когда items или searchQuery изменились
  const filteredItems = useMemo(
    () => items.filter(item =>
      item.name.toLowerCase().includes(searchQuery.toLowerCase())
    ),
    [items, searchQuery]
  );

  return (
    <ul>
      {filteredItems.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

Не злоупотребляйте `useMemo` — добавляйте его только туда, где профилировщик показал реальную проблему с производительностью.

---

## Загрузка данных с сервера: loading / error / data

Запрос к API — это всегда три состояния: загружается, ошибка, данные. Самый простой способ — три отдельных state:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false; // флаг для предотвращения race condition

    setIsLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => {
        if (!cancelled) {
          setUser(data);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err.message);
        }
      })
      .finally(() => {
        if (!cancelled) {
          setIsLoading(false);
        }
      });

    return () => {
      cancelled = true; // отменяем обработку, если userId изменился
    };
  }, [userId]);

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Race Conditions: почему это важно

**Race condition** — ситуация, когда два запроса отправлены, но ответы пришли в другом порядке. Без защиты старый ответ перезаписывает новый.

Сценарий: пользователь открыл профиль `userId=1`, потом быстро переключился на `userId=2`. Запрос для `userId=2` может прийти раньше запроса для `userId=1` — и тогда на экране отобразится профиль второго, а затем поверх него запишется профиль первого.

```jsx
// Защита через флаг cancelled (как в примере выше)
useEffect(() => {
  let cancelled = false;

  async function loadUser() {
    try {
      const data = await fetchUser(userId);
      if (!cancelled) setUser(data); // проверяем актуальность
    } catch (e) {
      if (!cancelled) setError(e.message);
    }
  }

  loadUser();

  return () => {
    cancelled = true; // при следующем userId этот эффект "протухает"
  };
}, [userId]);
```

**Лучший вариант для продакшна** — использовать библиотеки, которые управляют этим за вас: **React Query (TanStack Query)** или **SWR**. Они также дают кэширование, дедупликацию запросов и инвалидацию.

```jsx
// С React Query race conditions, загрузка и ошибки — из коробки
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
  });

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error.message} />;

  return <div>{user.name}</div>;
}
```

---

## Поднятие state (Lifting State Up)

Если два компонента должны работать с одними и теми же данными — state нужно поднять к их ближайшему общему родителю. Дочерние компоненты получают данные через props и сообщают об изменениях через callback-функции.

**Пример: синхронизация двух полей ввода температуры:**

```jsx
// Controlled компонент — управляется снаружи через props
function TemperatureInput({ label, value, onChange }) {
  return (
    <div>
      <label>{label}</label>
      <input
        type="number"
        value={value}
        onChange={(e) => onChange(e.target.value)}
      />
    </div>
  );
}

// Родитель держит state и синхронизирует оба поля
function TemperatureConverter() {
  const [celsius, setCelsius] = useState('');

  const toCelsius = (f) => ((Number(f) - 32) * 5) / 9;
  const toFahrenheit = (c) => (Number(c) * 9) / 5 + 32;

  const fahrenheit = celsius !== '' ? toFahrenheit(Number(celsius)) : '';

  return (
    <div>
      <TemperatureInput
        label="Цельсий"
        value={celsius}
        onChange={(val) => setCelsius(val)}
      />
      <TemperatureInput
        label="Фаренгейт"
        value={fahrenheit}
        onChange={(val) => setCelsius(String(toCelsius(val)))}
      />
    </div>
  );
}
```

**Пример: аккордеон, где раскрывается только одна панель:**

```jsx
// Panel — контролируемый компонент
function Panel({ title, isOpen, onToggle, children }) {
  return (
    <div className="panel">
      <button onClick={onToggle}>
        {title} {isOpen ? '▲' : '▼'}
      </button>
      {isOpen && <div className="panel-body">{children}</div>}
    </div>
  );
}

// Accordion держит state — какая панель открыта
function Accordion() {
  const [openIndex, setOpenIndex] = useState(null);

  const panels = [
    { title: 'О компании', content: 'Мы делаем крутые вещи...' },
    { title: 'Контакты', content: 'Email: hello@example.com' },
    { title: 'Вакансии', content: 'Всегда ищем таланты...' },
  ];

  return (
    <div>
      {panels.map((panel, index) => (
        <Panel
          key={index}
          title={panel.title}
          isOpen={openIndex === index}
          onToggle={() => setOpenIndex(openIndex === index ? null : index)}
        >
          {panel.content}
        </Panel>
      ))}
    </div>
  );
}
```

**Когда поднимать state:**
- Два компонента должны реагировать на одно и то же изменение данных
- Один компонент должен влиять на другой
- Нужна синхронизация — оба компонента отображают одно и то же

**Когда НЕ поднимать:** если поднятие уводит state слишком далеко вверх и вызывает prop drilling через много уровней — рассмотрите Context. Но сначала попробуйте поднять state и передать через props: это явно, предсказуемо и легко отследить.

---

## Типичные нарушения и антипаттерны

Краткий список проблем, с которыми сталкиваются при работе с state:

**1. Слишком много state — вычисляйте, а не храните:**
```jsx
// Плохо
const [items, setItems] = useState([]);
const [count, setCount] = useState(0); // count = items.length

// Хорошо
const [items, setItems] = useState([]);
const count = items.length; // вычисляется автоматически
```

**2. Противоречивые состояния — объединяйте в одну переменную:**
```jsx
// Плохо — может быть isLoading=true и isError=true одновременно
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);

// Хорошо — только одно состояние активно
const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'error' | 'success'
```

**3. Синхронизация через useEffect — признак лишнего state:**
```jsx
// Плохо — видите useEffect, который только вызывает setState?
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// Хорошо — вычислите напрямую
const fullName = firstName + ' ' + lastName;
```

**4. Глубокая вложенность — нормализуйте:**
```jsx
// Плохо — обновление вложенного объекта требует раскрытия всей цепочки
const [cart, setCart] = useState({
  user: { name: 'Алексей' },
  items: [{ id: 1, product: { name: 'Книга', price: 500 } }]
});

// Лучше — плоская структура, обновлять проще
const [cartItems, setCartItems] = useState([]);
const [userId, setUserId] = useState(null);
```

---

## Источники

- [Choosing the State Structure — react.dev](https://react.dev/learn/choosing-the-state-structure)
- [Sharing State Between Components — react.dev](https://react.dev/learn/sharing-state-between-components)
- [You Might Not Need an Effect — react.dev](https://react.dev/learn/you-might-not-need-an-effect)
- [Referencing Values with Refs — react.dev](https://react.dev/learn/referencing-values-with-refs)
