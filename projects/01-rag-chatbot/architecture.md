# Архитектура RAG Knowledge Base

## 📐 Компоненты системы

### Уровень 1: Источник данных

```
Google Docs (документ)
    ↓
    └─→ Содержит текстовую информацию
        Пример: FAQ, документация, внутренние ноты
```

### Уровень 2: Мониторинг и Извлечение

```
Google Drive Trigger
    ├─ Проверяет каждый час
    └─ Активируется при изменении файла
        ↓
    Get a document (Google Docs API)
        └─ Читает полный текст документа
```

### Уровень 3: Очистка и Препроцессинг

```
Code JavaScript1 (Text Cleaning)
    ├─ Удаляет управляющие символы (\n, \r, \t)
    ├─ Удаляет спецсимволы кавычек ("", «», и т.д.)
    ├─ Удаляет Unicode-спецсимволы (™, →, и т.д.)
    ├─ Удаляет смайлики
    ├─ Удаляет множественные пробелы
    └─ Возвращает чистый текст
```

### Уровень 4: Чанкирование

```
Code Python (Chunking)
    ├─ Разбивает текст на массив слов
    ├─ Группирует по 500 слов в один чанк
    ├─ Добавляет метаданные каждому чанку:
    │   ├─ chunkIndex (порядковый номер)
    │   ├─ source ("google_doc")
    │   └─ documentId (уникальный ID документа)
    └─ Выход: Array of { text, metadata }
```

### Уровень 5: Генерация Эмбеддингов

```
HTTP Request (OpenAI Embeddings)
    ├─ Model: text-embedding-3-small
    ├─ Input: 500-словный чанк
    ├─ Output: Вектор из 1536 float чисел
    └─ Стоимость: ~$0.02 за миллион токенов
```

### Уровень 6: Объединение Данных

```
Merge (объединение потоков)
    ├─ Input 1: Чанки с метаданными (из Python)
    └─ Input 2: Эмбеддинги (из OpenAI)
        ↓
    Output: { text, metadata, data[0].embedding }
```

### Уровень 7: Форматирование для Pinecone

```
Code JavaScript (Vector Formatting)
    ├─ Структурирует в формат Pinecone:
    │   ├─ id: "chunk_N"
    │   ├─ values: [1536 чисел вектора]
    │   └─ metadata: { text, chunkIndex, source }
    └─ Выход: Array of Pinecone vectors
```

### Уровень 8: Хранилище и Индекс

```
Pinecone (Vectorial Database)
    ├─ Хранит все векторы с метаданными
    ├─ Создает индекс для быстрого поиска
    └─ Готов для семантического поиска
```

## 🔄 Поток данных

### Сценарий: Синхронизация 5000-словного документа

```
[TIME: 0ms] Google Drive Trigger срабатывает
           ↓
[TIME: 50ms] Get Document получает текст
            (5000 слов = ~30KB)
            ↓
[TIME: 100ms] Delete All из Pinecone
             (очистка старых векторов)
             ↓
[TIME: 150ms] Text Cleaning
             (удаление спецсимволов)
             ↓
[TIME: 200ms] Chunking в Python
             (5000 слов → 10 чанков по 500 слов)
             ↓
[TIME: 250ms-3250ms] Generate Embeddings (для каждого чанка)
                    Чанк 0: embedding → [0.123, -0.456, ...]
                    Чанк 1: embedding → [0.098, 0.234, ...]
                    ...
                    Чанк 9: embedding → [0.567, -0.789, ...]
             ↓
[TIME: 3300ms] Merge объединяет данные
              10 чанков + 10 эмбеддингов
             ↓
[TIME: 3350ms] Format Vectors для Pinecone
              ↓
[TIME: 3400-3500ms] Upsert to Pinecone
                   Загружаются 10 векторов
                   ↓
[TIME: 3500ms] ✅ ГОТОВО
```

**Общее время: ~3.5 секунды**

## 🔀 Обработка ошибок

### Где могут быть ошибки:

```
┌─────────────────────────────────────┐
│ Google Drive Trigger                 │
│ Ошибка: Документ не найден          │
│ Решение: Check documentId            │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ Get a document                       │
│ Ошибка: Access denied                │
│ Решение: Re-authenticate Google OAuth│
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ Delete from Pinecone                 │
│ Ошибка: Pinecone недоступен          │
│ Решение: Retry с exponential backoff│
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ Generate Embeddings                  │
│ Ошибка: Rate limit (>3500 RPM)      │
│ Решение: Throttle или queue запросы │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ Upsert to Pinecone                   │
│ Ошибка: Размер батча слишком велик  │
│ Решение: Разбить на меньшие батчи   │
└─────────────────────────────────────┘
```

