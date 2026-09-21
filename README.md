# Telegram Bot API Webhook — Docs-as-Code Project

Публичная техническая документация по настройке и безопасной интеграции вебхуков Telegram Bot API через веб-сервер Nginx.

🔗 **Опубликованная документация:** [https://flavrinets-hash.github.io/Telegram_OpenAPI/](https://flavrinets-hash.github.io/Telegram_OpenAPI/)

---

## Описание проекта

Проект демонстрирует применение методологии **Docs-as-Code** и фреймворка **Diátaxis** для документирования серверной интеграции:
* **Reference:** Спецификация OpenAPI 3.0 с интерактивным Swagger UI для ключевых методов работы с вебхуками (`setWebhook`, `getWebhookInfo`, `deleteWebhook`).
* **How-To Guide:** Пошаговое практическое руководство по настройке обратного прокси (Nginx Reverse Proxy), обработке SSL/TLS, валидации секретного токена `X-Telegram-Bot-Api-Secret-Token` и устранению типовых ошибок сети и таймаутов.

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