# **Лабораторна робота №6**

**Тема:** Перший API на Nest.js

**Мета:** Створити структурований серверний застосунок на Nest.js з базовими HTTP-ендпоінтами для роботи з задачами. Закріпити навички роботи з модулями, контролерами, декораторами та змінними середовища.

**Обладнання:** IBM-сумісні ПК

**Програмне забезпечення:** ОС Windows 10/11, macOS або Linux; Node.js (v18+); npm; VS Code; обліковий запис GitHub; Git; curl або Postman

---

## **Контрольні запитання**

1. Яка різниця між контролером (Controller) та провайдером (Provider) в архітектурі Nest.js?
2. Що таке декоратор у TypeScript? Наведіть приклад декоратора HTTP-методу.
3. Яку роль виконує `@Module()` і що передається у його властивості `controllers`?
4. Чим відрізняються `@Param()` та `@Query()` — коли використовується кожен?
5. Навіщо зберігати налаштування у `.env` файлі та чому не можна додавати його в Git?

---

## **Теоретичні відомості**

### **Архітектура Nest.js**

Nest.js організовує код через три основні будівельні блоки.

**Модуль (Module)** — клас-контейнер, що об'єднує пов'язані компоненти. Кожен застосунок має щонайменше один кореневий модуль `AppModule`. Модулі реєструють контролери та провайдери через декоратор `@Module()`.

**Контролер (Controller)** — клас, що приймає HTTP-запити та повертає відповіді. Маршрут формується з базового шляху контролера (`@Controller('tasks')`) та декоратора методу (`@Get()`, `@Post()` тощо).

**Провайдер (Provider)** — будь-який клас з декоратором `@Injectable()`, що може бути впроваджений як залежність. Сервіси є найпоширенішим видом провайдерів.

```typescript
// Структура модуля з контролером (без сервісу — для поточної лабораторної)
@Module({
  controllers: [TasksController],
})
export class TasksModule {}
```

### **Контролери та декоратори**

Контролер описується декларативно через декоратори. Nest.js автоматично зіставляє вхідний запит з відповідним методом контролера:

```typescript
@Controller('tasks')           // базовий маршрут: /tasks
export class TasksController {

  @Get()                       // GET /tasks
  findAll() { ... }

  @Get('search')               // GET /tasks/search?status=pending
  findByStatus(@Query('status') status?: string) { ... }

  @Get(':id')                  // GET /tasks/42
  findOne(@Param('id') id: string) { ... }

  @Post()                      // POST /tasks
  create(@Body() body: CreateTaskDto) { ... }

  @Delete(':id')               // DELETE /tasks/42
  remove(@Param('id') id: string) { ... }
}
```

> **Важливо:** маршрути з фіксованим сегментом (`search`) мають бути оголошені **до** маршрутів з параметром (`:id`). Інакше Nest.js інтерпретує рядок `search` як значення параметра `id`.

### **DTO — об'єкт передачі даних**

DTO (Data Transfer Object) — простий клас, що описує форму вхідних даних запиту. На цьому етапі DTO виконує роль типової анотації без валідації:

```typescript
// src/tasks/dto/create-task.dto.ts
export class CreateTaskDto {
  title: string;
  description?: string;
  priority: "low" | "medium" | "high";
}
```

### **Змінні середовища та ConfigModule**

Чутливі налаштування (порти, паролі, ключі) зберігаються у файлі `.env`, який **ніколи не додається в Git**. `ConfigModule` із пакету `@nestjs/config` завантажує цей файл та робить змінні доступними через `process.env`:

```
# .env
PORT=3000
```

```typescript
// app.module.ts
@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), TasksModule],
})
export class AppModule {}
```

---

## **Хід роботи**

### **Етап 1: Створення проєкту**

1.1. Встановіть Nest CLI глобально (якщо ще не встановлено):

```bash
npm install -g @nestjs/cli
```

1.2. Перейдіть у папку `lab6/` (або створіть її) та згенеруйте проєкт:

```bash
mkdir lab6
cd lab6
nest new task-manager-api
cd task-manager-api
```

Виберіть `npm` як пакетний менеджер. Команда `nest new` створить папку `task-manager-api` з усіма файлами всередині.

1.3. Переконайтесь, що базовий сервер запускається:

```bash
npm run start:dev
```

Відкрийте у браузері `http://localhost:3000` — має відобразитись `Hello World!`.

1.4. Nest CLI автоматично створює `.gitignore` з `node_modules/` у списку виключень. Переконайтесь, що файл існує, і зафіксуйте початковий стан:

```bash
git add .
git commit -m "Lab 6: Initial Nest.js project setup"
```

---

### **Етап 2: Налаштування змінних середовища**

2.1. Встановіть пакет для роботи зі змінними середовища:

```bash
npm install @nestjs/config
```