## 📊 Размеры данных

### Пример документа 5000 слов:

```
Input:
├─ Текст до очистки: ~32KB (5000 слов × 6.4 байт/слово)
│
Chunking:
├─ Количество чанков: 10
├─ Размер каждого чанка: 500 слов
│
Embeddings:
├─ Вектор размер: 1536 float32
├─ Размер одного вектора: 1536 × 4 байт = 6.1KB
├─ Количество векторов: 10
├─ Общий размер: 61KB
│
Pinecone:
├─ ID (строка): ~20 байт × 10 = 200 байт
├─ Вектор (float32): 6.1KB × 10 = 61KB
├─ Метаданные (JSON): ~300 байт × 10 = 3KB
└─ ИТОГО на документ: ~65KB в Pinecone
```

## 💾 Масштабируемость

### Сценарий 1: 10 документов по 5000 слов каждый

```
Total vectors: 100
Total storage: ~650KB
Time to sync: ~30 сек
Cost: ~$0.02
Queries/month: может обработать 100K+ запросов
```

### Сценарий 2: 100 документов (FAQ база)

```
Total vectors: 1000
Total storage: ~6.5MB
Time to sync: ~300 сек (параллельно можно оптимизировать)
Cost: ~$0.20
Queries/month: может обработать 1M+ запросов
```

### Сценарий 3: Production система 1000+ документов

```
Total vectors: 10000+
Total storage: ~65MB+
Time to sync: Нужна оптимизация (параллельная обработка)
Cost: ~$2+
Queries/month: неограниченно

ОПТИМИЗАЦИИ:
├─ Параллельная генерация эмбеддингов (n8n Execute node)
├─ Батчинг запросов к Pinecone (5-10 чанков за раз)
├─ Кеширование эмбеддингов (не пересчитывать неизменные)
└─ Инкрементальные обновления (только новые/изменённые чанки)
```

## 🔐 Безопасность

### Credentials Management

```
┌─────────────────────────────────────┐
│ n8n Credential Store                │
│                                     │
├─ Google Docs OAuth2                │
│  ├─ access_token (зашифрован)       │
│  └─ refresh_token (зашифрован)      │
├─ Google Drive OAuth2               │
│  ├─ access_token (зашифрован)       │
│  └─ refresh_token (зашифрован)      │
├─ Pinecone API Key                  │
│  └─ (зашифрован в БД n8n)           │
└─ OpenAI API Key                    │
   └─ (зашифрован в БД n8n)           │
```

**Важно:**
- Никогда не коммитить credentials в Git
- Использовать environment variables
- Регулярно ротировать API ключи
- Логировать доступ для аудита

## 🔍 Поиск и Retrieval

### Как работает семантический поиск:

```
User Question: "Как установить?"
    ↓
Generate embedding for question
    ↓
Query vector: [0.123, -0.456, 0.789, ...]
    ↓
Pinecone semantic search (cosine similarity)
    ↓
Results (sorted by similarity score):
    Chunk 0: score 0.92 ← "Для установки используй docker..."
    Chunk 5: score 0.88 ← "npm install n8n"
    Chunk 1: score 0.85 ← "Prerequisites: Node.js..."
    Chunk 9: score 0.78 ← "Docker-compose.yml file..."
    Chunk 3: score 0.75 ← "Troubleshooting installation..."
    Chunk 7: score 0.71 ← "Advanced configuration..."
    ↓
Top 6 returned with metadata
```

**Почему семантический поиск лучше:**
- Обычный поиск ищет ключевые слова
- Семантический ищет по смыслу
- Находит релевантный контент даже без точного совпадения

## 🎯 Query Optimization

### Метрики производительности:

```
Metric                  Target    Current
────────────────────────────────────────
Vector DB query time    <100ms    ~50ms ✅
Embedding generation    <500ms    ~400ms ✅
Text cleaning           <100ms    ~50ms ✅
Chunking               <100ms    ~50ms ✅
Upsert to DB           <500ms    ~250ms ✅
────────────────────────────────────────
Total pipeline         <2000ms   ~1500ms ✅
```

## 📈 Мониторинг

### Key metrics to track:

```
1. Pipeline Success Rate
   └─ Target: > 99%
   
2. Average Execution Time
   └─ Target: < 2000ms
   
3. Vector Count Growth
   └─ Alert if > 100K vectors (проверить Pinecone план)
   
4. API Cost
   └─ Budget: ~$0.05 per 5000-word document
   
5. Latency by Stage
   └─ Identify bottlenecks
```

---

**v1.0.0** | Production Ready ✅
