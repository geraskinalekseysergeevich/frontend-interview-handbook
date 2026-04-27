# React: Основы

React — это библиотека для построения пользовательских интерфейсов. Главная идея: UI строится из маленьких независимых кирпичиков — **компонентов**. Каждый компонент отвечает за свой кусок экрана, управляет своими данными и знает, как перерисоваться при изменении состояния. Это не фреймворк с магией — это инструмент, который делает ровно то, что вы ему говорите.

---

## JSX: HTML внутри JavaScript

JSX — это синтаксический сахар над `React.createElement`. Выглядит как HTML, но это JavaScript. Браузер не понимает JSX напрямую — Babel или TypeScript транспилируют его в вызовы функций.

```jsx
// Это JSX
const element = <h1 className="title">Привет, мир</h1>;

// После компиляции это превращается примерно в:
const element = React.createElement('h1', { className: 'title' }, 'Привет, мир');
```

**Отличия JSX от HTML:**

- `class` → `className` (потому что `class` — зарезервированное слово в JS)
- `for` → `htmlFor` у `<label>`
- Все атрибуты в camelCase: `onclick` → `onClick`, `tabindex` → `tabIndex`, `stroke-width` → `strokeWidth`
- Теги всегда должны быть закрыты: `<img />`, `<br />`
- Выражения вставляются через фигурные скобки: `{user.name}`

```jsx
function UserCard({ user }) {
  return (
    <div className="card">
      <img src={user.avatar} alt={user.name} />
      <label htmlFor="email">Email</label>
      <input id="email" type="email" defaultValue={user.email} />
    </div>
  );
}
```

**Фрагменты** нужны, когда компонент возвращает несколько элементов, но вы не хотите лишний `<div>` в DOM:

```jsx
// Короткий синтаксис
function Row() {
  return (
    <>
      <td>Ячейка 1</td>
      <td>Ячейка 2</td>
    </>
  );
}

// Полный синтаксис (нужен когда нужен key)
function List({ items }) {
  return items.map(item => (
    <React.Fragment key={item.id}>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </React.Fragment>
  ));
}
```

---

## Virtual DOM: зачем нужен посредник

Работа с реальным DOM дорогая: каждое изменение может вызвать перерасчёт стилей, перестройку layout и перерисовку страницы. React решает это через **Virtual DOM** — это лёгкая JavaScript-копия реального дерева DOM.

Процесс работы:

1. Компонент возвращает JSX — React строит виртуальное дерево объектов
2. При изменении state создаётся **новое** виртуальное дерево
3. React сравнивает два дерева (алгоритм **reconciliation** / **diffing**)
4. Находит минимальный набор изменений
5. Применяет только эти изменения к реальному DOM

**Главный выигрыш**: React не перерисовывает весь UI — он обновляет только те узлы, которые реально изменились.

```jsx
// При изменении count React обновит только текст внутри <span>,
// а не перерисует весь компонент целиком
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Счётчик</h1>  {/* этот элемент не изменится */}
      <span>{count}</span>  {/* обновится только это */}
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

---

## Компоненты: функциональные и классовые

Сегодня **функциональные компоненты** — стандарт. Они лаконичнее, легче тестируются и поддерживают хуки. Классовые компоненты существуют в легаси-коде и знать их нужно, чтобы читать старые проекты.

**Функциональный компонент** — это просто функция, которая принимает props и возвращает JSX:

```jsx
// Современный способ
function Greeting({ name }) {
  return <h1>Привет, {name}!</h1>;
}

// Или стрелочная функция
const Greeting = ({ name }) => <h1>Привет, {name}!</h1>;
```

**Классовый компонент** — класс, наследующийся от `React.Component`:

```jsx
// Устаревший способ — только для чтения легаси-кода
class Greeting extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

**Когда использовать классовый компонент сегодня**: только если вам нужен `ErrorBoundary` — это единственное, что нельзя сделать функциональным компонентом (пока нет аналога `componentDidCatch` через хуки).

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error('Поймана ошибка:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }
    return this.props.children;
  }
}
```

---

## Props: данные сверху вниз

Props — это параметры компонента. Данные текут **сверху вниз**: родитель передаёт props дочернему компоненту, дочерний не может их изменить. Props — иммутабельны с точки зрения компонента.

```jsx
// Передача props
function App() {
  return (
    <UserProfile
      name="Алексей"
      age={28}
      isAdmin={true}
      hobbies={['код', 'кофе']}
    />
  );
}

// Получение через деструктуризацию
function UserProfile({ name, age, isAdmin, hobbies }) {
  return (
    <div>
      <h2>{name}, {age} лет</h2>
      {isAdmin && <span className="badge">Администратор</span>}
      <ul>
        {hobbies.map(hobby => <li key={hobby}>{hobby}</li>)}
      </ul>
    </div>
  );
}

