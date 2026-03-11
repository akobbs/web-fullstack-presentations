# **Лабораторна робота №4**

**Тема:** Багатосторінковий React-застосунок з маршрутизацією та управлінням станом

**Мета:** Побудувати багатосторінковий SPA «Task Manager» з використанням React Router v7 для навігації між сторінками та одного з підходів до глобального управління станом (useState, Context API або Zustand). Закріпити навички налаштування маршрутизації, роботи з хуками `useParams` і `useNavigate`, а також організації потоку даних у React-застосунку.

**Обладнання:** IBM-сумісні ПК

**Програмне забезпечення:** ОС Windows 10/11, macOS або Linux; веб-браузер (Chrome, Firefox, Edge) з розширенням React Developer Tools; обліковий запис GitHub; Git; Node.js (v18+); VS Code

---

## **Контрольні запитання**

1. Яка різниця між клієнтською та серверною навігацією? Чому SPA не перезавантажує сторінку при переході між маршрутами?
2. Для чого використовується компонент `<Outlet>`? Як він пов'язаний з вкладеними маршрутами?
3. Що таке prop drilling і як Context API вирішує цю проблему?
4. Яка різниця між `useReducer` та `useState`? Коли доцільно обирати `useReducer`?
5. Що таке Zustand store і чим він відрізняється від Context API за принципом роботи?

---

## **Теоретичні відомості**

### **React Router v7: ключові концепції**

**React Router** — бібліотека маршрутизації, що реалізує клієнтську навігацію у SPA без перезавантаження сторінки. Навігація реалізована через History API браузера.

Базова структура маршрутів:

```tsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Navigate to="/tasks" replace />} />
          <Route path="tasks" element={<TasksPage />} />
          <Route path="tasks/new" element={<NewTaskPage />} />
          <Route path="tasks/:id" element={<TaskDetailPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

`<Route index>` з `<Navigate>` — перенаправляє кореневий шлях `/` на `/tasks`, щоб користувач одразу бачив список задач.

`<Outlet />` — спеціальний компонент, що вставляється у Layout і позначає місце, де рендерується активний дочірній маршрут.

**Ключові хуки React Router:**

| Хук             | Призначення         | Приклад                      |
| --------------- | ------------------- | ---------------------------- |
| `useNavigate()` | Програмна навігація | `navigate("/tasks")`         |
| `useParams()`   | Параметри з URL     | `const { id } = useParams()` |

**`NavLink`** на відміну від `Link` передає у `className` функцію з об'єктом `{ isActive }`, що дозволяє підсвічувати активний пункт меню:

```tsx
<NavLink
  to="/tasks"
  end
  className={({ isActive }) =>
    isActive ? `${styles.link} ${styles.active}` : styles.link
  }
>
  Всі задачі
</NavLink>
```

> **Атрибут `end`:** без нього `NavLink to="/tasks"` залишатиметься активним навіть на `/tasks/new` або `/tasks/123`, бо ці шляхи теж починаються з `/tasks`. Атрибут `end` вмикає точне співпадіння.
>
> **Примітка:** React Router v6+ використовує алгоритм ранжування маршрутів — статичні сегменти (`new`) завжди мають вищий пріоритет за динамічні (`:id`), тому порядок оголошення не впливає на результат. Проте для читабельності рекомендується розміщувати `/tasks/new` перед `/tasks/:id`.

### **Управління глобальним станом: три підходи**

У цій лабораторній ви самостійно обираєте підхід до зберігання списку задач. Усі три варіанти є рівноцінними з точки зору оцінювання.

#### **Варіант А: підняття стану (useState)**

Зберігати масив задач у `App` та передавати до сторінок через props і callback-функції.

```tsx
// App.tsx — стан та обробники визначаються тут
const [tasks, setTasks] = useState<Task[]>(INITIAL_TASKS);

