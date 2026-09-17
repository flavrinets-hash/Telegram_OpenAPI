# Настройка вебхука Telegram Bot API через Nginx

Настройте Nginx в роли обратного прокси (reverse proxy), чтобы принимать входящие HTTPS-запросы от Telegram Bot API и безопасно передавать их в локальный сервис бота.

---

## Предусловия

Перед началом убедитесь, что подготовлены:

- Сервер под управлением Linux с установленным Nginx.
- Права суперпользователя (`root`) или доступ к вызову команд через `sudo`.
- Открытые входящие порты `80` (HTTP) и `443` (HTTPS) в брандмауэре.
- Доменное имя с настроенной A-записью, указывающей на публичный IP-адрес сервера.
- Выпущенный SSL/TLS-сертификат для домена (Telegram доставляет вебхуки только по HTTPS).
- API-токен бота, полученный у [@BotFather](https://t.me/BotFather).
- Запущенный локальный сервис бота (в руководстве используется адрес `http://127.0.0.1:8000`).

---

## 2. Настройка Nginx в режиме reverse proxy

1. Создайте отдельный конфигурационный файл для бота  `sudo nano /etc/nginx/conf.d/telegram-bot.conf`. 
2. Настройте блок `server` для обработки защищённых HTTPS-запросов:

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    # Пути к SSL/TLS сертификатам
    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # Проксирование вебхука к сервису бота
    location /bot-webhook {
        proxy_pass http://127.0.0.1:8000;

        # Передача оригинальных заголовков клиента
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> [!NOTE]
> **Ключевые директивы:**
> - `listen 443 ssl` — приём входящего HTTPS-трафика на стандартном защищённом порту.
> - `server_name` — ваше доменное имя, указанное в DNS.
> - `ssl_certificate` / `ssl_certificate_key` — публичный сертификат (`fullchain.pem`) и приватный ключ (`privkey.pem`).
> - `location /bot-webhook` — эндпоинт вебхука Telegram.
> - `proxy_pass` — адрес и порт запущенного локального сервиса бота.
> - `proxy_set_header` — проброс заголовков с оригинальным IP-адресом клиента и схемой запроса в приложение.

---

## 3. Проверка и применение конфигурации 
1. Проверьте синтаксис конфигурации на наличие ошибок:
   ```bash
   sudo nginx -t
   ```
   *Ожидаемый вывод: `syntax is ok` и `test is successful`.*

2. Перезагрузите Nginx для применения изменений без прерывания соединений:
   ```bash
   sudo systemctl reload nginx
   ```

---

## 4. Регистрация вебхука через Telegram Bot API 

1. Зарегистрируйте адрес вебхука в Telegram Bot API, выполнив запрос через `curl` в терминале:

```bash
curl --location 'https://api.telegram.org/bot<ВАШ_ТОКЕН>/setWebhook' \
--header 'Content-Type: application/json' \
--data '{
  "url": "https://your-domain.com/bot-webhook",
  "max_connections": 40,
  "allowed_updates": [
    "message",
    "callback_query"
  ],
  "drop_pending_updates": false,
  "secret_token": "super_secret_token_123"
}'
```

> [!NOTE]
> Префикс `bot` в URL обязателен, а сам токен подставляется вплотную к нему без пробелов (например: `https://api.telegram.org/bot123456:ABC-DEF.../setWebhook`).

*Примеры ответов Telegram Bot API:*

- **Пример успешного ответа (HTTP 200):**
  ```json
  {
    "ok": true,
    "result": true,
    "description": "Webhook was set"
  }
  ```

- **Пример ответа с ошибкой (HTTP 400 / 404):**  
  *(например, если URL не защищён HTTPS, домен недоступен или токен неверен)*
  ```json
  {
    "ok": false,
    "error_code": 400,
    "description": "Bad Request: HTTPS url must be provided for webhook"
  }
  ```

> **Ключевые параметры:**
> - `url` — обязательный параметр. Полный публичный HTTPS-адрес вашего вебхука, настроенный в Nginx.
> - `max_connections` — максимальное количество одновременных подключений от Telegram к вашему боту (от 1 до 100, по умолчанию 40).
> - `allowed_updates` — список типов обновлений, на которые подписывается бот (например, `message`, `callback_query`). Если параметр не передан, бот получает все поддерживаемые события, тоже самое происходит при передаче пустого массива. [Все поддерживаемые типы обновлений](https://core.telegram.org/bots/api#update)
> - `drop_pending_updates` — если передать `true`, Telegram сбросит все накопившиеся необработанные сообщения и не станет отправлять их боту при старте.
> - `secret_token` — секретная строка (1–256 символов, `A-Z`, `a-z`, `0-9`, `_`, `-`) для защиты. Telegram будет передавать её в заголовке `X-Telegram-Bot-Api-Secret-Token` при каждом запросе.

---

2. Проверьте статус подключения вебхука через метод `getWebhookInfo`:

```bash
curl --location 'https://api.telegram.org/bot<ВАШ_ТОКЕН>/getWebhookInfo'
```

*Шаблон успешного ответа сервера:*
```json
{
  "ok": true,
  "result": {
    "url": "string",
    "has_custom_certificate": true,
    "pending_update_count": 0,
    "ip_address": "197.0.2.1",
    "last_error_date": 0,
    "last_error_message": "string",
    "last_synchronization_error_date": 0,
    "max_connections": 0,
    "allowed_updates": [
      "message",
      "edited_channel_post",
      "callback_query"
    ]
  }
}
```
---

### Передача сертификата при использовании Self-Signed (самоподписанного) SSL

В этом случае файл **публичного** сертификата необходимо явно передать в Telegram через `multipart/form-data`:

```bash
curl --location 'https://api.telegram.org/bot<ВАШ_ТОКЕН>/setWebhook' \
--form 'url="https://your-domain.com/bot-webhook"' \
--form 'certificate=@"/path/to/YOUR_PUBLIC_KEY.pem"'
```

> [!WARNING]
> - Символ `@` перед путём к файлу обязателен — он указывает `curl` прикрепить и отправить файл.
> - Передаётся **только публичный сертификат** (`.pem` / `.crt`). Приватный ключ (`privkey.pem`) передавать категорически нельзя!
> - При успешной загрузке в методе `getWebhookInfo` параметр `has_custom_certificate` примет значение `true`.