# Нотатки PoC: що я побачив у запущеному застосунку

**Контекст:** PoC перевіряв зміни з `docs/contact-form-feature.md` (День 2) і локальний код на `npm run dev`. **Не** зводився до макету [Figma](https://match-daily-72208179.figma.site/) — аналіз макету окремо в `docs/figma-contact-form-analysis.md`.

**Середовище:** гілка `day-3`, `/uk/contacts` і `/en/contacts`, форма «Напишіть нам» (демо: дані не зберігаються після рестарту).

### Перелік файлів у PoC-дифі (`git diff`, 12 файлів)

Статистика: **12 files changed**, 65 insertions(+), 64 deletions(−).

| # | Файл | Проба | Що змінилось |
|---|------|-------|----------------|
| 1 | `app/shared/components/forms/contact-form/ContactForm.tsx` | 1 + 2 | P1: обов’язковий телефон. P2: `lengths.MESSAGE_MIN_LENGTH` замість `.min(10)`. |
| 2 | `app/lib/utils/validateContactData.ts` | **1** | Телефон перевіряється завжди (не лише `if (phoneNumber)`). |
| 3 | `app/shared/components/forms/offer-collaboration-form/OfferCollaborationForm.tsx` | **1** | `phoneNumber: string`. |
| 4 | `app/services/email/emails/emails.ts` | **1** | Номер у листі завжди, без гілки «без телефону». |
| 5 | `app/services/email/emailService.ts` | **1** | Типи: `phoneNumber: string`. |
| 6 | `i18n/messages/uk.json` | 1 + 2 | P1: `phoneNumber *`, `phoneNumberRequired`. P2: `messageMinLength` (5). |
| 7 | `i18n/messages/en.json` | 1 + 2 | Те саме англійською. |
| 8 | `app/shared/components/forms/contact-form/ContactForm.test.tsx` | 1 + 2 | P1: телефон у happy path, `*`. P2: `Прив`, текст min 5. |
| 9 | `app/lib/utils/validateContactData.test.ts` | 1 + 2 | P1: `phoneNumber` у valid payload, all-invalid. P2: `Four`, min length 5. |
| 10 | `app/services/email/emails/emails.test.ts` | **1** | Прибрані кейси «without phone number»; `phoneNumber` у mockData. |
| 11 | `app/services/email/emailServise.test.ts` | **1** | `phoneNumber` у виклику `sendContactEmail`. |
| 12 | `app/constants/validation.ts` | **2** | `MESSAGE_MIN_LENGTH`: 10 → 5. |

**Не входить до diff:** `app/constants/errors.ts`. У цьому файлі `MESSAGE_ERROR` збирається з `lengths.MESSAGE_MIN_LENGTH` — після зміни **лише** `validation.ts` текст серверної помилки для короткого повідомлення стає «between **5** and 1000 characters» **без** правки `errors.ts`. `PHONE_NUMBER_ERROR` лишається `Invalid phone number`.

---

## Проба 1: обов’язковий телефон

**Гіпотеза:** порожнє поле телефону блокує відправку (AC-01, RULE-04).

### Що змінили

Файли з таблиці вище з позначкою **Проба 1** (рядки 1–5, 8–9 частково, 10–11, i18n — лише телефон). Повний список — у **`git diff`**; **`errors.ts` не в diff**.

### Що вводили в браузері

| # | Мова | Сторінка | Ввід | Результат |
|---|------|----------|------|-----------|
| A | **uk** | `/uk/contacts` | Ім’я: `Олена`, email: `o@example.com`, повідомлення: `Хочу долучитися`, згода ✓, **телефон порожній** | Кнопка «Надіслати запит» не викликає успішну відправку; під полем телефону — помилка |
| B | **uk** | те саме | + телефон `+380631164627` (або маска `+380 (63) 116-4627`) | Форма проходить клієнтську валідацію, з’являється підтвердження (демо) |
| C | **en** | `/en/contacts` | Як A, імена/тексти англійською | Та сама логіка, тексти з `en.json` |

### Точний текст помилки (після змін Проби 1)

| Мова | Умова | Текст на екрані (ключ i18n) |
|------|--------|-----------------------------|
| **uk** | Порожній телефон | **Поле обов'язкове для заповнення** (`contactForm.errors.phoneNumberRequired`) |
| **uk** | Невалідний формат (не ця проба, але бачили поруч) | **Перевірте формат номера телефону** (`phoneNumberInvalid`) |
| **en** | Порожній телефон | **This field is required** (аналог `phoneNumberRequired` у `en.json`) |
| Підпис поля | uk / en | **Номер телефону \*** / **Phone number \*** |

### Тести

**Команда (вузький прогін):**

```bash
npm test -- --testPathPattern="ContactForm|validateContactData" --coverage=false
```

**Після Проби 1:** Test Suites: **2 passed**, 2 total · Tests: **21 passed**, 21 total.

**Що було червоним** (після зміни коду, поки тести ще описували необов’язковий телефон):

| Тест | Чому падав |
|------|------------|
| `ContactForm.test.tsx` → `should call onSubmit when submit button is clicked` | Happy path **без** номера: очікувався 1 виклик `onSubmit`, після обов’язкового телефону — **0** |
| `validateContactData.test.ts` → `should return no errors for valid input` | У payload не було `phoneNumber`; сервер почав вимагати номер → масив помилок **не порожній** |
| `emails.test.ts` → кейси *without phone number* (якщо запускали в набір) | Шаблони листів після змін очікують номер у кожному зверненні |

**Чому стали зеленими:** у валідних сценаріях додали `phoneNumber`; у `ContactForm.test` — номер у successful submit і/або окремий кейс порожнього телефону; у `validateContactData` — тест «без телефону → помилка»; оновили мок перекладів (`phoneNumber *`, `phoneNumberRequired`).

### Знахідки (2–3)

1. **Правило на екрані:** `ContactForm.tsx` — Zod `phoneSchema`: прибрано `optional()` і `z.literal('')`, додано `.min(1, phoneNumberRequired)`.
2. **Правило на API:** `validateContactData.ts` — раніше `if (phoneNumber) { … }`; після змін перевірка **завжди** (порожнє / відсутнє → не проходить).
3. **Що ще зачепило:** та сама `ContactForm` на **співпраці** і в **модалці нот**; тип `phoneNumber: string` у `OfferCollaborationForm`; листи `emails.ts` / `emailService.ts`; спільний валідатор для **`/api/collaboration`** (сторінка контактів) і **`/api/contact`**.

---

## Проба 2: мінімальна довжина повідомлення (10 → 5)

**Гіпотеза:** повідомлення від **5** символів (після trim на формі) проходить; менше 5 — помилка. **Це експеримент PoC, не AC Дня 2** (у специфікації мінімум лишався 10, поки замовник не погодить інше).

**На момент Проби 2** на формі вже діє обов’язковий телефон (Проба 1).

### Що змінили

Файли з таблиці з позначкою **Проба 2** (рядки 12, 1, 6–9 частково — min message). **`validateContactData.ts`** у diff лише через Пробу 1 (телефон); ліміт повідомлення там уже читає `lengths.MESSAGE_MIN_LENGTH` — після зміни константи сервер відхиляє &lt; 5 **без** edit цього файлу в Пробі 2. **`errors.ts`** — див. примітку в таблиці (не в diff, текст `MESSAGE_ERROR` оновлюється опосередковано).

### Що вводили в браузері

| # | Мова | Сторінка | Ввід (окрім валідних імені/email/телефону/згоди) | Результат |
|---|------|----------|---------------------------------------------------|-----------|
| A | **uk** | `/uk/contacts` | Повідомлення: `Прив` (4 символи) | Помилка мінімальної довжини під полем повідомлення |
| B | **uk** | те саме | `12345` (5 символів) | Валідація повідомлення проходить |
| C | **uk** | те саме | `Привіт` (6 символів) | Проходить (раніше при min 10 також проходило) |
| D | **en** | `/en/contacts` | `Hi` (2) vs `Hello` (5) | Та сама логіка, англійський текст помилки |

### Точний текст помилки (після змін Проби 2)

| Мова | Умова | Текст на екрані |
|------|--------|-----------------|
| **uk** | &lt; 5 символів (після trim на формі) | **Коментар має містити щонайменше 5 символів** (`contactForm.errors.messageMinLength`) |
| **en** | &lt; 5 символів | **The comment must contain at least 5 characters** |
| API (не показується в UI форми) | Коротке повідомлення на сервері | `Message must be a string between 5 and 1000 characters.` (`errors.MESSAGE_ERROR` у `errors.ts`) |

### Тести

**Вузький прогін (після оновлення тестів):** знову `ContactForm` + `validateContactData` — **22/22** (додали/змінили кейси під min 5 і обов’язковий телефон).

**Повний прогін (регресія):**

```bash
npm test -- --coverage=false
```

Tests: **198 passed**, 31 test suites (усі модулі репозиторію; більшість тестів **не** про довжину повідомлення).

**Що було червоним** (після `MESSAGE_MIN_LENGTH: 5` до оновлення тестів):

| Тест | Чому падав |
|------|------------|
| `ContactForm.test.tsx` → `should show errors when incorrect inputs` | У полі було `Привіт` (6 символів): при min **5** рядок `messageMinLength` **не з’являється**, тест ще очікував помилку (мок: «Напишіть кілька слів у повідомленні») |
| `validateContactData.test.ts` → `should return error if message is too short` | Дані `Too short` (**9** символів) при min 5 **валідні** — тест очікував `MESSAGE_ERROR` |
| `validateContactData.test.ts` → `should return no errors for valid input` | Після Проби 1 без `phoneNumber` у payload теж міг падати, якщо ще не оновили |

**Чому стали зеленими:** у негативному сценарії форми — повідомлення **`Прив`** (4 символи); у серверному тесті «too short» — рядок коротший за 5 (наприклад **`Four`**); додано/уточнено кейс **рівно 5** символів; у всіх «valid» payload — `phoneNumber`; оновлені рядки в `uk.json` / `en.json` (5 замість 10).

### Знахідки (2–3)

1. **Єдине число для сервера:** `app/constants/validation.ts` → `MESSAGE_MIN_LENGTH`; `validateContactData.ts` і шаблон `MESSAGE_ERROR` у **`errors.ts`** (файл не в diff) читають цю константу.
2. **До PoC був розсинхрон:** клієнт мав **`.min(10)`** у `ContactForm.tsx`, сервер — **10** з константи; Проба 2 зрівняла клієнт із `lengths.MESSAGE_MIN_LENGTH`.
3. **Побічний ефект:** та сама форма в **співпраці** і **GetNotesModal**; на формі `message` з **`.trim()`**, на сервері довжина рахується **без trim** — можлива розбіжність на кордоні (окреме питання для refinement).

---

## Короткий підсумок

| Проба | Узгодженість з Днем 2 | Браузер (головне) | Вузькі тести | Повний `npm test` |
|-------|------------------------|-------------------|--------------|-------------------|
| 1 | Так (обов’язковий телефон) | Порожній телефон → «Поле обов'язкове…» / en equivalent | 21 passed (2 suites) | — |
| 2 | Ні (min 5 — лише PoC) | `Прив` → помилка; `12345` → ок | 22 passed (2 suites) | 198 passed (31 suites) |

Скріншоти в нотатках не зафіксовано — за потреби додати до `docs/poc-summary-2026-11-24.md`.