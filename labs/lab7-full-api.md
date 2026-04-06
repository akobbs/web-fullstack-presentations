# **Лабораторна робота №7**

**Тема:** CRUD API на Nest.js

**Мета:** Реалізувати повноцінний CRUD API на Nest.js — операції створення, читання, оновлення та видалення задач із використанням сервісів, ін'єкції залежностей, DTO та валідації вхідних даних.

**Обладнання:** IBM-сумісні ПК

**Програмне забезпечення:** ОС Windows 10/11, macOS або Linux; Node.js (v18+); npm; VS Code; обліковий запис GitHub; Git; curl або Postman

---

## **Контрольні запитання**

1. Яка відповідальність сервісу (Service) в архітектурі Nest.js? Чому сервіс не повинен містити HTTP-виключення?
2. Що таке Dependency Injection? Чим він кращий за самостійне створення залежності через `new` всередині класу?
3. Яка різниця між `CreateTaskDto` та `UpdateTaskDto`? Чому поля `UpdateTaskDto` є необов'язковими?
4. Що робить параметр `whitelist: true` у `ValidationPipe`? Яку загрозу він усуває?
5. Що повертає сервіс, якщо задачу не знайдено? Де і яке виключення кидається у відповідь?

---

## **Теоретичні відомості**

### **Розподіл відповідальностей: Service та Controller**

**Сервіс** відповідає виключно за операції з даними. Він не знає нічого про HTTP — повертає або результат, або `null` (для методів пошуку) чи `boolean` (для видалення). **Контролер** перевіряє результат і вирішує, який HTTP-статус і відповідь повернути клієнту. HTTP-виключення (`NotFoundException` тощо) належать виключно контролеру.

```typescript
// ⚙️ Сервіс — лише робота з даними
// Імпорти: Injectable — з '@nestjs/common'
@Injectable()
export class TasksService {
  private tasks: Task[] = []; // масив задач у пам'яті

  findOne(id: string): Task | null {
    return this.tasks.find((t) => t.id === id) ?? null;
  }

  remove(id: string): boolean {
    const index = this.tasks.findIndex((t) => t.id === id);
    if (index === -1) return false;
    this.tasks.splice(index, 1);
    return true;
  }
}

// 📬 Контролер — перевіряє результат і кидає HTTP-виключення
// Імпорти: Controller, Get, Delete, Param, HttpCode, NotFoundException — з '@nestjs/common'
@Controller("tasks")
export class TasksController {
  constructor(private readonly tasksService: TasksService) {}

  @Get(":id")
  findOne(@Param("id") id: string): Task {
    const task = this.tasksService.findOne(id);
    if (!task) throw new NotFoundException(`Завдання #${id} не знайдено`);
    return task;
  }

  @Delete(":id")
  @HttpCode(204)
  remove(@Param("id") id: string): void {
    const removed = this.tasksService.remove(id);
    if (!removed) throw new NotFoundException(`Завдання #${id} не знайдено`);
  }
}
```

### **DTO та ValidationPipe**

**DTO (Data Transfer Object)** — клас TypeScript, що описує структуру та правила валідації вхідних даних ендпоінту. Використовується саме клас, а не інтерфейс, оскільки декоратори `class-validator` прикріплюються до класу як метадані в рантаймі — саме їх `ValidationPipe` читає під час перевірки.

```typescript
// src/tasks/dto/create-task.dto.ts
import {
  IsString,
  IsNotEmpty,
  IsIn,
  IsOptional,
  MaxLength,
} from "class-validator";

export class CreateTaskDto {
  @IsString()
  @IsNotEmpty({ message: "Назва не може бути порожньою" })
  @MaxLength(100)
  title: string;

  @IsOptional()
  @IsString()
  @MaxLength(500)
  description?: string;