const addTask = (task: Task) => setTasks((prev) => [...prev, task]);
const deleteTask = (id: string) =>
  setTasks((prev) => prev.filter((t) => t.id !== id));
const updateTask = (updated: Task) =>
  setTasks((prev) => prev.map((t) => (t.id === updated.id ? updated : t)));

// Передаються через props до кожної сторінки:
<Route
  path="tasks"
  element={<TasksPage tasks={tasks} onDelete={deleteTask} />}
/>;
```

#### **Варіант Б: Context API + useReducer**

Стан зберігається у контексті — будь-який компонент підписується через `useContext` без prop drilling.

Типовий патерн для файлу `store/TasksContext.tsx`:

1. Оголосити union-тип `Action` (ADD / DELETE / UPDATE)
2. Написати чисту функцію `tasksReducer(state, action)`
3. Створити контекст через `createContext`
4. Обгорнути `useReducer` у компонент-провайдер `TasksProvider`
5. Експортувати кастомний хук `useTasks()` з перевіркою на наявність контексту

```tsx
export function useTasks() {
  const ctx = useContext(TasksContext);
  if (!ctx) throw new Error("useTasks must be used within TasksProvider");
  return ctx;
}

// Використання у будь-якому компоненті:
const { tasks, dispatch } = useTasks();
dispatch({ type: "DELETE", payload: id });
```

#### **Варіант В: Zustand**

Zustand — мінімалістична бібліотека. Store оголошується як хук через `create<T>()` та доступний у будь-якому компоненті без Provider.

```ts
// store/useTasksStore.ts
interface TasksStore {
  tasks: Task[];
  addTask: (task: Task) => void;
  deleteTask: (id: string) => void;
  updateTask: (task: Task) => void;
}

export const useTasksStore = create<TasksStore>((set) => ({
  tasks: INITIAL_TASKS,
  // ... реалізуйте методи через set(state => ({ tasks: ... }))
}));

// Використання у будь-якому компоненті:
const { tasks, deleteTask } = useTasksStore();
```

---

## **Хід роботи**

### **Етап 1: Підготовка проєкту**

1.1. Якщо у вас немає репозиторію для курсу — створіть новий публічний репозиторій на GitHub з назвою `web-programming-labs`. Якщо репозиторій вже існує — використовуйте його.

1.2. Клонуйте репозиторій або перейдіть у локальну папку:

```bash
git clone https://github.com/[ваш-username]/web-programming-labs.git
cd web-programming-labs
```

1.3. Створіть новий React + TypeScript проєкт у папці `lab4`:

```bash
npm create vite@latest lab4 -- --template react-ts
cd lab4
npm install
```

1.4. Встановіть залежності для маршрутизації:

```bash
npm install react-router
```

1.5. Якщо ви обрали **Варіант В (Zustand)** — додатково встановіть:

```bash
npm install zustand
```

1.6. Видаліть шаблонний вміст `src/App.tsx` та очистіть `src/App.css`.

1.7. Замініть вміст `src/index.css` базовим скиданням стилів (дефолтний файл Vite обмежує ширину `#root` та додає зайві відступи):

```css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family:
    system-ui,
    -apple-system,
    "Segoe UI",
    Roboto,
    Helvetica,
    Arial,
    sans-serif;
  -webkit-font-smoothing: antialiased;
  color: #1e293b;
  background: #f8fafc;
}
```

1.8. Запустіть dev-сервер і переконайтесь, що застосунок відкривається в браузері:

```bash
npm run dev
```

1.9. Зафіксуйте початковий стан:

```bash
git add lab4/
git commit -m "Lab 4: Initial Vite + React + TypeScript setup"
git push origin main
```

**Результат:** Порожній React-застосунок запущений на `http://localhost:5173`.

---

### **Етап 2: Типи, початкові дані та структура проєкту**

2.1. Створіть таку структуру папок:

```text
lab4/src/
├── components/
│   ├── Layout/
│   │   ├── Layout.tsx
│   │   └── Layout.module.css
│   └── TaskCard/
│       ├── TaskCard.tsx
│       └── TaskCard.module.css
├── pages/
│   ├── TasksPage/
│   │   └── TasksPage.tsx
│   ├── TaskDetailPage/
│   │   ├── TaskDetailPage.tsx
│   │   └── TaskDetailPage.module.css
│   └── NewTaskPage/
│       ├── NewTaskPage.tsx
│       └── NewTaskPage.module.css
├── store/              ← для Варіантів Б та В
├── types/
│   └── task.ts
├── data/
│   └── initialTasks.ts
├── App.tsx
└── main.tsx
```

2.2. Скопіюйте файл `src/types/task.ts`:

```typescript
export type TaskStatus = "todo" | "in-progress" | "done";
export type TaskPriority = "low" | "medium" | "high";

export interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  priority: TaskPriority;
  createdAt: Date;
}
```

2.3. Створіть файл `src/data/initialTasks.ts` з початковими задачами. Кількість задач та їх зміст визначаються **варіантом студента**:

- **Варіант 1–10:** мінімум 4 задачі, тематика — розробка програмного забезпечення
- **Варіант 11–20:** мінімум 4 задачі, тематика — веб-дизайн та UI/UX
- **Варіант 21–30:** мінімум 4 задачі, тематика — тестування та QA

Масив повинен мати тип `Task[]`, бути іменованим експортом `INITIAL_TASKS`, а задачі — різнитися за `status` та `priority`.

> **Підказка щодо типу `createdAt`:** поле `createdAt` має тип `Date`, тому значення необхідно передавати як `new Date("2025-03-01")`, а не як рядок `"2025-03-01"`. Рядок спричинить runtime-помилку при виклику `.toLocaleDateString()`.

2.4. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add project structure, types and initial data"
```

**Результат:** Визначено типи, початкові дані та структуру проєкту.

---

### **Етап 3: Налаштування стану**

Реалізуйте **один** з варіантів управління станом (на ваш вибір). Орієнтуйтесь на теоретичні відомості вище.

#### **3-А: useState**

Перейдіть одразу до Етапу 4 — стан буде визначено в `App.tsx`.

#### **3-Б: Context API + useReducer**

Створіть файл `src/store/TasksContext.tsx` та реалізуйте повний патерн: тип `Action`, функцію `tasksReducer`, контекст, провайдер `TasksProvider` та хук `useTasks()`.

> **Підказка:** початкове значення стану для `useReducer` — це `INITIAL_TASKS` з `data/initialTasks.ts`.

#### **3-В: Zustand**

Створіть файл `src/store/useTasksStore.ts` та реалізуйте store з чотирма полями: `tasks`, `addTask`, `deleteTask`, `updateTask`. Кожен метод оновлює масив іммутабельно через `set`.

> **Підказка:** `set` у Zustand приймає функцію `(state) => ({ ...зміни })`. Використовуйте `filter`, `map` та spread — не мутуйте масив напряму.

3.1. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add state management (Variant A/B/C)"
```

**Результат:** Реалізовано обраний підхід до управління станом.

---

### **Етап 4: Layout та маршрутизація**

4.1. Скопіюйте стилі `src/components/Layout/Layout.module.css`:

```css
.layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f8fafc;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 2rem;
  height: 60px;
  background: #1e293b;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
}

.logo {
  font-size: 1.1rem;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: 0.01em;
}

.nav {
  display: flex;
  gap: 0.5rem;
}

.link {
  padding: 0.4rem 1rem;
  border-radius: 6px;
  font-size: 0.9rem;
  color: #94a3b8;
  text-decoration: none;
  transition:
    background 0.15s,
    color 0.15s;
}

.link:hover {
  background: #334155;
  color: #f1f5f9;
}

.active {
  background: #3b82f6;
  color: #fff;
}

.active:hover {
  background: #2563eb;
  color: #fff;
}

.main {
  flex: 1;
  padding: 2rem 1rem;
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
}

@media (min-width: 640px) {
  .main {
    padding: 2rem;
  }
}
```

