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