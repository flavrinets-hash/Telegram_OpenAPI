# Настройка webhook Telegram Bot API через Nginx: руководство и OpenAPI-спецификация

[![Deploy MkDocs to GitHub Pages](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/deploy.yml/badge.svg)](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/deploy.yml)
[![Markdown Quality Check](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/markdown-lint.yml/badge.svg)](https://github.com/flavrinets-hash/Telegram_OpenAPI/actions/workflows/markdown-lint.yml)

Практическая техническая документация по настройке и безопасной интеграции вебхуков Telegram Bot API через веб-сервер Nginx с сопутствующей спецификацией OpenAPI 3.0.

🔗 **Опубликованная документация:** [https://flavrinets-hash.github.io/Telegram_OpenAPI/](https://flavrinets-hash.github.io/Telegram_OpenAPI/)

**Стек технологий:** MkDocs (Material), OpenAPI 3.0, Nginx, GitHub Actions.

---

## Описание проекта

Проект демонстрирует применение методологии **Docs-as-Code** и принципов фреймворка **Diátaxis** для документирования серверной интеграции с **Telegram Bot API**.

На текущем этапе документация реализует ключевые прикладные модули:

* **Reference:** Спецификация OpenAPI 3.0 с интерактивным Swagger UI для методов управления вебхуком Telegram Bot API (`setWebhook`, `getWebhookInfo`, `deleteWebhook`) и структуры входящих событий `Update`.
* **How-To Guide:** Пошаговое практическое руководство по доставке вебхуков от серверов Telegram в локальный сервис бота через Nginx (SSL/TLS-терминация, валидация секретного токена `X-Telegram-Bot-Api-Secret-Token` и устранение типовых сетевых ошибок).
* *(В развитии)*: Расширение обучающего контура разделами **Tutorial** (интерактивный быстрый старт с нуля) и **Explanation** (концептуальный анализ архитектуры).

<!--
  TODO (Diátaxis Roadmap):
  - Tutorial: Добавить раздел Getting Started (короткий путь «с нуля до работающего webhook» за 10-15 минут).
  - Explanation: Концептуальное руководство (архитектура доставки событий, сравнение Webhook vs Polling, модель безопасности Nginx).
  Запланировано в спринте артефактов (Недели 3-4).
-->

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