4.2. Скопіюйте шаблон `src/components/Layout/Layout.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import { NavLink, Outlet } from "react-router";
import styles from "./Layout.module.css";

export default function Layout() {
  return (
    <div className={styles.layout}>
      <header className={styles.header}>
        <span className={styles.logo}>📋 Task Manager</span>
        <nav className={styles.nav}>
          {/* TODO: додайте NavLink на /tasks з текстом "Всі задачі".
              Активний стан: className={({ isActive }) =>
                isActive ? `${styles.link} ${styles.active}` : styles.link
              }
              Важливо: додайте атрибут end, щоб посилання не залишалось
              активним при переході на /tasks/new або /tasks/:id           */}

          {/* TODO: додайте NavLink на /tasks/new з текстом "+ Нова задача"
              з аналогічним className для активного стану.
              Атрибут end тут не потрібен.                                  */}
        </nav>
      </header>
      <main className={styles.main}>{/* TODO: відрендеріть <Outlet /> */}</main>
    </div>
  );
}
```

4.3. Скопіюйте шаблон `src/App.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router";
import Layout from "./components/Layout/Layout";
import TasksPage from "./pages/TasksPage/TasksPage";
import TaskDetailPage from "./pages/TaskDetailPage/TaskDetailPage";
import NewTaskPage from "./pages/NewTaskPage/NewTaskPage";
// TODO (Варіант А): імпортуйте useState, INITIAL_TASKS та тип Task
// TODO (Варіант Б): імпортуйте TasksProvider

export default function App() {
  // TODO (Варіант А): оголосіть tasks через useState<Task[]>(INITIAL_TASKS)
  //                   та визначте addTask, deleteTask, updateTask

  // TODO (Варіант Б): обгорніть повернений JSX у <TasksProvider>...</TasksProvider>
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Navigate to="/tasks" replace />} />
          {/* TODO: оголосіть маршрути:
              - <Route path="tasks" element={...} />   — список задач
              - <Route path="tasks/new" element={...} /> — нова задача
              - <Route path="tasks/:id" element={...} /> — деталі задачі

              Варіант А: передайте необхідні props у кожну сторінку:
                <TasksPage tasks={tasks} onDelete={deleteTask} />
                <NewTaskPage onAdd={addTask} />
                <TaskDetailPage tasks={tasks} onUpdate={updateTask} onDelete={deleteTask} />
              Варіанти Б і В: props не потрібні — стан беруть через хук. */}
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

4.4. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add Layout component and routing configuration"
```

**Результат:** Навігація між маршрутами працює, активне посилання підсвічується.

---

### **Етап 5: Компонент TaskCard та сторінка списку**

5.1. Скопіюйте стилі `src/components/TaskCard/TaskCard.module.css`:

```css
.card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-left: 4px solid #cbd5e1;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  transition: box-shadow 0.15s;
}

.card:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.low {
  border-left-color: #22c55e;
}
.medium {
  border-left-color: #f59e0b;
}
.high {
  border-left-color: #ef4444;
}

.header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 0.5rem;
}

.title {
  font-size: 1rem;
  font-weight: 600;
  color: #1e293b;
  margin: 0;
}

.badge {
  font-size: 0.75rem;
  white-space: nowrap;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  background: #f1f5f9;
  color: #64748b;
}

.description {
  font-size: 0.875rem;
  color: #64748b;
  margin: 0;
  line-height: 1.5;
}

.footer {
  display: flex;
  gap: 1rem;
  font-size: 0.8rem;
  color: #94a3b8;
}

.actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.25rem;
}

.detailBtn {
  flex: 1;
  padding: 0.4rem 0.75rem;
  background: #3b82f6;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.85rem;
}

.detailBtn:hover {
  background: #2563eb;
}

.deleteBtn {
  padding: 0.4rem 0.75rem;
  background: #fee2e2;
  color: #dc2626;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.85rem;
}

.deleteBtn:hover {
  background: #fecaca;
}
```

