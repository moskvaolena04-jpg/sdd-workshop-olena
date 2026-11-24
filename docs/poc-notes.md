# Нотатки PoC: що я побачив у запущеному застосунку

## Проба 1: обов'язковий телефон
Що змінив: ContactForm.tsx, validateContactData.ts, validation.ts, errors.ts
Що побачив у браузері: телефон обов'язковий, форма не відправляється без телефону, відображається помилка
Що показали тести: Test Suites: 2 passed, 2 total. Tests:       21 passed, 21 total

## Проба 2: мінімальна довжина повідомлення
Що змінив: ContactForm.tsx, validateContactData.ts, MESSAGE_MIN_LENGTH, errors.ts, MESSAGE_ERROR, 
Що побачив у браузері: поле message має новий min ліміт і пропускає 5+ символів. у випадку менше 5 символів відобаражається оновлена помилка.
Що показали тести: Test Suites: 31 passed, 31 total
Tests:       198 passed, 198 total