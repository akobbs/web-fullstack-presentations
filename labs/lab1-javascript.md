# **Лабораторна робота №1**

**Тема:** JavaScript практика

**Мета:** Закріпити практичні навички роботи з сучасним синтаксисом JavaScript (ES6+): деструктуризація, spread/rest оператори, методи масивів, класи, асинхронне програмування та модулі.

**Обладнання:** IBM-сумісні ПК

**Програмне забезпечення:** ОС Windows 10/11, macOS або Linux; веб-браузер (Chrome, Firefox, Edge); обліковий запис GitHub; Git; VS Code з розширенням Live Server

## **Контрольні запитання**

1. Яка різниця між `let`, `const` та `var`? Чому `var` вважається застарілим?
2. Чим відрізняються методи масивів `map`, `filter` та `reduce`?
3. Що таке деструктуризація і як вона працює для об'єктів та масивів?
4. Яка різниця між `export default` та named export?
5. Що таке Promise і які три стани він може мати?

## **Теоретичні відомості**

### **Деструктуризація та Spread/Rest**

**Деструктуризація** — синтаксис ES6, що дозволяє "розпаковувати" значення з масивів або властивості з об'єктів у окремі змінні. **Spread** (`...`) розгортає масив або об'єкт на окремі елементи. **Rest** (`...`) збирає решту елементів у масив.

```javascript
const { name, age } = { name: "John", age: 25 };
const [first, ...rest] = [1, 2, 3, 4];
const merged = { ...obj1, ...obj2 };
```

### **Методи масивів**

Функціональні методи масивів — основа сучасного JavaScript:

- `map(fn)` — трансформує кожен елемент, повертає новий масив
- `filter(fn)` — відбирає елементи за умовою, повертає новий масив
- `reduce(fn, init)` — згортає масив до одного значення
- `find(fn)` — повертає перший елемент, що задовольняє умову
- `sort(fn)` — сортує масив (мутує оригінал!)

### **Класи ES6**

Класи надають зручний синтаксис для об'єктно-орієнтованого програмування. Підтримують конструктори, наслідування (`extends`), статичні методи (`static`), приватні поля (`#field`) та геттери/сеттери.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} видає звук`;
  }
}
class Dog extends Animal {
  speak() {
    return `${this.name} гавкає!`;
  }
}
```

### **Асинхронність: Promise та async/await**

**Promise** — об'єкт, що представляє результат асинхронної операції. Має три стани: pending (очікування), fulfilled (виконано), rejected (відхилено).

**async/await** — синтаксичний цукор для роботи з Promise:

```javascript
async function fetchData(url) {
  try {
    const response = await fetch(url);
    return await response.json();
  } catch (error) {
    console.error("Помилка:", error.message);
  }
}
```

### **Модулі ES6**

Модулі дозволяють розбивати код на логічні файли з ізольованою областю видимості:

```javascript
// math.js
export function sum(a, b) {
  return a + b;
}
export default class Calculator {
  /* ... */
}

// app.js
import Calculator, { sum } from "./math.js";
```

## **Хід роботи**

У цій лабораторній роботі ви виконаєте 5 практичних завдань, що покривають ключові теми сучасного JavaScript.

### **Етап 1: Підготовка репозиторію та структури проєкту**

1.1. Якщо у вас ще немає репозиторію для курсу, створіть новий публічний репозиторій на GitHub з назвою `web-programming-labs` (або аналогічною). Якщо репозиторій вже створено для попередніх лабораторних — використовуйте його.

1.2. Клонуйте репозиторій (або перейдіть у локальну папку, якщо він вже клонований):

```
git clone https://github.com/[ваш-username]/web-programming-labs.git
cd web-programming-labs
```

1.3. Створіть папку `lab1` з такою структурою файлів:

```
web-programming-labs/
├── lab1/
│   ├── src/
│   │   ├── config.js
│   │   ├── task-1.js
│   │   ├── task-2.js
│   │   ├── task-3.js
│   │   ├── task-4.js
│   │   └── task-5/
│   │       ├── index.js
│   │       ├── utils.js
│   │       └── data.js
│   ├── index.html
│   └── README.md
└── ...
```

1.4. Створіть файл `lab1/src/config.js` з вашим варіантом:

```javascript
// config.js — Конфігурація варіанту
export const VARIANT = 0; // Ваш номер у журналі (1-30)
```

**Важливо:** значення `VARIANT` впливає на вхідні дані у завданні 2. Кожен студент повинен використовувати свій номер з журналу.

1.5. **Встановіть розширення Live Server у VS Code** (необхідне для роботи ES6 модулів):

- Відкрийте VS Code
- Перейдіть у вкладку Extensions (Ctrl+Shift+X)
- Знайдіть **"Live Server"** (автор: Ritwick Dey)
- Натисніть **Install**
- Після встановлення ви зможете запускати HTML-файли через правий клік → **"Open with Live Server"** або кнопку **"Go Live"** у нижній панелі VS Code

**Чому це потрібно:** ES6 модулі (`import`/`export`) не працюють при відкритті HTML-файлу напряму через `file://` протокол — браузер блокує такі запити з міркувань безпеки. Live Server створює локальний веб-сервер, що вирішує цю проблему.