5.2. Скопіюйте шаблон `src/components/TaskCard/TaskCard.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import { useNavigate } from "react-router";
import type { Task, TaskPriority, TaskStatus } from "../../types/task";
import styles from "./TaskCard.module.css";

const PRIORITY_LABELS: Record<TaskPriority, string> = {
  low: "🟢 Низький",
  medium: "🟡 Середній",
  high: "🔴 Високий",
};

const STATUS_LABELS: Record<TaskStatus, string> = {
  todo: "📌 Очікує",
  "in-progress": "⚙️ В роботі",
  done: "✅ Виконано",
};

interface TaskCardProps {
  task: Task;
  onDelete: (id: string) => void;
}

export default function TaskCard({ task, onDelete }: TaskCardProps) {
  // TODO: отримайте функцію navigate через хук useNavigate()

  return (
    <div className={`${styles.card} ${styles[task.priority]}`}>
      <div className={styles.header}>
        <h3 className={styles.title}>{task.title}</h3>
        <span className={styles.badge}>{PRIORITY_LABELS[task.priority]}</span>
      </div>

      {task.description && (
        <p className={styles.description}>{task.description}</p>
      )}

      <div className={styles.footer}>
        <span>{STATUS_LABELS[task.status]}</span>
        <span>{task.createdAt.toLocaleDateString("uk-UA")}</span>
      </div>

      <div className={styles.actions}>
        <button
          className={styles.detailBtn}
          onClick={() => {
            // TODO: перейдіть на /tasks/${task.id} через navigate
          }}
        >
          Деталі →
        </button>
        <button
          className={styles.deleteBtn}
          onClick={() => {
            // TODO: викличте onDelete з task.id
          }}
        >
          🗑️ Видалити
        </button>
      </div>
    </div>
  );
}
```

5.3. Скопіюйте шаблон `src/pages/TasksPage/TasksPage.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import TaskCard from "../../components/TaskCard/TaskCard";
// TODO (Варіант А): імпортуйте тип Task для оголошення інтерфейсу TasksPageProps
// TODO (Варіанти Б і В): імпортуйте та використайте хук стану

// TODO (Варіант А): оголосіть інтерфейс TasksPageProps: { tasks: Task[]; onDelete: (id: string) => void }
//                   та додайте деструктуризацію props у сигнатуру функції

export default function TasksPage(/* TODO (Варіант А): { tasks, onDelete } */) {
  // TODO (Варіант Б): const { tasks, dispatch } = useTasks()
  //                   для видалення: dispatch({ type: "DELETE", payload: id })
  // TODO (Варіант В): const { tasks, deleteTask } = useTasksStore()

  return (
    <div>
      <h2>📋 Задачі ({/* TODO: відобразіть tasks.length */})</h2>

      {/* TODO: якщо tasks.length === 0, відобразіть:
               <p>Задач поки немає. Створіть першу!</p>            */}

      <div
        style={{
          display: "flex",
          flexDirection: "column",
          gap: "0.75rem",
          marginTop: "1rem",
        }}
      >
        {/* TODO: відрендеріть tasks.map(...) — для кожної задачі <TaskCard>
                  з атрибутами key={task.id} та task={task}
                  Варіант А: onDelete={onDelete}
                  Варіант В: onDelete={deleteTask}
                  Варіант Б: onDelete={(id) => dispatch({ type: "DELETE", payload: id })}  */}
      </div>
    </div>
  );
}
```

