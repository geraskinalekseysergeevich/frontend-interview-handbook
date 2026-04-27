# Инструменты сборки и качества кода

Современное frontend-приложение — это не просто HTML, CSS и JS. Это TypeScript, JSX, SCSS, импорты модулей, оптимизированные бандлы, автоматические проверки кода. Всё это возможно благодаря экосистеме инструментов: Webpack, Babel, ESLint. Понимание их роли помогает не только настраивать проекты, но и быстро разбираться в чужих конфигурациях.

---

## Webpack: сборщик модулей

Браузер не понимает `import`/`require`, не умеет нативно работать с SCSS или JSX, не знает, как оптимизировать тысячи маленьких файлов в один эффективный бандл. Webpack решает всё это.

**Webpack** — это статический сборщик модулей. Он строит граф зависимостей вашего приложения (кто кого импортирует) и упаковывает всё в один или несколько оптимизированных файлов.

### Четыре ключевых концепта

**Entry** — точка входа. С чего начать строить граф зависимостей:

```javascript
// webpack.config.js
export default {
  entry: './src/index.js',
  // Или несколько точек входа:
  entry: {
    app: './src/index.js',
    admin: './src/admin.js',
  },
};
```

**Output** — куда положить результат:

```javascript
import path from 'node:path';

export default {
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js', // хэш для cache-busting
  },
};
```

**Loaders** — обработчики файлов. Webpack по умолчанию понимает только JS и JSON. Loaders добавляют поддержку других форматов:

```javascript
export default {
  module: {
    rules: [
      // Обработка JS/JSX через Babel
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      // Обработка CSS
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'], // порядок важен! справа налево
      },
      // Обработка SCSS
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
      // Изображения
      {
        test: /\.(png|jpg|gif|svg)$/,
        type: 'asset/resource',
      },
    ],
  },
};
```

**Plugins** — расширяют возможности Webpack за пределы простой трансформации файлов:

```javascript
import HtmlWebpackPlugin from 'html-webpack-plugin';
import MiniCssExtractPlugin from 'mini-css-extract-plugin';

export default {
  plugins: [
    // Автоматически создаёт index.html и вставляет теги <script>
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
    // Выносит CSS в отдельные файлы (вместо встраивания в JS)
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
  ],
};
```

**Mode** — включает встроенные оптимизации:

```javascript
export default {
  mode: 'production',    // минификация, tree-shaking, оптимизации
  // mode: 'development', // source maps, hot reload, читаемый бандл
};
```

---

## Babel: транспиляция JavaScript

Вы хотите писать современный JavaScript (ES2022+, JSX, TypeScript), но пользователи могут сидеть на старых браузерах, которые его не поддерживают. Babel переводит ваш код в совместимую версию.

**Babel** — компилятор JavaScript. Берёт современный синтаксис и преобразует его в более старый, понятный большинству браузеров.

```javascript
// Вы пишете:
const greet = (name) => `Hello, ${name}!`;
const { a, ...rest } = obj;

// Babel превращает в:
var greet = function(name) {
  return 'Hello, ' + name + '!';
};
var a = obj.a;
var rest = Object.assign({}, obj);
// (из которого убран ключ a)
```

### Конфигурация через пресеты

Конфигурация Babel хранится в файле `babel.config.json` или `.babelrc`:

```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "> 0.5%, last 2 versions, not dead"
    }],
    "@babel/preset-react",
    "@babel/preset-typescript"
  ]
}
```

**@babel/preset-env** — главный пресет. Автоматически определяет, какие трансформации нужны, исходя из списка целевых браузеров. Не транспилирует то, что уже поддерживается.

**@babel/preset-react** — добавляет поддержку JSX-синтаксиса:
```jsx
// JSX:
const element = <div className="container">Hello</div>;

// Babel превращает в:
const element = React.createElement('div', { className: 'container' }, 'Hello');
```

**@babel/preset-typescript** — удаляет TypeScript-аннотации (не валидирует типы, это задача `tsc`):
```typescript
// TypeScript:
function add(a: number, b: number): number {
  return a + b;
}

// После Babel:
function add(a, b) {
  return a + b;
}
```

---

## ESLint: статический анализ кода

ESLint проверяет код на потенциальные ошибки и нарушения стиля ещё до запуска — без исполнения кода. Это называется статическим анализом.

### Базовая настройка

```bash
npm init @eslint/config@latest
```

Современная конфигурация использует файл `eslint.config.js`:

```javascript
import { defineConfig } from 'eslint/config';
import js from '@eslint/js';
import reactPlugin from 'eslint-plugin-react';

export default defineConfig([
  {
    files: ['**/*.{js,jsx}'],
    plugins: {
      js,
      react: reactPlugin,
    },
    extends: ['js/recommended'],
    rules: {
      // Три уровня: 'off', 'warn', 'error'
      'no-unused-vars': 'warn',        // предупреждение
      'no-console': 'warn',            // предупреждение
      'no-undef': 'error',             // ошибка — прерывает CI
      'eqeqeq': 'error',               // запрещает == в пользу ===
      'react/prop-types': 'warn',
    },
    languageOptions: {
      ecmaVersion: 2022,
      sourceType: 'module',
    },
  },
]);
```

### Запуск

```bash
# Проверить конкретный файл
npx eslint src/utils.js

# Проверить всё приложение
npx eslint src/

# Автоматически исправить что можно
npx eslint src/ --fix
```

ESLint понимает, что исправить автоматически (стиль, некоторые синтаксические паттерны), а что требует ручного вмешательства (логические ошибки).

---

## lint-staged: pre-commit хуки

Запускать ESLint на всём проекте перед каждым коммитом — долго. lint-staged запускает проверки только на файлах, которые вы собираетесь закоммитить.

В связке используются два инструмента:

**Husky** — управляет Git-хуками (скриптами, которые запускаются до/после git-операций):

```bash
npm install --save-dev husky lint-staged
npx husky init
```

**lint-staged** — запускает команды только на staged-файлах:

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

Теперь при каждом `git commit` автоматически запускается ESLint и Prettier только на изменённых файлах. Если есть ошибки — коммит прерывается.

---

## Как инструменты связаны в типичном проекте

```
Исходный код (JSX/TS/SCSS)
         │
         ▼
      [Babel]          ← транспилирует JSX, TypeScript, современный JS
         │
         ▼
      [Webpack]        ← собирает граф зависимостей, применяет loaders
         │             ← loaders вызывают babel-loader, css-loader и т.д.
         ▼
  Оптимизированный     ← минификация, tree-shaking, разбивка на чанки
  бандл в /dist
```

```
При разработке:
Код → [ESLint] → предупреждения в редакторе
    → [git commit] → [Husky] → [lint-staged] → [ESLint --fix] → коммит
    → [Jest] → тесты (CI/CD)
    → [webpack-dev-server] → hot reload в браузере
```

Каждый инструмент делает свою часть работы:
- **Babel** — трансформирует синтаксис
- **Webpack** — организует модули в бандл
- **ESLint** — следит за качеством кода
- **lint-staged + Husky** — не дают плохому коду попасть в репозиторий

---

## Источники
- [Webpack — Core Concepts](https://webpack.js.org/concepts/)
- [Babel — Documentation](https://babeljs.io/docs/)
- [ESLint — Getting Started](https://eslint.org/docs/latest/use/getting-started)
