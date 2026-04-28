# Самостійна робота №11: E2E тестування

**Тема:** E2E тестування

**Мета:** Ознайомитися з підходом End-to-End тестування, вивчити основи Playwright або Cypress, навчитися писати E2E тести та інтегрувати їх з CI.

**Кількість годин:** 3

---

## Теоретичний матеріал

### Що таке E2E тестування

End-to-End (E2E) тестування перевіряє роботу застосунку від початку до кінця з точки зору реального користувача. На відміну від unit-тестів (окремі функції) та інтеграційних тестів (взаємодія модулів), E2E тести працюють з реальним браузером, взаємодіючи зі всіма шарами: UI, API, база даних.

E2E тести найдорожчі у підтримці, але найбільш наближені до реальності. Піраміда тестування рекомендує: багато unit-тестів, менше інтеграційних, ще менше E2E.

### Playwright

Playwright - сучасний фреймворк для E2E тестування від Microsoft. Підтримує Chromium, Firefox, WebKit (Safari). Особливості: автоматичне очікування елементів (auto-waiting), мережевий перехват, паралельне виконання, трейси для дебагу.

Встановлення: `npm init playwright@latest`. Основний API:

```typescript
import { test, expect } from '@playwright/test';

test('user can login', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'user@test.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toHaveText('Dashboard');
});
```

### Cypress

Cypress - популярний фреймворк E2E тестування з інтерактивним Test Runner. Працює всередині браузера, що дає прямий доступ до DOM. Особливості: time-travel debugging (візуальна історія кроків), автоматичні скріншоти та відео, перехоплення мережевих запитів (`cy.intercept()`).

```javascript
describe('Login', () => {
  it('should login successfully', () => {
    cy.visit('/login');
    cy.get('[name="email"]').type('user@test.com');
    cy.get('[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
    cy.get('h1').should('contain', 'Dashboard');
  });
});
```

### Основні патерни E2E тестів

**Page Object Model (POM)** - інкапсуляція взаємодії зі сторінкою у клас. Зменшує дублювання та спрощує підтримку. **Fixtures** - підготовка тестових даних. **Тестові селектори** - використання `data-testid` для стабільних селекторів.

### Інтеграція з CI

E2E тести запускаються у CI/CD pipeline (GitHub Actions, GitLab CI). Потрібен headless-браузер. Playwright включає готові Docker-образи. Стратегії: запуск на кожний PR, нічні запуски повного набору, паралелізація для швидкості.

---

## Контрольні питання

1. Чим E2E тести відрізняються від unit та інтеграційних тестів?
2. Що таке піраміда тестування і яке місце у ній займають E2E тести?
3. Які переваги Playwright порівняно з Cypress?
4. Що таке auto-waiting у Playwright і яку проблему він вирішує?
5. Що таке Page Object Model і чому він рекомендується для E2E тестів?
6. Чому для тестових селекторів краще використовувати `data-testid`, а не CSS-класи?
7. Як інтегрувати E2E тести з GitHub Actions?

---

## Практичне завдання

Напишіть E2E тести (Playwright або Cypress) для типової сторінки авторизації:

1. Тест: сторінка логіну відображається коректно (всі поля та кнопки присутні).
2. Тест: помилка при спробі входу з невалідними даними.
3. Тест: успішний вхід та перенаправлення на головну сторінку.

---

## Порядок виконання

1. Вивчити теоретичний матеріал, наведений у цьому документі.
2. Ознайомитись з рекомендованими джерелами.
3. Відповісти на контрольні питання.
4. Виконати практичне завдання.

---

## Рекомендовані джерела

- [Playwright - документація](https://playwright.dev/docs/intro) - офіційна документація Playwright
- [Cypress - документація](https://docs.cypress.io/) - офіційна документація Cypress
- [Playwright - Best Practices](https://playwright.dev/docs/best-practices) - рекомендації по написанню тестів
- [Playwright - CI/CD](https://playwright.dev/docs/ci) - інтеграція з CI
- [Testing Trophy - Kent C. Dodds](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) - альтернативний погляд на піраміду тестування