1.6. Створіть файл `lab1/index.html`:

```html
<!DOCTYPE html>
<html lang="uk">
  <head>
    <meta charset="UTF-8" />
    <title>ЛР1 — JavaScript практика</title>
  </head>
  <body>
    <h1>Лабораторна робота №1 — JavaScript практика</h1>
    <p>Відкрийте консоль розробника (F12) для перегляду результатів.</p>
    <script type="module">
      // Розкоментуйте потрібне завдання для перевірки:
      // import './src/task-1.js';
      // import './src/task-2.js';
      // import './src/task-3.js';
      // import './src/task-4.js';
      // import './src/task-5/index.js';
    </script>
  </body>
</html>
```

1.7. Зафіксуйте початкову структуру:

```
git add lab1/
git commit -m "Lab 1: Initial project structure"
git push origin main
```

**Результат:** Репозиторій містить папку `lab1` з підготовленою структурою проєкту.

---

### **Етап 2: Виконання завдань**

Для кожного завдання:

- Реалізуйте функції відповідно до умови
- Додайте коментарі, що пояснюють логіку **своїми словами**
- Виведіть результати через `console.log()`
- Зробіть **окремий коміт** після завершення

---

#### **Завдання 1: Деструктуризація та Spread/Rest (task-1.js)**

**1.1.** `getFullName(user)` — приймає об'єкт з полями `firstName`, `lastName`, `middleName` (опціональне). Використовуючи деструктуризацію з значенням за замовчуванням, повертає повне ім'я у форматі `"Прізвище І. Б."`. Якщо `middleName` відсутній — повертає `"Прізвище І."`.

```javascript
const user1 = {
  firstName: "Петро",
  lastName: "Іванов",
  middleName: "Сергійович",
};
getFullName(user1); // "Іванов П. С."

const user2 = { firstName: "Анна", lastName: "Коваль" };
getFullName(user2); // "Коваль А."
```

**1.2.** `mergeObjects(...objects)` — приймає довільну кількість об'єктів (rest) та повертає один об'єкт, що є результатом їх об'єднання (spread). Пізніші об'єкти перезаписують властивості попередніх.

```javascript
mergeObjects({ a: 1 }, { b: 2 }, { a: 3, c: 4 }); // { a: 3, b: 2, c: 4 }
```

**1.3.** `removeDuplicates(...arrays)` — приймає довільну кількість масивів, об'єднує їх та повертає масив без дублікатів (використайте spread та `Set`).

```javascript
removeDuplicates([1, 2, 3], [2, 3, 4], [4, 5]); // [1, 2, 3, 4, 5]
```

**1.4.** `createUpdatedUser(user, updates)` — повертає новий об'єкт користувача з оновленими полями, не мутуючи оригінал. Якщо `updates` містить вкладений об'єкт `address`, він також має бути об'єднаний, а не замінений.

```javascript
const user = { name: "John", age: 25, address: { city: "Kyiv", zip: "01001" } };
const updated = createUpdatedUser(user, { age: 26, address: { zip: "02002" } });
// updated: { name: "John", age: 26, address: { city: "Kyiv", zip: "02002" } }
// user — залишається незмінним
```

Виведіть результат кожної функції у консоль:

