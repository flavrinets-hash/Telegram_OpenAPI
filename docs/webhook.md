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

## 1. Настройка Nginx в режиме reverse proxy

1. Создайте отдельный конфигурационный файл для бота:
   ```bash
   sudo nano /etc/nginx/conf.d/telegram-bot.conf
   ```
2. Настройте блок `server` для обработки защищённых HTTPS-запросов и фильтрации трафика:

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    # Пути к SSL/TLS сертификатам
    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # Проксирование вебхука к сервису бота
    location /bot-webhook {
        # Отклонять запросы без правильного секретного токена
        if ($http_x_telegram_bot_api_secret_token != "super_secret_token_123") {
            return 403;
        }

        proxy_pass http://127.0.0.1:8000;

        # Передача оригинальных заголовков клиента
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

!!! note "Ключевые директивы конфигурации"
    - `listen 443 ssl` — приём входящего HTTPS-трафика на стандартном защищённом порту.
    - `server_name` — ваше доменное имя, указанное в DNS.
    - `ssl_certificate` / `ssl_certificate_key` — публичный сертификат (`fullchain.pem`) и приватный ключ (`privkey.pem`).
    - `location /bot-webhook` — эндпоинт вебхука Telegram.
    - `proxy_pass` — адрес и порт запущенного локального сервиса бота.
    - `proxy_set_header` — проброс заголовков с оригинальным IP-адресом клиента и схемой запроса в приложение.

### Защита вебхука на уровне Nginx через проверку переменной

Ограничьте доступ для спамеров и неавторизованных запросов до того, как они дойдут до вашего бота. Любые входящие HTTP-заголовки от клиента (Telegram) автоматически преобразуются во внутренние переменные Nginx по следующему правилу:

- В начале добавляется префикс `$http_`.
- Имя заголовка переводится в нижний регистр.
- Все дефисы `-` заменяются на знаки подчёркивания `_`.

Например, заголовок секретного токена `X-Telegram-Bot-Api-Secret-Token` становится переменной:
```nginx
$http_x_telegram_bot_api_secret_token
```

Условие `if ($http_x_telegram_bot_api_secret_token != "super_secret_token_123")` отклоняет любые запросы без правильного секретного токена со статусом `403 Forbidden`.

---

## 2. Проверка и применение конфигурации

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

## 3. Регистрация вебхука через Telegram Bot API

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

!!! note "Обратите внимание на префикс"
    Префикс `bot` в URL обязателен, а сам токен подставляется вплотную к нему без пробелов (например: `https://api.telegram.org/bot123456:ABC-DEF.../setWebhook`).

**Примеры ответов Telegram Bot API:**

=== "Успешный ответ (HTTP 200)"

    ```json
    {
      "ok": true,
      "result": true,
      "description": "Webhook was set"
    }
    ```

=== "Ошибка (HTTP 400 / 404)"

    ```json
    {
      "ok": false,
      "error_code": 400,
      "description": "Bad Request: HTTPS url must be provided for webhook"
    }
    ```

!!! info "Ключевые параметры запроса"
    - `url` — **обязательный параметр**. Полный публичный HTTPS-адрес вашего вебхука, настроенный в Nginx.
    - `max_connections` — максимальное количество одновременных подключений от Telegram к вашему боту (от 1 до 100, по умолчанию 40).
    - `allowed_updates` — список типов обновлений, на которые подписывается бот (например, `message`, `callback_query`). Если параметр не передан или передан пустой массив, бот получает все поддерживаемые события. [Все поддерживаемые типы обновлений](https://core.telegram.org/bots/api#update).
    - `drop_pending_updates` — если передать `true`, Telegram сбросит все накопившиеся необработанные сообщения и не станет отправлять их боту при старте.
    - `secret_token` — секретная строка (1–256 символов, `A-Z`, `a-z`, `0-9`, `_`, `-`) для защиты. Telegram будет передавать её в заголовке `X-Telegram-Bot-Api-Secret-Token` при каждом запросе.

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
    "url": "https://your-domain.com/bot-webhook",
    "has_custom_certificate": true,
    "pending_update_count": 0,
    "ip_address": "197.0.2.1",
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

### Передача сертификата при использовании Self-Signed SSL

В этом случае файл **публичного** сертификата необходимо явно передать в Telegram через `multipart/form-data`:

```bash
curl --location 'https://api.telegram.org/bot<ВАШ_ТОКЕН>/setWebhook' \
--form 'url="https://your-domain.com/bot-webhook"' \
--form 'certificate=@"/path/to/YOUR_PUBLIC_KEY.pem"'
```

!!! warning "Внимание при отправке сертификата"
    - Символ `@` перед путём к файлу обязателен — он указывает `curl` прикрепить и отправить файл.
    - Передаётся **только публичный сертификат** (`.pem` / `.crt`). Приватный ключ (`privkey.pem`) передавать категорически нельзя!
    - При успешной загрузке в методе `getWebhookInfo` параметр `has_custom_certificate` примет значение `true`.

---

## 4. Устранение неполадок (Troubleshooting)

Если бот не реагирует на входящие сообщения после настройки вебхука, выполните следующие шаги для локализации и устранения проблемы.

### 1. Проверка статуса вебхука через Telegram API

Первым шагом запросите диагностическую информацию у серверов Telegram:

```bash
curl --location 'https://api.telegram.org/bot<ВАШ_ТОКЕН>/getWebhookInfo'
```

В ответе обратите внимание на поля `last_error_date` и `last_error_message`:

???+ failure "`Connection timed out` / `Connection refused`"
    **Telegram не может связаться с вашим сервером.**
    
    - Убедитесь, что служба Nginx активна: `sudo systemctl status nginx`.
    - Проверьте настройки брандмауэра и убедитесь, что порт 443 открыт: `sudo ufw status` (для открытия: `sudo ufw allow 443/tcp`).
    - В облачных хостингах проверьте правила Firewall / Security Groups в веб-панели управления.

???+ failure "`SSL error: self signed certificate` / `certificate verify failed`"
    **Ошибка валидации TLS-сертификата.**
    
    - Убедитесь, что в директиве `ssl_certificate` указан файл полной цепочки с промежуточными сертификатами `fullchain.pem`, а не изолированный `cert.pem`.
    - Убедитесь, что домен в DNS-записи и в запросе `setWebhook` совпадает с доменом, для которого выпущен сертификат.

???+ failure "`Wrong response code (502 Bad Gateway)`"
    **Nginx успешно принял запрос от Telegram, но локальное приложение бота недоступно.**
    
    - Убедитесь, что процесс бота запущен и слушает указанный порт: `sudo ss -tulpn | grep 8000`.
    - Проверьте журнал ошибок самого приложения бота.

???+ failure "`Wrong response code (403 Forbidden)`"
    **Nginx отклонил запрос из-за непройденной проверки секретного токена.**
    
    - Проверьте, совпадает ли токен в условии `if ($http_x_telegram_bot_api_secret_token != "...")` со значением `secret_token`, переданным при вызове `setWebhook`.

???+ failure "`Wrong response code (504 Gateway Timeout)`"
    **Локальное приложение бота не успело ответить вовремя** (по умолчанию Nginx ожидает ответ 60 секунд).
    
    В логах Nginx это фиксируется как `upstream timed out` в `error.log` или статус `504` / `499` в `access.log`. Не получив ответ `200 OK`, Telegram расценивает доставку как сбой и начинает циклично слать запрос заново.
    
    - Мгновенно возвращайте статус `HTTP 200 OK` при получении вебхука, не дожидаясь окончания долгой обработки, и выносите длительные операции в фоновые задачи (`asyncio.create_task`, `BackgroundTasks`, `Celery`).
    - Отправляйте результат отдельным исходящим запросом через метод Bot API ([`bot.send_message`](https://core.telegram.org/bots/api#sendmessage)), а не внутри ответа на вебхук.
    - Не пытайтесь решить это увеличением `proxy_read_timeout` в Nginx: у серверов Telegram действует собственный таймаут ожидания, после которого соединение всё равно будет разорвано со стороны Telegram.

---

### 2. Анализ журналов (логов) Nginx

Для отслеживания входящих запросов и ошибок соединения обратитесь к системным журналам Nginx в режиме реального времени:

```bash
# Журнал входящих запросов (проверка факта поступления запроса от Telegram)
sudo tail -f /var/log/nginx/access.log

# Журнал ошибок (сбои proxy_pass, проблемы с сокетами или сертификатами)
sudo tail -f /var/log/nginx/error.log
```
