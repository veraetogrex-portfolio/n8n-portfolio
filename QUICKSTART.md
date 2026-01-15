# 🚀 Быстрый старт

Запусти первый проект за 15 минут.

## 1️⃣ Установка n8n (5 мин)

### Docker (рекомендуется)
```bash
docker run -it -p 5678:5678 n8nio/n8n
# Открой http://localhost:5678
```

### Или локально
```bash
npm install -g n8n
n8n start
```

## 2️⃣ Получение API ключей (5 мин)

### OpenAI API (для embeddings и LLM)
```
1. https://platform.openai.com/api-keys
2. Create new secret key
3. Скопируй: sk-proj-...
```

### Pinecone (вектор БД)
```
1. https://www.pinecone.io
2. Create free project
3. Create index: knowledge-base (1536 dimensions)
4. Скопируй API key и host
```

### Telegram Bot (опционально)
```
1. Telegram → @BotFather
2. /newbot
3. Скопируй token: 123456:ABC-...
```

## 3️⃣ Подготовка конфигурации (2 мин)

```bash
# Скопируй пример
cp .env.example .env

# Открой .env и заполни:
OPENAI_API_KEY=sk-proj-...
PINECONE_API_KEY=pc-...
PINECONE_INDEX=knowledge-base
PINECONE_HOST=knowledge-base-xxx.svc.aped-xxxx.pinecone.io
TELEGRAM_BOT_TOKEN=123456:ABC-...
```

⚠️ **ВАЖНО:** Никогда не коммитай .env файл!

## 4️⃣ Импорт и запуск (3 мин)

### В n8n:
```
1. Workflows → Import
2. Выбери: projects/01-rag-chatbot/workflow.json
3. Нажми Import
```

### Подключение credentials:
```
1. Google Docs node → нажми "Authenticate"
2. Выбери Google Account
3. Дай доступ к Google Docs
(аналогично для остальных)
```

### Запуск:
```
1. Нажми "Execute Workflow"
2. Смотри логи справа
3. Готово! 🎉
```

---

## 📍 Статусы проектов

| Проект | Статус | Файл |
|--------|--------|------|
| 1. RAG Chatbot | ✅ Готово | `projects/01-rag-chatbot/` |
| 2. Support Auto | 🔄 В работе | `projects/02-support-automation/` |
| 3. Payment Integration | 📅 Планируется | `projects/03-payment-integration/` |
| 4. Data Pipeline | 📅 Планируется | `projects/04-data-pipeline/` |
| 5. Notification System | 📅 Планируется | `projects/05-notification-system/` |

---

## 🆘 Troubleshooting

### ❌ "documentId not found"
```
✅ Решение: В Google Drive Trigger выбери свой документ
```

### ❌ "Pinecone API error"
```
✅ Решение: Проверь API key и host в .env
```

### ❌ "OpenAI rate limit"
```
✅ Решение: Подожди минуту и повтори
```

### ❌ "Telegram bot не отвечает"
```
✅ Решение: Убедись что webhook зарегистрирован у @BotFather
```

---

## 📚 Дальше

- 📖 Полный README: `README.md`
- 🏗️ Архитектура: `projects/01-rag-chatbot/architecture.md`
- 📋 Структура: `STRUCTURE.md`
- 💼 Для интервью: `PORTFOLIO_GUIDE.md`

---

**Готово к запуску? 🚀 Начни с проекта #1!**
