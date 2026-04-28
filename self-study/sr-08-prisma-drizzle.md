# Самостійна робота №8: Альтернативні ORM: Prisma та Drizzle

**Тема:** Альтернативні ORM: Prisma та Drizzle

**Мета:** Ознайомитися з ORM Prisma та Drizzle як альтернативами TypeORM, вивчити їх основні концепції, порівняти підходи та зрозуміти переваги кожного рішення.

**Кількість годин:** 3

---

## Теоретичний матеріал

### Prisma

Prisma - сучасна ORM для Node.js та TypeScript з унікальним підходом: модель даних описується у власному файлі схеми (`schema.prisma`), а не через класи чи декоратори.

**Prisma Schema** - декларативний опис моделей, зв'язків та налаштувань БД. Підтримує PostgreSQL, MySQL, SQLite, MongoDB, SQL Server. Приклад:

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

**Prisma Client** - автоматично згенерований типобезпечний клієнт для запитів. Генерується командою `npx prisma generate`. Запити: `prisma.user.findMany()`, `prisma.user.create()`, `prisma.user.update()`. Підтримує фільтрацію, сортування, пагінацію, вкладені запити (include/select).

**Prisma Migrate** - система міграцій на основі змін у schema. Команди: `prisma migrate dev` (розробка), `prisma migrate deploy` (продакшн). Prisma Studio - веб-інтерфейс для перегляду та редагування даних.

### Drizzle

Drizzle ORM - легковісна, SQL-подібна, типобезпечна ORM. Філософія: бути максимально близькою до SQL, зберігаючи типобезпеку TypeScript.

Схема описується TypeScript-кодом:

```typescript
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: text('name'),
});
```

Запити нагадують SQL: `db.select().from(users).where(eq(users.email, 'test@test.com'))`. Drizzle також підтримує relational queries для зручної роботи зі зв'язками. Drizzle Kit - CLI для генерації та виконання міграцій.

### Порівняння з TypeORM

| Критерій | TypeORM | Prisma | Drizzle |
|----------|---------|--------|---------|
| Підхід | Active Record / Data Mapper | Schema-first | SQL-like |
| Типобезпека | Часткова | Повна (генерація) | Повна (інференція) |
| Схема | Декоратори TypeScript | Prisma Schema | TypeScript-код |
| Міграції | Автогенерація + ручні | Автоматичні з schema | CLI генерація |
| Розмір бандлу | Середній | Великий (engine) | Мінімальний |
| SQL-контроль | Середній | Обмежений | Максимальний |

---

## Контрольні питання

1. Як описується модель даних у Prisma Schema? Чим цей підхід відрізняється від TypeORM?
2. Що таке Prisma Client і як він генерується?
3. Як працює система міграцій у Prisma?
4. Яка філософія Drizzle ORM і чому її називають "SQL-like"?
5. Як описується схема БД у Drizzle? Чим цей підхід відрізняється від Prisma?
6. Порівняйте типобезпеку TypeORM, Prisma та Drizzle.
7. У яких сценаріях краще обрати Prisma, а в яких - Drizzle?

---

## Порядок виконання

1. Вивчити теоретичний матеріал, наведений у цьому документі.
2. Ознайомитись з рекомендованими джерелами.
3. Відповісти на контрольні питання.

---

## Рекомендовані джерела

- [Prisma - офіційна документація](https://www.prisma.io/docs) - повна документація Prisma
- [Prisma - Getting Started](https://www.prisma.io/docs/getting-started) - швидкий старт
- [Drizzle ORM - документація](https://orm.drizzle.team/docs/overview) - повна документація Drizzle
- [Drizzle ORM - Getting Started](https://orm.drizzle.team/docs/get-started) - швидкий старт Drizzle
- [Prisma vs Drizzle - порівняння](https://www.prisma.io/docs/orm/more/comparisons/prisma-and-drizzle) - офіційне порівняння від Prisma
- [TypeORM vs Prisma](https://www.prisma.io/docs/orm/more/comparisons/prisma-and-typeorm) - порівняння TypeORM та Prisma
