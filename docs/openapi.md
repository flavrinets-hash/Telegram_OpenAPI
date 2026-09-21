# Спецификация OpenAPI (Telegram Bot API — Webhook)

Спецификация описывает контракты взаимодействия с методами Telegram Bot API для управления вебхуком, а также формат входящих событий (обратных вызовов `callbacks` / `Update`).

---

## Интерактивная документация

Вы можете просмотреть интерактивную документацию через Swagger UI или скачать файл спецификации:

<div class="grid cards" markdown>

-   :material-play-circle-outline:{ .lg .middle } __[Открыть Swagger UI](swagger.html)__

    ---

    Интерактивный просмотр эндпоинтов, моделей данных и примеров запросов в интерфейсе Swagger.

    [Открыть интерактивный интерфейс :octicons-arrow-right-24:](swagger.html){ .md-button .md-button--primary target="_blank" }

-   :material-file-code-outline:{ .lg .middle } __[Скачать OpenAPI YAML](assets/openapi_telegram.yaml)__

    ---

    Исходный файл спецификации в формате OpenAPI 3.0.3 для импорта в Postman, Insomnia или генераторы кода.

    [Скачать схему :octicons-download-24:](assets/openapi_telegram.yaml){ .md-button download="openapi_telegram.yaml" }

</div>

---

## Описание эндпоинтов

### 1. `POST /bot{token}/setWebhook`
Регистрация публичного HTTPS URL для приёма обновлений от Telegram Bot API.

- **Параметры пути:**
    - `token` (`string`, обязательный) — секретный токен вашего бота.
- **Тело запроса (`Content-Type`):**
    - `application/json` (`BaseWebhook`) — стандартная установка без собственного SSL-сертификата.
    - `multipart/form-data` (`WebhookWithCertificate`) — установка с передачей файла самоподписанного сертификата (`certificate`).
- **Callbacks (`onUpdate`):**
    - Telegram отправляет событие `Update` на указанный в теле запроса `url` методом `POST`.

### 2. `GET /bot{token}/getWebhookInfo`
Получение текущего статуса вебхука, адреса, даты последней ошибки и количества ожидающих обновлений.

- **Параметры пути:**
    - `token` (`string`, обязательный) — токен бота.
- **Ответ (`200 OK`):**
    - `WebhookInfo` со сведениями о состоянии доставки и синхронизации.

### 3. `POST /bot{token}/deleteWebhook`
Удаление текущей интеграции с вебхуком и опциональный сброс очереди сообщений.

- **Query-параметры:**
    - `drop_pending_updates` (`boolean`, опционально) — сбросить все ожидающие обновления.

---

## Модели данных

Спецификация описывает ключевые схемы данных Telegram Bot API:

* **`Update`** — входящее событие от Telegram (`update_id`, входящее или изменённое сообщение, пост в канале).
* **`Message`** — сообщение (`message_id`, автор `from`, чат `chat`, дата отправки и текст).
* **`Chat`** — данные чата (тип `private` / `group` / `supergroup` / `channel`, ID и название).
* **`User`** — профиль пользователя или бота (ID, имя, username, статус бота).
* **`WebhookInfo`** — статус вебхука, адрес доставки, дата последней ошибки и число ожидающих сообщений.

---

## Исходная спецификация (OpenAPI 3.0.3)

??? example "Показать YAML-спецификацию целиком"

    ```yaml
    --8<-- "assets/openapi_telegram.yaml"
    ```