5.4. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add TaskCard component and TasksPage"
```

**Результат:** Сторінка `/tasks` відображає список задач.

---

### **Етап 6: Сторінка деталей задачі**

6.1. Скопіюйте стилі `src/pages/TaskDetailPage/TaskDetailPage.module.css`:

```css
.container {
  max-width: 600px;
}

.back {
  display: inline-block;
  margin-bottom: 1.5rem;
  color: #64748b;
  text-decoration: none;
  font-size: 0.9rem;
}

.back:hover {
  color: #3b82f6;
}

.card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.5rem 2rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.title {
  font-size: 1.4rem;
  font-weight: 700;
  color: #1e293b;
  margin: 0;
}

.description {
  color: #64748b;
  line-height: 1.6;
  margin: 0;
}

.meta {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  font-size: 0.9rem;
  color: #475569;
}

.meta label {
  font-weight: 600;
  color: #1e293b;
}

.select {
  margin-left: 0.5rem;
  padding: 0.25rem 0.5rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.9rem;
  cursor: pointer;
}

.deleteBtn {
  padding: 0.5rem 1.25rem;
  background: #ef4444;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  width: fit-content;
}

.deleteBtn:hover {
  background: #dc2626;
}

.notFound {
  text-align: center;
  padding: 3rem;
  color: #94a3b8;
}
```

6.2. Скопіюйте шаблон `src/pages/TaskDetailPage/TaskDetailPage.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import { Link, useNavigate, useParams } from "react-router";
import type { Task, TaskStatus } from "../../types/task";
import styles from "./TaskDetailPage.module.css";

// TODO (Варіант А): оголосіть інтерфейс props та додайте їх до сигнатури:
//   { tasks: Task[]; onUpdate: (task: Task) => void; onDelete: (id: string) => void }
// TODO (Варіант Б): імпортуйте useTasks з TasksContext
// TODO (Варіант В): імпортуйте useTasksStore

export default function TaskDetailPage(/* TODO (Варіант А): { tasks, onUpdate, onDelete } */) {
  // TODO: отримайте id з URL — const { id } = useParams<{ id: string }>()
  // TODO: отримайте navigate через useNavigate()
  // TODO (Варіант А): tasks, onUpdate, onDelete — з props
  // TODO (Варіант Б): const { tasks, dispatch } = useTasks()
  // TODO (Варіант В): const { tasks, updateTask, deleteTask } = useTasksStore()

  // TODO: знайдіть задачу — const task = tasks.find(t => t.id === id)
  const task: Task | undefined = undefined; // Task використовується лише для цієї анотації — замініть весь рядок на tasks.find(...)

  if (!task) {
    return (
      <div className={styles.notFound}>
        <p>❌ Задачу не знайдено.</p>
        <Link to="/tasks">← Назад до списку</Link>
      </div>
    );
  }

  const handleStatusChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    // TODO (Варіант А): викличте onUpdate({ ...task, status: e.target.value as TaskStatus })
    // TODO (Варіант В): викличте updateTask({ ...task, status: e.target.value as TaskStatus })
    // TODO (Варіант Б): dispatch({ type: "UPDATE", payload: { ...task, status: e.target.value as TaskStatus } })
  };

  const handleDelete = () => {
    // TODO (Варіант А): викличте onDelete(task.id)
    // TODO (Варіант В): викличте deleteTask(task.id)
    // TODO (Варіант Б): dispatch({ type: "DELETE", payload: task.id })
    // TODO: перейдіть на /tasks через navigate
  };

  return (
    <div className={styles.container}>
      <Link to="/tasks" className={styles.back}>
        ← Назад до списку
      </Link>

      <div className={styles.card}>
        <h2 className={styles.title}>{task.title}</h2>

        {task.description && (
          <p className={styles.description}>{task.description}</p>
        )}

        <div className={styles.meta}>
          <div>
            <label>Пріоритет: </label>
            {task.priority === "high"
              ? "🔴 Високий"
              : task.priority === "medium"
                ? "🟡 Середній"
                : "🟢 Низький"}
          </div>
          <div>
            <label>Статус: </label>
            <select
              className={styles.select}
              value={task.status}
              onChange={handleStatusChange}
            >
              <option value="todo">📌 Очікує</option>
              <option value="in-progress">⚙️ В роботі</option>
              <option value="done">✅ Виконано</option>
            </select>
          </div>
          <div>
            <label>Створено: </label>
            {task.createdAt.toLocaleDateString("uk-UA")}
          </div>
        </div>

        <button className={styles.deleteBtn} onClick={handleDelete}>
          🗑️ Видалити задачу
        </button>
      </div>
    </div>
  );
}
```

6.3. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add TaskDetailPage with status update and delete"
```

