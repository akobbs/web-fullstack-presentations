# **Лабораторна робота №3**

**Тема:** React-застосунок

**Мета:** Створити інтерактивний односторінковий застосунок «Task Manager» з використанням React та TypeScript. Закріпити навички роботи з компонентами, props, хуком `useState`, формами з валідацією через React Hook Form та Zod, а також CSS Modules для стилізації.

**Обладнання:** IBM-сумісні ПК

**Програмне забезпечення:** ОС Windows 10/11, macOS або Linux; веб-браузер (Chrome, Firefox, Edge) з розширенням React Developer Tools; обліковий запис GitHub; Git; Node.js (v18+); VS Code

---

## **Контрольні запитання**

1. Що таке однонаправлений потік даних у React і як він реалізується через props та callback-функції?
2. Яка різниця між контрольованими (controlled) та неконтрольованими (uncontrolled) компонентами форм?
3. Що таке іммутабельне оновлення стану і чому не можна мутувати масив або об'єкт у `useState` напряму?
4. Для чого потрібен атрибут `key` при рендерингу списків? Чому `index` масиву є поганим ключем?
5. Яку роль відіграє Zod у зв'язці з React Hook Form? Які переваги типобезпечної валідації?

---

## **Теоретичні відомості**

### **Компоненти та Props**

Функціональний компонент - це TypeScript-функція, що приймає об'єкт `props` та повертає JSX. Типи props описуються через `interface`. Дані передаються лише зверху вниз (від батька до дитини), а зворотній зв'язок - через callback-функції як props.

```tsx
interface TaskCardProps {
  task: Task;
  onDelete: (id: string) => void;
}

function TaskCard({ task, onDelete }: TaskCardProps) {
  return (
    <div>
      <h3>{task.title}</h3>
      <button onClick={() => onDelete(task.id)}>Видалити</button>
    </div>
  );
}
```

### **Хук useState**

`useState` зберігає локальний стан компонента. Кожен виклик функції-сеттера викликає перерендер. Масиви та об'єкти оновлюються іммутабельно - через spread-оператор або методи, що повертають нові колекції (`map`, `filter`).

```tsx
const [tasks, setTasks] = useState<Task[]>([]);

// Додавання без мутації
setTasks((prev) => [...prev, newTask]);

// Видалення без мутації
setTasks((prev) => prev.filter((t) => t.id !== id));

// Оновлення без мутації
setTasks((prev) => prev.map((t) => (t.id === id ? { ...t, status } : t)));
```

### **React Hook Form та Zod**

**React Hook Form** - бібліотека для керування формами. Зменшує кількість перерендерів порівняно з повністю контрольованим підходом. Основні API: `useForm`, `register`, `handleSubmit`, `formState.errors`.

**Zod** - бібліотека для оголошення схем даних та їх валідації. Схема слугує одночасно і валідатором, і джерелом TypeScript-типів (`z.infer<typeof schema>`).

Інтеграція здійснюється через `@hookform/resolvers/zod`:

```tsx
import { z } from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const taskSchema = z.object({
  title: z
    .string()
    .min(3, "Мінімум 3 символи")
    .max(100, "Максимум 100 символів"),
  description: z.string().max(500, "Максимум 500 символів"),
  priority: z.enum(["low", "medium", "high"]),
});

type TaskFormData = z.infer<typeof taskSchema>;

const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm<TaskFormData>({
  resolver: zodResolver(taskSchema),
});
```

### **CSS Modules**

CSS Modules ізолюють стилі на рівні компонента - імена класів хешуються при збірці, що унеможливлює колізії між компонентами. Файл стилів підключається як модуль, а класи доступні як властивості імпортованого об'єкта.

```tsx
import styles from "./TaskCard.module.css";

<div className={styles.card}>...</div>;
```

---

## **Хід роботи**

### **Етап 1: Підготовка репозиторію та проєкту**

1.1. Якщо у вас немає репозиторію для курсу - створіть новий публічний репозиторій на GitHub з назвою `web-programming-labs`. Якщо репозиторій вже існує - використовуйте його.

