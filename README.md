# AeroGrid 3.0 — Вітрова електростанція з моніторингом швидкості вітру

Мобільний застосунок на Android для диспетчерського SCADA-моніторингу вітрової електростанції, прогнозування генерації через REST API, токен-базованої автентифікації та відмовостійкої обробки мережевих збоїв. Продовження проєкту ЛР №1 та ЛР №2.

## Загальна інформація

* **Дисципліна:** Розробка програмного забезпечення мобільних пристроїв
* **Університет:** Національний технічний університет України «Київський політехнічний інститут імені Ігоря Сікорського»
* **Кафедра:** Кафедра інженерії програмного забезпечення в енергетиці
* **Практична робота:** №3 (практичні заняття №9–12)
* **Варіант:** №2 — «Вітрова електростанція з моніторингом швидкості вітру»

## Нові можливості ЛР №3 (ПЗ №9–12)

### ПЗ №9 — Патерни Repository та Data Source
* Чітке розділення доступу до даних на `LocalWindDataSource` (таблиця `wind_forecasts_cache` у Room SQLite v4, міграції `2 → 3 → 4` із каталогом реальних ВЕС України) та `RemoteWindDataSource` (Retrofit2).
* `WindForecastRepository`: TTL-кешування (5 хвилин), примусове оновлення (`forceRefresh`), автоматичний **fallback на локальний кеш** при відсутності інтернету або збої API, повний CRUD прогнозів, фільтрація, сортування (`DATE_ASC`, `POWER_DESC`, `POWER_ASC`, `WIND_DESC`) та бізнес-агрегація метрик парку.

### ПЗ №10 — REST API із Retrofit2 та Gson
* `WindApiService` з мультиендпоінтним контрактом: `/stations`, `/forecasts`, `/stations/{id}/forecasts`, `/stations/{id}/forecasts/stats`, `/stations/{id}/forecasts/generate/{days}`, `/forecast/today`, `/forecast/error/{code}`.
* DTO-моделі з `@SerializedName` (`RemoteWindStationDto`, `WindForecastDto`, `WindForecastStatsDto`).
* Вбудований `MockWindBackendInterceptor` у ланцюжку `OkHttpClient`, що забезпечує повну автономну роботу через реальний стек Retrofit → Gson → OkHttp (із перемикачем на зовнішній сервер `http://10.0.2.2:5156/`).

### ПЗ №11 — JWT-автентифікація, DataStore та RBAC
* Вхід (`POST /login`) та реєстрація (`POST /register`) через REST API із випуском підписаного JWT-токена (`Header.Payload.Signature`) у `JwtUtils`.
* Постійне збереження токена та профілю в `AuthDataStore` (`Preferences DataStore`) і автоматична підстановка заголовка `Authorization: Bearer <token>` через `AuthInterceptor`.
* Перевірка терміну дії токена, імітація протермінування, перевірка доступу без токена (`HTTP 401`) та адаптація інтерфейсу під ролі (`ADMIN` — повна адмін-панель із `/admin-only` та створенням ВЕС, `USER` — приховування адмін-елементів та обробка `HTTP 403`).
* Демо-акаунти для швидкої перевірки ролей:
  * **ADMIN:** `admin@windpower.kpi.ua` / `Admin2026`
  * **USER:** `operator@windpower.kpi.ua` / `User2026`

### ПЗ №12 — Обробка мережевих помилок та Retry
* Перевірка підключення до Інтернету через `ConnectivityManager` / `NetworkCapabilities` (+ перемикач імітації офлайн-режиму для тестування).
* Централізований `NetworkErrorHandler` і `CoroutineExceptionHandler` з диференціацією кодів `400`, `401`, `403`, `404`, `500`, `503` та мережевих таймаутів.
* Автоматичний `retry` (до 3 спроб) для тимчасових збоїв (`500`, `503`, таймаути) з демонстрацією відновлення на запиті `/forecast/today?flaky=true`.
* Реєстрація усіх помилок у `Logcat` та локальний файл `error_log.txt` (`FileErrorLogSink`) із переглядом та очищенням журналу прямо в UI і сповіщеннями через `Snackbar`.

## Збережена функціональність ЛР №1–2
* Single-Activity + `NavHost` на 9 екранів (Моніторинг з Canvas-турбіною, Телеметрія MVVM, Прогноз з `{userName}` та `{turbinesCount}`, База даних Room, Диспетчер, Парк турбін CRUD, Аеродинамічний калькулятор Беца, Реєстрація ВЕС).

## Технології
* **Мова:** Kotlin 2.2.10 (JVM target 11)
* **UI:** Jetpack Compose (BOM 2026.02.01), Material Design 3, темна тема Eco-Tech
* **Мережа:** Retrofit 2.11.0, Converter-Gson 2.11.0, OkHttp 4.12.0, Gson 2.11.0
* **Збереження стану та БД:** Room 2.7.2 (v4 + KSP, 9 реальних ВЕС України), Jetpack Preferences DataStore 1.1.7
* **Тестування:** JUnit 4 (`Lab3NetworkAndRepositoryTest`, `ModulesTest`, `DispatcherAuthRepositoryTest`, `WindPowerCalculatorTest`)

## Збірка та запуск

```bash
./gradlew testDebugUnitTest
./gradlew assembleDebug
```
