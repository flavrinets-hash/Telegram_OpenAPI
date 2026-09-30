# Telegram Webhook via Nginx — Docs-as-Code Project

Практическая техническая документация по настройке и безопасной интеграции вебхуков Telegram Bot API через веб-сервер Nginx с сопутствующей спецификацией OpenAPI 3.0.

🔗 **Опубликованная документация:** [https://flavrinets-hash.github.io/Telegram_OpenAPI/](https://flavrinets-hash.github.io/Telegram_OpenAPI/)

---

## Описание проекта

Проект демонстрирует применение методологии **Docs-as-Code** и фреймворка **Diátaxis** для документирования серверной интеграции с **Telegram Bot API**:

* **Reference:** Спецификация OpenAPI 3.0 с интерактивным Swagger UI для методов управления вебхуком Telegram Bot API (`setWebhook`, `getWebhookInfo`, `deleteWebhook`) и структуры входящих событий `Update`.
* **How-To Guide:** Пошаговое практическое руководство по доставке вебхуков от серверов Telegram в локальный сервис бота через Nginx (SSL/TLS-терминация, валидация секретного токена `X-Telegram-Bot-Api-Secret-Token` и устранение типовых сетевых ошибок).

---

## Стек технологий

* **Спецификация:** OpenAPI 3.0 (YAML)
* **Генератор документации:** MkDocs (тема `Material for MkDocs`)
* **Интерактивная документация API:** Swagger UI
* **Хостинг и деплой:** GitHub Pages
* **Веб-сервер:** Nginx (Reverse Proxy, SSL-терминация, фильтрация заголовков)

---

## Локальный запуск

1. Клонируйте репозиторий:

   ```bash
   git clone https://github.com/flavrinets-hash/Telegram_OpenAPI.git
   cd Telegram_OpenAPI
   ```

2. Установите зависимости:

   ```bash
   pip install mkdocs mkdocs-material
   ```

3. Запустите локальный сервер документации:

   ```bash
   python -m mkdocs serve
   ```

   После запуска документация будет доступна по адресу `http://127.0.0.1:8000`.