```javascript
console.log("=== Завдання 1: Деструктуризація та Spread/Rest ===");
console.log(
  "1.1:",
  getFullName({
    firstName: "Петро",
    lastName: "Іванов",
    middleName: "Сергійович",
  }),
);
console.log("1.2:", mergeObjects({ a: 1 }, { b: 2 }, { a: 3, c: 4 }));
// ... і так далі
```

---

#### **Завдання 2: Методи масивів (task-2.js)**

На початку файлу імпортуйте конфігурацію та створіть вхідні дані:

```javascript
import { VARIANT } from "./config.js";

const products = [
  {
    id: 1,
    name: "Ноутбук",
    price: 25000 + VARIANT * 100,
    category: "electronics",
    inStock: true,
  },
  {
    id: 2,
    name: "Навушники",
    price: 2500 + VARIANT * 10,
    category: "electronics",
    inStock: true,
  },
  {
    id: 3,
    name: "Футболка",
    price: 800 + VARIANT * 5,
    category: "clothing",
    inStock: false,
  },
  {
    id: 4,
    name: "Книга 'JavaScript'",
    price: 450 + VARIANT * 3,
    category: "books",
    inStock: true,
  },
  {
    id: 5,
    name: "Рюкзак",
    price: 1500 + VARIANT * 8,
    category: "accessories",
    inStock: true,
  },
  {
    id: 6,
    name: "Клавіатура",
    price: 3200 + VARIANT * 15,
    category: "electronics",
    inStock: false,
  },
  {
    id: 7,
    name: "Кросівки",
    price: 4200 + VARIANT * 20,
    category: "clothing",
    inStock: true,
  },
  {
    id: 8,
    name: "Книга 'TypeScript'",
    price: 520 + VARIANT * 4,
    category: "books",
    inStock: true,
  },
  {
    id: 9,
    name: "Чохол для телефону",
    price: 350 + VARIANT * 2,
    category: "accessories",
    inStock: true,
  },
  {
    id: 10,
    name: "Монітор",
    price: 12000 + VARIANT * 50,
    category: "electronics",
    inStock: true,
  },
];
```

Реалізуйте функції, використовуючи **виключно** методи масивів (`map`, `filter`, `reduce`, `sort`). Використання `for`, `while`, `for...of` — **заборонено**:

**2.1.** `getAvailableProducts(products)` — повертає масив назв товарів, що є в наявності (`inStock === true`).

**2.2.** `getProductsByCategory(products, category)` — повертає масив товарів вказаної категорії, відсортованих за ціною від дешевих до дорогих.

**2.3.** `getTotalPrice(products)` — повертає загальну суму цін усіх товарів, що є в наявності.

**2.4.** `getProductsSummary(products)` — повертає об'єкт з підсумком по категоріях:

```javascript
// Очікуваний формат результату (конкретні значення залежать від вашого variant):
{
  electronics: { count: 4, totalPrice: ... },
  clothing: { count: 2, totalPrice: ... },
  books: { count: 2, totalPrice: ... },
  accessories: { count: 2, totalPrice: ... }
}
```

Виведіть результати у консоль:

```javascript
console.log("=== Завдання 2: Методи масивів ===");
console.log("Варіант:", VARIANT);
console.log("2.1:", getAvailableProducts(products));
console.log("2.2:", getProductsByCategory(products, "electronics"));
console.log("2.3:", getTotalPrice(products));
console.log("2.4:", getProductsSummary(products));
```

---

#### **Завдання 3: Класи (task-3.js)**

Створіть ієрархію класів для медіа-бібліотеки:

**3.1.** Клас `MediaItem`:

- Приватне поле `#id` (генерується автоматично через статичний лічильник `static #nextId = 1`)
- Поля: `title`, `year`
- Getter `id` — повертає значення приватного `#id`
- Getter `age` — повертає кількість років з моменту видання (`new Date().getFullYear() - this.year`)
- Метод `getInfo()` — повертає рядок у форматі `"[ID] Назва (рік)"`
- Статичний метод `compare(a, b)` — порівнює два елементи за роком видання (для використання у `sort()`)

**3.2.** Клас `Book extends MediaItem`:

- Конструктор: `constructor(title, year, author, pages)` — викликає `super(title, year)` та зберігає додаткові поля
- Додаткові поля: `author`, `pages`
- Перевизначений метод `getInfo()` — повертає `"[ID] Назва — Автор (рік, стор. N)"`

**3.3.** Клас `Movie extends MediaItem`:

- Конструктор: `constructor(title, year, director, duration)` — викликає `super(title, year)` та зберігає додаткові поля
- Додаткові поля: `director`, `duration` (у хвилинах)
- Перевизначений метод `getInfo()` — повертає `"[ID] Назва — Режисер (рік, N хв)"`

Продемонструйте роботу:

```javascript
console.log("=== Завдання 3: Класи ===");

const book1 = new Book("Кобзар", 1840, "Тарас Шевченко", 280);
const book2 = new Book("Clean Code", 2008, "Robert Martin", 464);
const movie1 = new Movie("Тіні забутих предків", 1965, "Сергій Параджанов", 97);

console.log(book1.getInfo());
console.log(movie1.getInfo());
console.log("Вік книги:", book1.age, "років");

const items = [book1, book2, movie1];
const sorted = [...items].sort(MediaItem.compare);
console.log(
  "Відсортовано за роком:",
  sorted.map((i) => i.getInfo()),
);
```

---

#### **Завдання 4: async/await та Promises (task-4.js)**

**4.1.** `delay(ms)` — повертає Promise, що виконується через `ms` мілісекунд.

```javascript
await delay(1000); // чекає 1 секунду
```

**4.2.** `simulateFetch(url)` — симулює мережевий запит. Повертає Promise, що через 200–500мс (випадкова затримка):

- Якщо URL не починається з `"https"` — реджектиться з помилкою `"Invalid URL: ..."`
- Якщо URL починається з `"https"` — з ймовірністю 70% резолвиться з об'єктом `{ url, status: 200, data: "OK" }`, а з ймовірністю 30% реджектиться з помилкою `"Server error: 500"` (симуляція нестабільного сервера)

**4.3.** `fetchWithRetry(url, attempts)` — async-функція, що викликає `simulateFetch(url)` і у разі помилки повторює виклик до `attempts` разів. Між спробами — затримка 500мс (використайте `delay`). Виводить номер спроби у консоль. Якщо всі спроби невдалі — кидає останню помилку.

**4.4.** `fetchMultiple(urls)` — async-функція, що приймає масив URL та завантажує їх **паралельно** через `Promise.allSettled()`. Повертає об'єкт з двома масивами: `successful` (успішні результати) та `failed` (повідомлення помилок).

```javascript
const result = await fetchMultiple([
  "https://jsonplaceholder.typicode.com/posts",
  "http://invalid-url",
  "https://jsonplaceholder.typicode.com/users",
]);
// { successful: [{...}, {...}], failed: ["Invalid URL: http://invalid-url"] }
```

Обгорніть весь демонстраційний код у async-функцію та викличте її:

```javascript
async function main() {
  console.log("=== Завдання 4: async/await ===");

  // 4.1
  console.time("delay");
  await delay(1000);
  console.timeEnd("delay"); // ~1000ms

  // 4.2
  try {
    const result = await simulateFetch(
      "https://jsonplaceholder.typicode.com/posts",
    );
    console.log("Успіх:", result);
  } catch (error) {
    console.error("Помилка:", error.message);
  }

  // 4.3 — retry для нестабільного сервера
  try {
    const result = await fetchWithRetry(
      "https://jsonplaceholder.typicode.com/posts",
      5,
    );
    console.log("fetchWithRetry результат:", result);
  } catch (error) {
    console.error("Всі спроби невдалі:", error.message);
  }

  // 4.4
  const results = await fetchMultiple([
    "https://jsonplaceholder.typicode.com/posts",
    "http://invalid-url",
    "https://jsonplaceholder.typicode.com/users",
  ]);
  console.log("Результати:", results);
}

main();
```

---

#### **Завдання 5: Модулі (task-5/)**

Розбийте код на три файли з використанням ES6 модулів:

**5.1.** `data.js` — експортує:

- Константу `LIBRARY_NAME` (named export) — назва бібліотеки (рядок)
- Масив `books` (named export) — мінімум 5 книг з полями: `title`, `author`, `year`, `pages`, `genre`