// Значения по умолчанию
function Button({ label = 'Нажми меня', variant = 'primary', onClick }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
}
```

**`props.children`** — это всё, что вы вложили между открывающим и закрывающим тегом компонента:

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <div className="card-header">{title}</div>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Использование
function App() {
  return (
    <Card title="Новость">
      <p>Текст новости здесь</p>
      <a href="/more">Читать далее</a>
    </Card>
  );
}
```

---

## State: память компонента

Если props — это данные снаружи, то **state** — это внутренняя память компонента. Когда state меняется, React перерисовывает компонент.

Обычная переменная не работает как state: React не знает, что она изменилась, и не запустит перерисовку.

```jsx
import { useState } from 'react';

function PasswordInput() {
  // [текущее значение, функция обновления] = useState(начальное значение)
  const [password, setPassword] = useState('');
  const [isVisible, setIsVisible] = useState(false);

  return (
    <div>
      <input
        type={isVisible ? 'text' : 'password'}
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Введите пароль"
      />
      <button onClick={() => setIsVisible(!isVisible)}>
        {isVisible ? 'Скрыть' : 'Показать'}
      </button>
    </div>
  );
}
```

**Когда использовать state**: когда компонент должен «помнить» какое-то значение между рендерами и изменение этого значения должно вызывать перерисовку. Не кладите в state то, что можно вычислить из уже существующих данных — об этом подробнее в файле по управлению состоянием.

---

## Жизненный цикл через useEffect

У классовых компонентов были методы `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`. В функциональных компонентах это всё заменяет `useEffect`.

**Монтирование (mount)** — компонент появился на экране:

```jsx
useEffect(() => {
  console.log('Компонент смонтирован');
  document.title = 'Новая страница';
}, []); // Пустой массив = только при монтировании
```

**Обновление (update)** — что-то изменилось:

```jsx
useEffect(() => {
  document.title = `Вы вошли как ${username}`;
}, [username]); // Запускается при изменении username
```

**Размонтирование (unmount)** — компонент удаляется с экрана. Функция очистки выполняется перед удалением:

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setTime(new Date());
  }, 1000);

  // Функция очистки — обязательна для таймеров, подписок, слушателей
  return () => {
    clearInterval(timer);
  };
}, []);
```

**Полный пример с подпиской на WebSocket:**

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const socket = new WebSocket(`wss://chat.example.com/${roomId}`);

    socket.onmessage = (event) => {
      setMessages(prev => [...prev, JSON.parse(event.data)]);
    };

    // Очищаем подключение при смене комнаты или размонтировании
    return () => {
      socket.close();
    };
  }, [roomId]); // При смене roomId — пересоздаём подключение

  return (
    <ul>
      {messages.map((msg, i) => <li key={i}>{msg.text}</li>)}
    </ul>
  );
}
```

---

## Хуки

Хуки — это функции, которые «подключают» компоненты к возможностям React. Работают только на верхнем уровне компонента, не внутри условий или циклов.

### useState

Уже разобрали выше. Хранит значение между рендерами и вызывает перерисовку при изменении.

### useEffect

Синхронизация с внешним миром: таймеры, API, подписки, браузерные API. Разобрали выше.

### useReducer

Когда state — это объект со сложной логикой переходов, `useReducer` удобнее `useState`:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'decrement':
      return { ...state, count: state.count - 1 };
    case 'reset':
      return { count: 0, history: [] };
    default:
      throw new Error(`Неизвестное действие: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, history: [] });

  return (
    <div>
      <p>Счёт: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Сброс</button>
    </div>
  );
}
```

### useRef

Хранит мутабельное значение, которое **не вызывает перерисовку** при изменении. Подробнее — в разделе про рефы.

### useContext

Читает значение из ближайшего Provider. Подробнее — в разделе про Context API.

---

## Паттерны компонентов

### HOC (Higher-Order Component)

Функция, которая принимает компонент и возвращает новый компонент с дополнительными возможностями:

```jsx
// HOC для добавления проверки авторизации
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { user } = useAuth();

    if (!user) {
      return <Navigate to="/login" />;
    }

    return <WrappedComponent {...props} user={user} />;
  };
}

// Использование
const ProtectedDashboard = withAuth(Dashboard);

// В роутере
<Route path="/dashboard" element={<ProtectedDashboard />} />
```

### Render Props

Компонент получает функцию через props и вызывает её для рендеринга:

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove} style={{ height: '300px' }}>
      {render(position)}
    </div>
  );
}

// Использование
<MouseTracker
  render={({ x, y }) => (
    <p>Курсор: {x}, {y}</p>
  )}
/>
```