2.2. Створіть файл `.env` у корені проєкту з таким вмістом:

```
PORT=3000
```

2.3. Переконайтесь, що `.env` є у `.gitignore` (Nest CLI додає його автоматично).

2.4. Підключіть `ConfigModule` у `src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
export class AppModule {}
```

> **Зверніть увагу:** на наступному етапі Nest CLI автоматично додасть `TasksModule` до цього файлу. Після генерації `app.module.ts` виглядатиме інакше — це очікувана поведінка.

2.5. Оновіть `src/main.ts`, щоб порт читався зі змінних середовища:

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const port = parseInt(process.env.PORT ?? "3000", 10);
  await app.listen(port);
  console.log(`Server started on port ${port}`);
}

bootstrap();
```

2.6. Створіть файл `.env.example` — шаблон зі змінними, який додається в Git. Для публічних значень за замовчуванням (як порт) значення можна залишити, чутливі дані (паролі, ключі) — залишати порожніми:

```
PORT=3000
```

2.7. Зафіксуйте:

```bash
git add .
git commit -m "Lab 6: Add ConfigModule and environment variables"
```

---

### **Етап 3: Структура модуля задач**

3.1. Згенеруйте модуль та контролер задач:

```bash
nest g module tasks
nest g controller tasks
```

Nest CLI автоматично зареєструє контролер у `TasksModule` та підключить `TasksModule` до `AppModule`.

3.2. Перегляньте згенеровану структуру:

```
src/
  tasks/
    tasks.module.ts
    tasks.controller.ts
    tasks.controller.spec.ts
```

3.3. Створіть файл `src/tasks/entities/task.entity.ts` з типами задачі:

```typescript
export type TaskStatus = "pending" | "in-progress" | "done";
export type TaskPriority = "low" | "medium" | "high";

export interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  priority: TaskPriority;
  createdAt: string;
}
```

3.4. Створіть папку `src/tasks/dto/` та файл `src/tasks/dto/create-task.dto.ts`:

```typescript
export class CreateTaskDto {
  title: string;
  description?: string;
  priority: "low" | "medium" | "high";
}
```

3.5. Зафіксуйте:

```bash
git add .
git commit -m "Lab 6: Add tasks module structure and types"
```

---

### **Етап 4: Реалізація контролера**

4.1. Замість сервісу зберігатимемо задачі у масиві всередині самого контролера. Це спрощений підхід: в реальних застосунках дані зберігаються у базі даних через окремий сервіс.

4.2. Заповніть `src/tasks/tasks.controller.ts`. Імпорти та сигнатури методів надано — реалізуйте кожен метод самостійно. Початковий масив `tasks` заповніть щонайменше трьома задачами з різними статусами та пріоритетами (це спростить перевірку фільтрації):

```typescript
import {
  Controller,
  Get,
  Post,
  Delete,
  Body,
  Param,
  Query,
} from "@nestjs/common";
import { Task } from "./entities/task.entity";
import { CreateTaskDto } from "./dto/create-task.dto";

@Controller("tasks")
export class TasksController {
  private tasks: Task[] = [
    // Додайте щонайменше 3 початкові задачі з різними статусами та пріоритетами.
    // Структура кожної задачі:
    // { id: '1', title: '...', description: '...', status: 'pending', priority: 'high', createdAt: '2025-01-01T10:00:00.000Z' }
  ];

  // GET /tasks
  // Повертає весь масив задач
  @Get()
  findAll(): Task[] {
    return []; // TODO: повернути масив tasks
  }

  // GET /tasks/search?status=pending
  // Якщо параметр status не передано — повертає всі задачі
  // Якщо передано — фільтрує масив за полем status
  // Важливо: цей маршрут має бути оголошений до @Get(':id')
  @Get("search")
  findByStatus(@Query("status") status?: string): Task[] {
    return []; // TODO: реалізувати фільтрацію
  }

  // GET /tasks/:id
  // Якщо задачу не знайдено — повернути об'єкт { message: '...' }
  @Get(":id")
  findOne(@Param("id") id: string): Task | { message: string } {
    return { message: "TODO" }; // TODO: знайти задачу за id
  }

  // POST /tasks
  // Створює нову задачу зі статусом 'pending' та поточним часом
  // id генерується як Date.now().toString()
  @Post()
  create(@Body() dto: CreateTaskDto): Task {
    throw new Error("Not implemented"); // TODO: створити та додати задачу до масиву
  }