  @IsIn(["low", "medium", "high"], {
    message: "Пріоритет має бути: low, medium або high",
  })
  priority: "low" | "medium" | "high";
}
```

Для `PATCH`-ендпоінту всі поля є необов'язковими — клієнт може оновити лише частину задачі. Такий DTO оголошується окремо: всі поля позначаються `@IsOptional()`.

`ValidationPipe` підключається глобально в `main.ts`. Параметр `whitelist: true` автоматично видаляє з тіла запиту будь-які поля, яких немає в DTO — це захист від масового призначення (mass assignment).

### **HTTP-статуси**

| Ситуація                                  | Статус          |
| ----------------------------------------- | --------------- |
| Задачу знайдено / успішно оновлено        | 200 OK          |
| Задачу успішно створено                   | 201 Created     |
| Задачу видалено (тіло відповіді відсутнє) | 204 No Content  |
| Задачу не знайдено                        | 404 Not Found   |
| Невалідні дані у запиті                   | 400 Bad Request |

Статуси 201 та 204 задаються явно через декоратор `@HttpCode()`. Статус 404 генерується автоматично при виклику `NotFoundException`.

---

## **Вимоги до реалізації**

### **Проєкт**

- Nest.js застосунок зі стандартною структурою, згенерований через CLI
- Порт сервера зчитується зі змінних середовища через `@nestjs/config`; файл `.env` не потрапляє до репозиторію, поруч наявний `.env.example`
- Глобальний `ValidationPipe` з параметрами `whitelist: true` та `transform: true` підключений у `main.ts`

### **Модуль задач**

- Тип `Task` з полями: `id`, `title`, `description`, `status`, `priority`, `createdAt`
- `CreateTaskDto` та `UpdateTaskDto` з декораторами валідації (`class-validator`); усі поля `UpdateTaskDto` є необов'язковими
- Сервіс із методами `findAll`, `findByStatus`, `findOne`, `create`, `update`, `remove`; дані зберігаються в масиві в пам'яті; сервіс не містить HTTP-виключень — повертає дані, `null` або `boolean`
- Початковий масив містить щонайменше три задачі з різними статусами та пріоритетами

### **Ендпоінти**

| Метод  | URL                      | Статус                                       |
| ------ | ------------------------ | -------------------------------------------- |
| GET    | /tasks                   | 200 (порожній масив якщо задач немає)        |
| GET    | /tasks/search?status=... | 200 (порожній масив якщо нічого не знайдено) |
| GET    | /tasks/:id               | 200 / 404                                    |
| POST   | /tasks                   | 201 / 400                                    |
| PATCH  | /tasks/:id               | 200 / 400 / 404                              |
| DELETE | /tasks/:id               | 204 / 404                                    |

### **Тестування**

Перевірте за допомогою curl або Postman:

- коректну роботу кожного ендпоінта
- повернення `400` з переліком порушень при невалідних даних
- роботу `whitelist: true` — надішліть запит із зайвим полем (наприклад, `isAdmin: true`), потім отримайте задачу через GET і переконайтесь, що зайвого поля в ній немає
- повернення `404` при зверненні до неіснуючого `id`

### **Публікація**

- Проєкт опублікований на GitHub з описовою історією комітів
- `README.md` містить таблицю ендпоінтів із HTTP-статусами

---

## **Зміст звіту**

1. **Титульна сторінка** з темою, метою роботи, обладнанням, програмним забезпеченням

2. **Відповіді на контрольні запитання** (коротко та по суті)

3. **Виконання роботи:** скріншоти реалізованого коду, результатів тестування всіх ендпоінтів (включно з граничними сценаріями) та історії комітів на GitHub

4. **Посилання** на GitHub-репозиторій

5. **Висновок** (3–5 речень): що реалізовано, які архітектурні рішення прийнято, які навички отримано

---

## **Додаткові матеріали**

- [Офіційна документація Nest.js — Providers](https://docs.nestjs.com/providers)
- [Офіційна документація Nest.js — Pipes](https://docs.nestjs.com/pipes)
- [Офіційна документація Nest.js — Exception filters](https://docs.nestjs.com/exception-filters)
- [class-validator — список декораторів](https://github.com/typestack/class-validator#validation-decorators)