1.2. Клонуйте репозиторій або перейдіть у локальну папку:

```bash
git clone https://github.com/[ваш-username]/web-programming-labs.git
cd web-programming-labs
```

1.3. Створіть новий React + TypeScript проєкт у папці `lab3`:

```bash
npm create vite@latest lab3 -- --template react-ts
cd lab3
npm install
```

1.4. Встановіть необхідні залежності:

```bash
npm install react-hook-form zod @hookform/resolvers clsx
```

1.5. Видаліть шаблонний вміст `src/App.tsx` та `src/App.css`. Очистіть `src/index.css` або залиште лише базові стилі (reset, шрифти).

1.6. Запустіть dev-сервер і переконайтесь, що застосунок завантажується:

```bash
npm run dev
```

1.7. Зафіксуйте початковий стан:

```bash
git add lab3/
git commit -m "Lab 3: Initial Vite + React + TypeScript setup"
git push origin main
```

**Результат:** Порожній React-застосунок запущений на `http://localhost:5173`.

---

### **Етап 2: Типи та структура проєкту**

2.1. Створіть папки для організації коду:

```
lab3/src/
├── components/
│   ├── TaskForm/
│   │   ├── TaskForm.tsx
│   │   └── TaskForm.module.css
│   ├── TaskCard/
│   │   ├── TaskCard.tsx
│   │   └── TaskCard.module.css
│   └── TaskList/
│       ├── TaskList.tsx
│       └── TaskList.module.css
├── types/
│   └── task.ts
├── App.tsx
├── App.module.css
└── main.tsx
```

2.2. Створіть файл `src/types/task.ts` з типами домену:

```typescript
export type TaskStatus = "todo" | "in-progress" | "done";
export type TaskPriority = "low" | "medium" | "high";

export interface Task {
  id: string;
  title: string;
  description: string; // порожній рядок означає "опис відсутній"
  status: TaskStatus;
  priority: TaskPriority;
  createdAt: Date;
}
```

2.3. Зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 3: Add project structure and Task types"
```

**Результат:** Визначено базову структуру проєкту та типи.

---

### **Етап 3: Компонент TaskCard**

3.1. Реалізуйте компонент `src/components/TaskCard/TaskCard.tsx`, який:

- Приймає props: `task: Task`, `onDelete: (id: string) => void`, `onStatusChange: (id: string, status: TaskStatus) => void`
- Відображає: заголовок, опис (якщо рядок непорожній), пріоритет, поточний статус та дату створення у форматі `дд.мм.рррр`
- Містить кнопку «Видалити», що викликає `onDelete(task.id)`
- Містить `<select>` для зміни статусу, що викликає `onStatusChange` при зміні
- Отримує різні CSS-класи залежно від пріоритету задачі (використовуйте `clsx` та CSS Modules)
- Експортується як **default export**: `export default TaskCard`

> **Підказка щодо типізації `<select>`:** значення `e.target.value` має тип `string`, тоді як `onStatusChange` очікує `TaskStatus`. Звужуйте тип явно - це безпечно, оскільки `<option>` містять лише допустимі значення:
>
> ```tsx
> onChange={(e) => onStatusChange(task.id, e.target.value as TaskStatus)}
> ```

3.2. Створіть стилі `src/components/TaskCard/TaskCard.module.css`. Мінімально необхідні класи:

- `.card` - базова картка (рамка, відступи, заокруглення)
- `.cardLow`, `.cardMedium`, `.cardHigh` - кольорова ліва рамка для пріоритетів (зелена, жовта, червона)
- `.title` - стиль заголовка
- `.meta` - рядок з датою та пріоритетом
- `.actions` - рядок із кнопкою та селектом

> **Примітка щодо імен класів у CSS Modules:** уникайте дефісів у назвах класів - звертатися до них доведеться через `styles['card-high']`. Використовуйте camelCase: `styles.cardHigh`.

3.3. Перевірте компонент, тимчасово додавши мок у `App.tsx`:

```tsx
import TaskCard from "./components/TaskCard/TaskCard";
import type { Task } from "./types/task";