**Результат:** Сторінка `/tasks/:id` відображає деталі задачі та дозволяє змінювати статус і видаляти задачу.

---

### **Етап 7: Сторінка створення задачі**

7.1. Скопіюйте стилі `src/pages/NewTaskPage/NewTaskPage.module.css`:

```css
.container {
  max-width: 520px;
}

.title {
  font-size: 1.4rem;
  font-weight: 700;
  color: #1e293b;
  margin: 0 0 1.5rem;
}

.form {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.5rem 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.field label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #374151;
}

.field input,
.field textarea,
.field select {
  padding: 0.5rem 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.9rem;
  font-family: inherit;
  transition: border-color 0.15s;
}

.field input:focus,
.field textarea:focus,
.field select:focus {
  outline: none;
  border-color: #3b82f6;
}

.field textarea {
  resize: vertical;
  min-height: 80px;
}

.error {
  font-size: 0.8rem;
  color: #ef4444;
}

.actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 0.25rem;
}

.submitBtn {
  flex: 1;
  padding: 0.55rem 1rem;
  background: #3b82f6;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 600;
}

.submitBtn:hover {
  background: #2563eb;
}

.cancelBtn {
  padding: 0.55rem 1rem;
  background: #f1f5f9;
  color: #475569;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
}

.cancelBtn:hover {
  background: #e2e8f0;
}
```

7.2. Скопіюйте шаблон `src/pages/NewTaskPage/NewTaskPage.tsx` та заповніть місця, позначені коментарями `// TODO`:

```tsx
import { useState } from "react";
import { useNavigate } from "react-router";
import type { Task, TaskPriority } from "../../types/task";
import styles from "./NewTaskPage.module.css";

// TODO (Варіант А): оголосіть інтерфейс props: { onAdd: (task: Task) => void }
//                   та додайте їх до сигнатури функції
// TODO (Варіант Б): імпортуйте useTasks з TasksContext
// TODO (Варіант В): імпортуйте useTasksStore

export default function NewTaskPage(/* TODO (Варіант А): { onAdd } */) {
  // TODO: отримайте navigate через useNavigate()
  // TODO (Варіант А): addTask — це onAdd з props
  // TODO (Варіант Б): const { dispatch } = useTasks(); виклик: dispatch({ type: "ADD", payload: newTask })
  // TODO (Варіант В): const { addTask } = useTasksStore()

  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [priority, setPriority] = useState<TaskPriority>("medium");
  const [error, setError] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    if (title.trim().length < 3) {
      setError("Назва має містити щонайменше 3 символи");
      return;
    }

    const newTask: Task = {
      id: crypto.randomUUID(),
      title: title.trim(),
      description: description.trim(),
      status: "todo",
      priority,
      createdAt: new Date(),
    };

    // TODO (Варіант А): викличте onAdd(newTask)
    // TODO (Варіант В): викличте addTask(newTask)
    // TODO (Варіант Б): dispatch({ type: "ADD", payload: newTask })
    // TODO: перейдіть на /tasks через navigate
  };

  return (
    <div className={styles.container}>
      <h2 className={styles.title}>📝 Нова задача</h2>

      <form className={styles.form} onSubmit={handleSubmit}>
        <div className={styles.field}>
          <label htmlFor="title">Назва *</label>
          <input
            id="title"
            type="text"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            placeholder="Введіть назву задачі"
          />
          {error && <span className={styles.error}>{error}</span>}
        </div>

        <div className={styles.field}>
          <label htmlFor="description">Опис</label>
          <textarea
            id="description"
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            placeholder="Додатковий опис (необов'язково)"
          />
        </div>

        <div className={styles.field}>
          <label htmlFor="priority">Пріоритет</label>
          <select
            id="priority"
            value={priority}
            onChange={(e) => setPriority(e.target.value as TaskPriority)}
          >
            <option value="low">🟢 Низький</option>
            <option value="medium">🟡 Середній</option>
            <option value="high">🔴 Високий</option>
          </select>
        </div>

        <div className={styles.actions}>
          <button type="submit" className={styles.submitBtn}>
            ✅ Створити задачу
          </button>
          <button
            type="button"
            className={styles.cancelBtn}
            onClick={() => {
              // TODO: перейдіть на /tasks через navigate
            }}
          >
            Скасувати
          </button>
        </div>
      </form>
    </div>
  );
}
```

