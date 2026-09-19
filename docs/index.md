# Telegram Bot API: Webhook & OpenAPI

Добро пожаловать в проект по настройке и интеграции **Webhook** для **Telegram Bot API** с использованием обратного прокси **Nginx** и спецификации **OpenAPI 3.0**.

---

## Архитектура взаимодействия

При работе в режиме Webhook серверы Telegram самостоятельно доставляют входящие события (сообщения, нажатия inline-кнопок и т.д.) по протоколу HTTPS на ваш веб-сервер. Nginx фильтрует запросы по секретному токену и передаёт их локальному приложению бота.

```mermaid
sequenceDiagram
    autonumber
    actor User as Пользователь
    participant TG as Telegram Bot API
    participant Nginx as Nginx Reverse Proxy
    participant Bot as Сервис бота (127.0.0.1:8000)

    User->>TG: Отправка сообщения в чат
    TG->>Nginx: POST /bot-webhook (HTTPS + Secret Token)
    Note over Nginx: Проверка заголовка<br/>X-Telegram-Bot-Api-Secret-Token
    alt Неверный токен
        Nginx-->>TG: 403 Forbidden
    else Токен корректен
        Nginx->>Bot: Проксирование запроса (HTTP)
        Bot-->>Nginx: 200 OK (Мгновенный ответ)
        Nginx-->>TG: 200 OK
        Bot->>TG: Исходящий ответ (sendMessage)
    end
```

---

## Разделы документации

<div class="grid cards" markdown>

-   :material-server-network:{ .lg .middle } __[Настройка Webhook](webhook.md)__

    ---

    Пошаговое руководство по развёртыванию обратного прокси Nginx, выпуску SSL-сертификатов, регистрации вебхука в Telegram Bot API и устранению неполадок (Troubleshooting).

    [:octicons-arrow-right-24: Перейти к руководству](webhook.md)

-   :material-api:{ .lg .middle } __[Спецификация OpenAPI](openapi.md)__

    ---

    Описание схемы API (OpenAPI 3.0.3) для метода `setWebhook`, поддержка отправки сертификатов (`multipart/form-data`) и структура событий `Update`.

    [:octicons-arrow-right-24: Открыть спецификацию](openapi.md)

</div>