const mockTask: Task = {
  id: "1",
  title: "Тестова задача",
  description: "Перевірка відображення картки",
  status: "todo",
  priority: "high",
  createdAt: new Date(),
};

function App() {
  return (
    <TaskCard
      task={mockTask}
      onDelete={(id) => console.log("delete", id)}
      onStatusChange={(id, status) => console.log("status", id, status)}
    />
  );
}

export default App;
```

Після перевірки у браузері видаліть мок - `App.tsx` буде повністю переписано в Етапі 6. Зафіксуйте:

```bash
git add .
git commit -m "Lab 3: Add TaskCard component"
```

---

### **Етап 4: Компонент TaskList**

4.1. Реалізуйте компонент `src/components/TaskList/TaskList.tsx`, який:

- Приймає props: `tasks: Task[]`, `onDelete: (id: string) => void`, `onStatusChange: (id: string, status: TaskStatus) => void`
- Якщо масив `tasks` порожній - відображає повідомлення «Задач немає. Додайте першу задачу!» замість списку
- Рендерить список карток `<TaskCard>` з коректним атрибутом `key={task.id}`
- Передає `onDelete` та `onStatusChange` у кожну `TaskCard` як props
- Експортується як **default export**: `export default TaskList`

  4.2. Додайте базові стилі у `TaskList.module.css` (відступи між картками, максимальна ширина контейнера).

  4.3. Зафіксуйте:

```bash
git add .
git commit -m "Lab 3: Add TaskList component"
```

---

### **Етап 5: Компонент TaskForm**

5.1. Реалізуйте схему валідації Zod у `src/components/TaskForm/TaskForm.tsx`:

```typescript
import { z } from "zod";

const taskSchema = z.object({
  title: z
    .string()
    .min(3, "Заголовок має містити щонайменше 3 символи")
    .max(100, "Заголовок не може перевищувати 100 символів"),
  description: z.string().max(500, "Опис не може перевищувати 500 символів"),
  priority: z.enum(["low", "medium", "high"], {
    message: "Оберіть пріоритет",
  }),
});

export type TaskFormData = z.infer<typeof taskSchema>;
```

5.2. Реалізуйте сам компонент форми з використанням `useForm` та `zodResolver`:

- Props: `onSubmit: (data: TaskFormData) => void` - `id`, `status` та `createdAt` додаються в `App.tsx` при обробці сабміту
- Поля форми: текстове поле для `title`, `<textarea>` для `description` (необов'язкове для заповнення - порожній рядок є валідним значенням), `<select>` для `priority` з опцією-заглушкою `value=""`
- Після успішного сабміту - скидайте форму через `reset()`
- Під кожним полем відображайте повідомлення про помилку з `formState.errors`
- Кнопка «Додати задачу» має атрибут `type="submit"`

> **Підказка щодо `<select>` з Zod:** додайте першою опцію-заглушку, щоб Zod міг показати помилку при незаповненому полі:
>
> ```tsx
> <select {...register("priority")}>
>   <option value="">Оберіть пріоритет</option>
>   <option value="low">Низький</option>
>   <option value="medium">Середній</option>
>   <option value="high">Високий</option>
> </select>
> ```

5.3. Додайте стилі форми у `TaskForm.module.css`:

- `.form` - вертикальний flex-контейнер з відступами
- `.field` - обгортка поля (label + input/textarea/select + помилка)
- `.error` - стиль для повідомлення про помилку (червоний колір, дрібний шрифт)
- `.submit` - стиль кнопки сабміту

  5.4. Зафіксуйте:

```bash
git add .
git commit -m "Lab 3: Add TaskForm with React Hook Form and Zod validation"
```

---

### **Етап 6: Збірка застосунку в App.tsx**

6.1. Визначте початкові задачі **поза** компонентом. Значення `VARIANT` впливає на вміст - кожен студент підставляє свій номер у журналі:

```typescript
import type { Task } from "./types/task";