7.3. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 4: Add NewTaskPage with task creation form"
```

**Результат:** Форма на `/tasks/new` успішно створює задачу та перенаправляє на список.

---

### **Етап 8: Публікація на GitHub**

8.1. Переконайтесь, що застосунок повністю функціональний: список задач відображається, задачі можна переглядати, змінювати статус, видаляти та створювати нові.

8.2. Перевірте історію комітів:

```bash
git log --oneline
```

Очікувана структура:

```text
xxxxxxx Lab 4: Add NewTaskPage with task creation form
xxxxxxx Lab 4: Add TaskDetailPage with status update and delete
xxxxxxx Lab 4: Add TaskCard component and TasksPage
xxxxxxx Lab 4: Add Layout component and routing configuration
xxxxxxx Lab 4: Add state management (Variant A/B/C)
xxxxxxx Lab 4: Add project structure, types and initial data
xxxxxxx Lab 4: Initial Vite + React + TypeScript setup
```

8.3. Відправте всі зміни на GitHub:

```bash
git push origin main
```

**Результат:** Репозиторій містить повністю функціональний застосунок з описовою історією комітів.

---

## **Додаткове завдання (за бажанням)**

Реалізуйте фільтрацію задач на сторінці `/tasks` за статусом або пріоритетом. Стан фільтра зберігайте у компоненті `TasksPage` через `useState`. Фільтрацію виконуйте при рендері — без модифікації вихідного масиву задач у сторі.

---

## **Зміст звіту**

1. **Титульна сторінка** з темою, метою роботи, обладнанням, програмним забезпеченням

2. **Відповіді на контрольні запитання** (коротко, по суті)

3. **Виконання завдань:**

   3.1. Скріншот структури репозиторію на GitHub (папка `lab4` з файлами)

   3.2. Скріншот сторінки `/tasks` з відображеним списком задач

   3.3. Скріншот сторінки `/tasks/:id` для однієї з задач

   3.4. Скріншот сторінки `/tasks/new` з заповненою формою

   3.5. Скріншот сторінки `/tasks` після додавання нової задачі

   3.6. Скріншот обраної реалізації стану (файл store або відповідна частина `App.tsx`)

   3.7. Скріншот історії комітів (вкладка Commits на GitHub)

4. **Обґрунтування вибору** підходу до управління станом (2–3 речення)

5. **Посилання** на ваш GitHub репозиторій

---

## **Додаткові матеріали**

- [React Router — офіційна документація](https://reactrouter.com/home)
- [Zustand — офіційна документація](https://zustand.docs.pmnd.rs/)
- [React — useContext](https://react.dev/reference/react/useContext)
- [React — useReducer](https://react.dev/reference/react/useReducer)