**5.2.** `utils.js` — експортує:

- `getBooksByGenre(books, genre)` — фільтрація за жанром (named export)
- `getAveragePages(books)` — середня кількість сторінок (named export)
- `getOldestBook(books)` — повертає найстарішу книгу (named export)
- `default export` — клас `BookCollection`, який:
  - Приймає масив книг у конструкторі
  - Має метод `getSortedByYear()` — повертає копію масиву, відсортовану за роком (від старих до нових)
  - Має метод `addBook(book)` — додає книгу
  - Має getter `count` — повертає кількість книг

**5.3.** `index.js` — головний файл, що:

- Імпортує `LIBRARY_NAME` та `books` з `data.js`
- Імпортує `BookCollection` як default import з `utils.js`
- Імпортує named exports з `utils.js` (хоча б один — з перейменуванням через `as`)
- Демонструє роботу всіх імпортованих функцій та класу
- Виводить результати у консоль

```javascript
console.log("=== Завдання 5: Модулі ===");
console.log("Бібліотека:", LIBRARY_NAME);
console.log("Всього книг:", books.length);
// ... демонстрація всіх функцій
```

---

### **Етап 3: Публікація на GitHub**

3.1. Переконайтесь, що кожне завдання має окремий коміт:

```
git log --oneline
```

Приклад очікуваної історії комітів:

```
f7a8b9c Lab 1: Task 5 - ES6 modules
e4d5c6b Lab 1: Task 4 - async/await and Promises
c1b2a3d Lab 1: Task 3 - Classes and inheritance
b8c9d0e Lab 1: Task 2 - Array methods
a5b6c7d Lab 1: Task 1 - Destructuring and spread/rest
d2e3f4a Lab 1: Initial project structure
```

3.2. Заповніть `lab1/README.md`:

```markdown
# Лабораторна робота №1 — JavaScript практика

**Варіант:** [номер]

## Структура проєкту

- `src/task-1.js` — Деструктуризація та spread/rest
- `src/task-2.js` — Методи масивів
- `src/task-3.js` — Класи та наслідування
- `src/task-4.js` — async/await та Promises
- `src/task-5/` — ES6 модулі

## Запуск

1. Відкрийте проєкт у VS Code
2. Запустіть `index.html` через Live Server (правий клік → Open with Live Server)
3. Відкрийте консоль розробника (F12) для перегляду результатів
```

3.3. Відправте всі зміни на GitHub:

```
git push origin main
```

**Результат:** Репозиторій містить всі завдання з описовою історією комітів.

## **Зміст звіту**

1. **Титульна сторінка** з темою, метою роботи, обладнанням, програмним забезпеченням

2. **Відповіді на контрольні питання** (коротко, по суті)

3. **Виконання завдань:**

   3.1. Скріншот структури репозиторію на GitHub (папка `lab1` з файлами)

   3.2. Для кожного завдання (1–5): скріншот коду та скріншот результату виконання в консолі браузера

   3.3. Скріншот історії комітів (вкладка Commits на GitHub)

4. **Посилання** на ваш GitHub репозиторій

5. **Висновки** (що навчились робити, які навички отримали, з якими труднощами зіткнулись)

## **Типові помилки та їх вирішення**

| Проблема                                              | Рішення                                                               |
| ----------------------------------------------------- | --------------------------------------------------------------------- |
| `import` не працює у браузері                         | Переконайтесь, що `<script>` має атрибут `type="module"`              |
| `CORS error` або `Failed to fetch` при відкритті HTML | Файл відкрито через `file://` — запустіть через Live Server у VS Code |
| `Cannot use import statement outside a module`        | Файл повинен бути підключений як модуль (`type="module"`)             |
| Promise не виводить результат у `console.log`         | Promise — асинхронний; використовуйте `.then()` або `await`           |
| `#field` не працює                                    | Приватні поля підтримуються у сучасних браузерах; оновіть браузер     |

## **Додаткові матеріали**

1. Сучасний підручник JavaScript — https://uk.javascript.info/

2. JavaScript Tutorial (W3Schools, українською) — https://w3schoolsua.github.io/js/index.html

3. MDN Web Docs — JavaScript — https://developer.mozilla.org/en-US/docs/Web/JavaScript