// Замініть 0 на ваш номер у журналі (1–30)
const VARIANT = 0;

const INITIAL_TASKS: Task[] = [
  {
    id: `task-${VARIANT}-1`,
    title: `Задача А-${VARIANT}: налаштування середовища`,
    description: "Встановити Node.js, VS Code та необхідні розширення",
    status: "done",
    priority: "high",
    createdAt: new Date(2025, 0, (VARIANT % 28) + 1),
  },
  {
    id: `task-${VARIANT}-2`,
    title: `Задача Б-${VARIANT}: вивчення документації`,
    description: "Ознайомитись з офіційною документацією React",
    status: "in-progress",
    priority: "medium",
    createdAt: new Date(2025, 1, (VARIANT % 28) + 1),
  },
  {
    id: `task-${VARIANT}-3`,
    title: `Задача В-${VARIANT}: написати компонент`,
    description: "",
    status: "todo",
    priority: "low",
    createdAt: new Date(2025, 2, (VARIANT % 28) + 1),
  },
];
```

6.2. Реалізуйте кореневий компонент `src/App.tsx`. Імпортуйте `TaskFormData` з `TaskForm.tsx`, щоб типізувати `handleAddTask`:

```tsx
import { useState } from "react";
import type { TaskStatus } from "./types/task";
import type { TaskFormData } from "./components/TaskForm/TaskForm";
import TaskForm from "./components/TaskForm/TaskForm";
import TaskList from "./components/TaskList/TaskList";
import styles from "./App.module.css";
```

Стан та логіка компонента:

- `const [tasks, setTasks] = useState(INITIAL_TASKS)`
- `const [filter, setFilter] = useState<TaskStatus | 'all'>('all')`
- Функція `handleAddTask` приймає `TaskFormData` та формує повний об'єкт `Task`:
  - `id: crypto.randomUUID()`
  - `status: 'todo'`
  - `createdAt: new Date()`
- Функція `handleDeleteTask` - видаляє задачу за `id` іммутабельно
- Функція `handleStatusChange` - оновлює статус задачі іммутабельно
- Відфільтровані задачі обчислюються перед поверненням JSX:

```tsx
const filteredTasks =
  filter === "all" ? tasks : tasks.filter((task) => task.status === filter);
```

6.3. Розмістіть компоненти у розмітці `App.tsx`:

```tsx
return (
  <div className={styles.app}>
    <header className={styles.header}>
      <h1>Task Manager</h1>
      <p className={styles.stats}>
        Всього: {tasks.length} | Нові:{" "}
        {tasks.filter((t) => t.status === "todo").length} | В роботі:{" "}
        {tasks.filter((t) => t.status === "in-progress").length} | Виконані:{" "}
        {tasks.filter((t) => t.status === "done").length}
      </p>
    </header>

    <main className={styles.main}>
      <aside className={styles.sidebar}>
        <TaskForm onSubmit={handleAddTask} />
      </aside>

      <section className={styles.content}>
        <div className={styles.filters}>
          <label htmlFor="filter">Фільтр:</label>
          <select
            id="filter"
            value={filter}
            onChange={(e) => setFilter(e.target.value as TaskStatus | "all")}
          >
            <option value="all">Усі</option>
            <option value="todo">Нові</option>
            <option value="in-progress">В роботі</option>
            <option value="done">Виконані</option>
          </select>
        </div>

        <TaskList
          tasks={filteredTasks}
          onDelete={handleDeleteTask}
          onStatusChange={handleStatusChange}
        />
      </section>
    </main>
  </div>
);
```

6.4. Додайте базові стилі у `App.module.css` (двоколонковий макет через grid або flex, стилі для header та stats).

6.5. Зафіксуйте:

```bash
git add .
git commit -m "Lab 3: Assemble App with state, filtering and layout"
```

---

### **Етап 7: Публікація на GitHub**

7.1. Переконайтесь у наявності описової історії комітів:

```bash
git log --oneline
```

Очікувана структура:

```
xxxxxxx Lab 3: Add README
xxxxxxx Lab 3: Assemble App with state, filtering and layout
xxxxxxx Lab 3: Add TaskForm with React Hook Form and Zod validation
xxxxxxx Lab 3: Add TaskList component
xxxxxxx Lab 3: Add TaskCard component
xxxxxxx Lab 3: Add project structure and Task types
xxxxxxx Lab 3: Initial Vite + React + TypeScript setup
```

7.2. Створіть файл `lab3/README.md`:

```markdown
# Лабораторна робота №3 - React-застосунок

