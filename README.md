# Настройка webhook Telegram Bot API через Nginx: руководство и OpenAPI-спецификация

[![Deploy MkDocs to GitHub Pages](https://img.shields.io/github/actions/workflow/status/flavrinets-hash/Telegram_OpenAPI/deploy.yml?style=flat-square&logo=github&label=Deploy%20Docs)](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/deploy.yml)
[![Markdown Quality Check](https://img.shields.io/github/actions/workflow/status/flavrinets-hash/Telegram_OpenAPI/markdown-lint.yml?style=flat-square&logo=markdown&label=Markdown%20Lint)](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/markdown-lint.yml)

Практическая техническая документация по настройке и безопасной интеграции вебхуков Telegram Bot API через веб-сервер Nginx с сопутствующей спецификацией OpenAPI 3.0.

🔗 **Опубликованная документация:** [https://flavrinets-hash.github.io/Telegram_OpenAPI/](https://flavrinets-hash.github.io/Telegram_OpenAPI/)

**Стек технологий:** MkDocs (Material), OpenAPI 3.0, Postman, Nginx, GitHub Actions.

---

## Описание проекта

Проект демонстрирует применение методологии **Docs-as-Code** и принципов фреймворка **Diátaxis** для документирования серверной интеграции с **Telegram Bot API**.

На текущем этапе документация реализует ключевые прикладные модули:

* **Reference:** Спецификация OpenAPI 3.0 с интерактивным Swagger UI для методов управления вебхуком Telegram Bot API (`setWebhook`, `getWebhookInfo`, `deleteWebhook`) и структуры входящих событий `Update`.
* **How-To Guide:** Пошаговое практическое руководство по доставке вебхуков от серверов Telegram в локальный сервис бота через Nginx (SSL/TLS-терминация, валидация секретного токена `X-Telegram-Bot-Api-Secret-Token` и устранение типовых сетевых ошибок).

---

## Локальный запуск

1. Клонируйте репозиторий:

   ```bash
   git clone https://github.com/flavrinets-hash/Telegram_OpenAPI.git
   cd Telegram_OpenAPI
   ```

2. Установите зависимости:

   ```bash
   pip install -r requirements.txt
   ```

3. Запустите локальный сервер документации:

   ```bash
   python -m mkdocs serve
   ```

   После запуска документация будет доступна по адресу `http://127.0.0.1:8000`.

---

## Тестирование через Postman

В директории [`docs/assets/`](docs/assets/) подготовлены артефакты для тестирования жизненного цикла вебхука:

* [`docs/assets/telegram_webhook.postman_collection.json`](docs/assets/telegram_webhook.postman_collection.json) — коллекция запросов (`setWebhook`, `getWebhookInfo`, `deleteWebhook`) с преднастроенными тестами и переменными окружения.
* [`docs/assets/telegram_webhook.postman_test_run.json`](docs/assets/telegram_webhook.postman_test_run.json) — сохранённый отчёт успешного выполнения сквозного E2E-сценария.

Коллекция готова к импорту в Postman. Для запуска сценария достаточно указать токен бота в переменной окружения `token` (в поле *Current Value*).
