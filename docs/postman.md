# Тестирование API в Postman

Для проверки интеграции и валидации контракта подготовлена коллекция запросов **Postman**, созданная на основе спецификации OpenAPI 3.0. Коллекция реализует сквозной сценарий тестирования жизненного цикла вебхука Telegram Bot API.

---

## Артефакты для загрузки

Вы можете скачать файлы коллекции и отчёта о прогоне для локального использования:

<div class="grid cards" markdown>

-   :material-cloud-download-outline:{ .lg .middle } __[Скачать коллекцию Postman](assets/telegram_webhook.postman_collection.json)__

    ---

    Готовая коллекция запросов (`v2.1`) с методами управления вебхуком, переменными окружения и встроенными тестами.

    [Скачать коллекцию :octicons-download-24:](assets/telegram_webhook.postman_collection.json){ .md-button .md-button--primary download="telegram_webhook.postman_collection.json" }

-   :material-check-circle-outline:{ .lg .middle } __[Скачать отчёт о прогоне](assets/telegram_webhook.postman_test_run.json)__

    ---

    Журнал выполнения тестов в Postman Collection Runner (100% Passed, статус-коды 200 OK, валидация полей).

    [Скачать отчёт :octicons-download-24:](assets/telegram_webhook.postman_test_run.json){ .md-button download="telegram_webhook.postman_test_run.json" }

</div>

---

## Переменные коллекции (Variables)

Для безопасности и гибкости все запросы параметризованы через переменные коллекции:

| Переменная | Описание | Пример значения |
| :--- | :--- | :--- |
| `baseUrl` | Базовый URL серверов Telegram Bot API | `https://api.telegram.org` |
| `token` | Секретный токен авторизации бота | `123456789:ABCdef...` |

!!! warning "Безопасность токена бота"
    При работе с коллекцией указывайте токен **исключительно в колонке Current Value**. Значение в колонке *Initial Value* экспортируется в файлы и репозитории — оставляйте его пустым или заполненным плейсхолдером (`YOUR_BOT_TOKEN`), чтобы предотвратить компрометацию секретов.

---

## Сквозной E2E-сценарий (Collection Runner)

Коллекция спроектирована для автоматизированного последовательного выполнения в **Collection Runner**:

```text
1. setWebhook ──> 2. getWebhookInfo ──> 3. deleteWebhook
  (Регистрация)        (Проверка)           (Очистка)
```

1. **`POST /bot{token}/setWebhook`** — регистрирует публичный HTTPS-адрес сервера Nginx и задаёт `secret_token` для валидации заголовка `X-Telegram-Bot-Api-Secret-Token`.
2. **`GET /bot{token}/getWebhookInfo`** — запрашивает актуальный статус интеграции у Telegram, проверяя привязку URL и отсутствие ошибок синхронизации.
3. **`POST /bot{token}/deleteWebhook`** — удаляет зарегистрированный вебхук, возвращая бота в исходное состояние и гарантируя идемпотентность тестового прогона.

---

## Автоматические проверки (Tests)

К запросам привязаны тестовые скрипты на JavaScript, выполняемые средой Postman Sandbox при каждом вызове:

```javascript
// 1. Проверка успешного HTTP-статуса
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2. Проверка формата тела ответа
pm.test("Response is JSON", function () {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});

// 3. Проверка признака успеха в модели ответа Telegram
pm.test("Response ok is true", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.ok).to.eql(true);
});
```

### Результаты валидации

Сводка тестового прогона (`postman_test_run.json`):

| Метрика | Значение | Описание |
| :--- | :--- | :--- |
| **Статус** | `Finished` | Все запросы цепочки выполнены |
| **Успешных проверок** | `5` | Все условия `pm.test` пройдены успешно |
| **Ошибок (Failed)** | `0` | Отсутствуют сбои в assertions |
| **Коды ответов** | `200 OK` | Все вызовы приняты Telegram API |