**Варіант:** [номер]

## Опис

Task Manager - односторінковий React-застосунок для управління задачами.
Підтримує створення, видалення, зміну статусу та фільтрацію задач за статусом.

## Технології

- React 18 + TypeScript
- Vite
- React Hook Form + Zod (валідація форм)
- CSS Modules

## Структура

- `src/types/` - TypeScript-інтерфейси домену
- `src/components/TaskForm/` - форма створення задачі
- `src/components/TaskCard/` - картка окремої задачі
- `src/components/TaskList/` - список задач

## Запуск

npm install
npm run dev
```

7.3. Відправте всі зміни:

```bash
git add .
git commit -m "Lab 3: Add README"
git push origin main
```

**Результат:** Повністю функціональний застосунок опублікований у репозиторії.

---

## **Зміст звіту**

1. **Титульна сторінка** з темою, метою роботи, обладнанням, програмним забезпеченням

2. **Відповіді на контрольні питання** (коротко та по суті)

3. **Виконання роботи:**

   3.1. Скріншот структури проєкту у VS Code (дерево файлів папки `lab3/src`)

   3.2. Скріншот коду та результату для кожного компонента:
   - `TaskCard` - картка з відображенням пріоритету та кнопками
   - `TaskList` - список задач та стан «немає задач» при порожньому масиві
   - `TaskForm` - форма зі спрацьованою валідацією (скріншот з помилками під полями)

     3.3. Скріншот готового застосунку в браузері з кількома задачами різного пріоритету

     3.4. Скріншот роботи фільтрації (застосунок з увімкненим фільтром, що показує лише частину задач)

     3.5. Скріншот React Developer Tools з деревом компонентів

     3.6. Скріншот історії комітів (вкладка Commits на GitHub)

4. **Посилання** на GitHub-репозиторій

5. **Висновки** (що реалізовано, які навички отримано, з якими труднощами зіткнулись)

---

## **Типові помилки та їх вирішення**

| Проблема                                                           | Рішення                                                                                                                          |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| `Argument of type 'string' is not assignable to type 'TaskStatus'` | Використовуйте `as TaskStatus` після `e.target.value` у `<select>`. Це безпечно, якщо `<option>` містять лише допустимі значення |
| Форма не скидається після сабміту                                  | Викличте `reset()` з `useForm` у тілі `onSubmit` після виклику зовнішнього `onSubmit`                                            |
| Видалення або оновлення задачі не відображається в UI              | Перевірте іммутабельність: `setTasks` має отримувати новий масив, а не змінений існуючий                                         |
| Zod не показує помилку для порожнього `<select>`                   | Додайте першою опцію-заглушку з `value=""` - тоді порожній рядок не відповідатиме `z.enum` і Zod поверне помилку                 |
| `key` prop warning у консолі                                       | Переконайтесь, що `key={task.id}` встановлено безпосередньо на елемент, що повертається з `.map()`                               |
| Стилі CSS Modules не застосовуються                                | Перевірте, що файл має розширення `.module.css` і клас звертається як `styles.назваКласу`, а не як рядок                         |

---

## **Додаткові матеріали**

1. Сучасний підручник JavaScript - https://uk.javascript.info/
2. React офіційна документація - https://react.dev/
3. React Hook Form документація - https://react-hook-form.com/
4. Zod документація - https://zod.dev/