Сегодня render props часто заменяют кастомными хуками — они проще в использовании и не создают лишних уровней в дереве компонентов.

### Custom Hooks

Кастомный хук — это функция, начинающаяся с `use`, которая использует другие хуки внутри:

```jsx
// Хук для работы с локальным хранилищем
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Ошибка записи в localStorage:', error);
    }
  };

  return [storedValue, setValue];
}

// Использование как обычного useState, но с персистентностью
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Тема: {theme}
    </button>
  );
}
```

---

## Context API

Context решает проблему **prop drilling** — когда приходится прокидывать props через несколько уровней компонентов, которые сами эти данные не используют.

**Создание и использование контекста:**

```jsx
// 1. Создаём контекст
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light'); // значение по умолчанию

// 2. Оборачиваем дерево в Provider
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  );
}

// 3. Читаем в любом вложенном компоненте через useContext
function ThemeToggle() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Сейчас: {theme}
    </button>
  );
}
```

**Типичный паттерн** — выносить контекст и Provider в отдельный файл:

```jsx
// contexts/AuthContext.jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (credentials) => {
    const userData = await api.login(credentials);
    setUser(userData);
  };

  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Кастомный хук для удобства
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth должен использоваться внутри AuthProvider');
  }
  return context;
}
```

**Когда НЕ использовать Context**: если данные нужны только на 1-2 уровня вниз — просто передайте через props. Context — не замена всем props, а инструмент для поистине «глобальных» для поддерева данных: тема, авторизация, локаль, роутер.

---

## Рефы: useRef и forwardRef

Реф — это «лазейка» из React в реальный DOM или способ хранить значение без вызова ре-рендера.

**Доступ к DOM-элементу:**

```jsx
function SearchInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    // После монтирования inputRef.current — это <input> элемент
    inputRef.current.focus();
    inputRef.current.select(); // выделить текст
  };

  return (
    <div>
      <input ref={inputRef} type="search" placeholder="Поиск..." />
      <button onClick={focusInput}>Поставить фокус</button>
    </div>
  );
}
```

**Хранение значения без ре-рендера:**

```jsx
function Timer() {
  const [isRunning, setIsRunning] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const intervalRef = useRef(null); // ID таймера не нужен для рендеринга

  const start = () => {
    setIsRunning(true);
    intervalRef.current = setInterval(() => {
      setElapsed(prev => prev + 100);
    }, 100);
  };

  const stop = () => {
    setIsRunning(false);
    clearInterval(intervalRef.current);
  };

  return (
    <div>
      <p>{(elapsed / 1000).toFixed(1)}с</p>
      <button onClick={isRunning ? stop : start}>
        {isRunning ? 'Стоп' : 'Старт'}
      </button>
    </div>
  );
}
```

**`React.forwardRef`** нужен, когда родительский компонент хочет получить доступ к DOM-узлу внутри дочернего:

```jsx
// Дочерний компонент "пробрасывает" ref к внутреннему input
const FancyInput = React.forwardRef(function FancyInput({ placeholder, ...props }, ref) {
  return (
    <div className="input-wrapper">
      <input
        ref={ref}  // ref попадёт на этот <input>
        className="fancy-input"
        placeholder={placeholder}
        {...props}
      />
    </div>
  );
});

// Родитель может управлять фокусом
function Form() {
  const emailRef = useRef(null);

  const handleSubmitError = () => {
    emailRef.current.focus(); // фокус на поле с ошибкой
  };

  return (
    <form>
      <FancyInput ref={emailRef} placeholder="Email" type="email" />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

**Когда использовать рефы:**
- Управление фокусом, выделением текста, воспроизведением медиа
- Запуск анимаций через императивные API
- Хранение ID таймеров/интервалов
- Интеграция со сторонними DOM-библиотеками

**Когда НЕ использовать рефы:** если значение нужно для рендеринга — используйте `useState`. Рефы — это escape hatch, обходной путь из декларативного мира React в императивный мир DOM.

---

## Источники

- [Your First Component — react.dev](https://react.dev/learn/your-first-component)
- [Passing Props to a Component — react.dev](https://react.dev/learn/passing-props-to-a-component)
- [State: A Component's Memory — react.dev](https://react.dev/learn/state-a-components-memory)
- [Synchronizing with Effects — react.dev](https://react.dev/learn/synchronizing-with-effects)
- [Passing Data Deeply with Context — react.dev](https://react.dev/learn/passing-data-deeply-with-context)
- [useRef Reference — react.dev](https://react.dev/reference/react/useRef)