  // DELETE /tasks/:id
  // Якщо задачу не знайдено — повернути об'єкт { message: '...' }
  // Якщо знайдено — видалити та повернути підтвердження
  @Delete(":id")
  remove(@Param("id") id: string): { message: string } {
    return { message: "TODO" }; // TODO: видалити задачу за id
  }
}
```

> **Зверніть увагу:** масив `tasks` зберігається в оперативній пам'яті — після перезапуску сервера дані скидаються до початкових. Це нормально для поточного завдання.

4.3. Зафіксуйте:

```bash
git add .
git commit -m "Lab 6: Implement TasksController"
```

---

### **Етап 5: Тестування ендпоінтів**

5.1. Запустіть сервер у режимі розробки:

```bash
npm run start:dev
```

5.2. Протестуйте кожен ендпоінт за допомогою curl (або Postman):

**GET /tasks** — отримати всі задачі:

```bash
curl http://localhost:3000/tasks
```

**GET /tasks/search?status=pending** — фільтрація за статусом:

```bash
curl "http://localhost:3000/tasks/search?status=pending"
```

**GET /tasks/:id** — отримати одну задачу за id (підставте реальний id з ваших початкових даних):

```bash
curl http://localhost:3000/tasks/1
```

**POST /tasks** — створити нову задачу:

```bash
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Нова задача", "description": "Опис", "priority": "medium"}'
```

> На Windows (cmd/PowerShell) використовуйте однорядковий варіант з подвійними лапками:
>
> ```
> curl -X POST http://localhost:3000/tasks -H "Content-Type: application/json" -d "{\"title\": \"Нова задача\", \"priority\": \"medium\"}"
> ```

**DELETE /tasks/:id** — видалити задачу:

```bash
curl -X DELETE http://localhost:3000/tasks/1
```

**Перевірка неіснуючого id:**

```bash
curl http://localhost:3000/tasks/999
```

Відповідь має містити поле `message` з повідомленням про відсутність задачі.

5.3. Якщо під час тестування ви виявили та виправили помилки в коді — зафіксуйте зміни:

```bash
git add .
git commit -m "Lab 6: Fix controller after testing"
```

---

### **Етап 6: Публікація на GitHub**

6.1. Переконайтесь у наявності описової історії комітів:

```bash
git log --oneline
```

Очікувана структура (коміт з виправленнями присутній лише якщо були знайдені помилки):

```
xxxxxxx Lab 6: Add README
xxxxxxx Lab 6: Fix controller after testing   ← якщо були виправлення
xxxxxxx Lab 6: Implement TasksController
xxxxxxx Lab 6: Add tasks module structure and types
xxxxxxx Lab 6: Add ConfigModule and environment variables
xxxxxxx Lab 6: Initial Nest.js project setup
```

6.2. Створіть файл `README.md` у корені проєкту:

```markdown
# Лабораторна робота №6 — Перший API на Nest.js

## Опис

Task Manager API — серверний застосунок на Nest.js для управління задачами.
Підтримує отримання списку, фільтрацію за статусом, створення та видалення задач.

## Технології

- Nest.js + TypeScript
- @nestjs/config

## Ендпоінти

| Метод  | URL                      | Опис                   |
| ------ | ------------------------ | ---------------------- |
| GET    | /tasks                   | Отримати всі задачі    |
| GET    | /tasks/search?status=... | Фільтрація за статусом |
| GET    | /tasks/:id               | Отримати задачу за id  |
| POST   | /tasks                   | Створити нову задачу   |
| DELETE | /tasks/:id               | Видалити задачу        |

## Запуск

1. Скопіюйте `.env.example` у `.env`
2. `npm install`
3. `npm run start:dev`
```

6.3. Відправте всі зміни:

```bash
git add .
git commit -m "Lab 6: Add README"
git push origin main
```

---

## **Зміст звіту**

1. **Титульна сторінка** з темою, метою роботи, обладнанням, програмним забезпеченням

2. **Відповіді на контрольні запитання** (коротко та по суті)

3. **Виконання роботи:**

   3.1. Скріншот структури проєкту у VS Code (дерево файлів папки `src/`)

   3.2. Скріншот файлів `.env` та `.env.example`

   3.3. Скріншот коду `tasks.controller.ts` з реалізованими ендпоінтами та початковими даними

   3.4. Скріншоти результатів тестування кожного ендпоінта (GET всіх, GET з фільтром, GET одного, POST, DELETE) — термінал з curl або Postman

   3.5. Скріншот відповіді при зверненні до неіснуючого id

   3.6. Скріншот термінала з запущеним сервером

   3.7. Скріншот історії комітів (вкладка Commits на GitHub)

4. **Посилання** на GitHub-репозиторій

5. **Висновок** (3–5 речень): що було реалізовано, які навички отримано

---

## **Додаткові матеріали**

- [Офіційна документація Nest.js — Controllers](https://docs.nestjs.com/controllers)
- [Офіційна документація Nest.js — Modules](https://docs.nestjs.com/modules)
- [Офіційна документація Nest.js — Configuration](https://docs.nestjs.com/techniques/configuration)
