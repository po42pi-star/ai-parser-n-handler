# Отчёт по задаче: Обработка отзывов на сайте с использованием AI

## 1. Архитектура проекта и путь данных

Система состоит из трёх основных компонентов: веб-приложения (app_test), которое отображает отзывы пользователям, AI-обработчика (ai_handler), который автоматически генерирует ответы на отзывы, и базы данных PostgreSQL для хранения информации. Когда пользователь оставляет новый отзыв на сайте, он сохраняется в базу данных и становится доступным через REST API. Worker-сервис периодически опрашивает сайт на наличие новых отзывов, анализирует их тональность и генерирует ответ с помощью OpenAI API. Сформированный ответ создаётся как отдельный комментарий-ответ от имени AI-помощника, а исходный отзыв помечается как обработанный. При настройке Telegram-уведомлений система также отправляет алерт в чат о новом отзыве.

## 2. Файлы проекта и их назначение

### Веб-приложение (app_test/)
| Файл | Назначение |
|------|------------|
| `main.py` | FastAPI приложение, маршруты и запуск сервера |
| `api/routes.py` | API эндпоинты для получения и создания отзывов |
| `config.py` | Конфигурация приложения (подключение к БД, токен worker) |
| `models/review.py` | SQLAlchemy модель отзыва |
| `db/session.py` | Сессия базы данных |
| `Dockerfile`, `docker-compose.yml` | Контейнеризация |

### AI-обработчик (ai_handler/)
| Файл | Назначение |
|------|------------|
| `worker.py` | Точка входа, главный цикл опроса сайта |
| `client.py` | HTTP-клиент для взаимодействия с сайтом (fetch/create/update reviews) |
| `processor.py` | Логика обработки: анализ тональности, генерация ответа через OpenAI |
| `config.py` | Конфигурация (API ключи, URL, токены) |
| `state.py` | Хранение состояния обработанных отзывов |
| `telegram_bot.py` | Отправка уведомлений в Telegram |
| `models.py` | Pydantic модели данных (RemoteReview, ReviewCreatePayload и т.д.) |

## 3. Взаимодействие client.py, processor.py и worker.py

```
┌─────────────────────────────────────────────────────────────────┐
│                           worker.py                              │
│  (главный цикл: poll → fetch → process → create/update)         │
└─────────────────────────────────────────────────────────────────┘
              │                    │                    │
              ▼                    ▼                    ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│     client.py       │  │    processor.py     │  │    telegram_bot.py  │
│                     │  │                     │  │                     │
│ • check_site()      │  │ • detect_tone()     │  │ • send_notification │
│ • fetch_reviews()   │  │ • generate_response │  │                     │
│ • create_review()   │  │   (OpenAI API)      │  │                     │
│ • update_review()   │  │ • build_fallback()  │  │                     │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
              │                    │
              ▼                    ▼
        ┌─────────────────────────────────────┐
        │           REST API сайта            │
        │      (GET/POST/PATCH /api/reviews)  │
        └─────────────────────────────────────┘
```

1. **worker.py** — главный оркестратор. Запускается в бесконечном цикле с интервалом `WORKER_POLL_INTERVAL`. На каждом шаге:
   - Проверяет доступность сайта через `client.check_site()`
   - Получает список новых отзывов через `client.fetch_new_reviews()`
   - Для каждого нового отзыва вызывает `processor.generate_response()`
   - Создаёт ответ через `client.create_review()`
   - Обновляет исходный отзыв через `client.update_review()`

2. **client.py** — HTTP-клиент. Инкапсулирует все вызовы к REST API сайта:
   - `fetch_reviews()` — получает все отзывы
   - `create_review()` — создаёт ответ-отзыв
   - `update_review()` — обновляет статус отзыва

3. **processor.py** — бизнес-логика:
   - `detect_tone()` — определяет позитив/негатив/нейтр по ключевым словам
   - `generate_response()` — генерирует ответ через OpenAI API или fallback-шаблон

## 4. Файл состояния (state file) и предотвращение повторной обработки

### Зачем нужен:
Файл состояния (`state_file_path` по умолчанию `data/state.json`) хранит идентификаторы уже обработанных отзывов и уведомлений. Без него при перезапуске worker мог бы повторно обработать те же отзывы, создав дубликаты ответов.

### Как работает:
```python
# state.py
processed_reviews: set[int] = set()  # ID обработанных отзывов
processed_notifications: set[int] = set()  # ID обработанных уведомлений

def is_review_processed(review_id: int) -> bool:
    return review_id in state.processed_reviews

def mark_review_as_processed(review_id: int):
    state.processed_reviews.add(review_id)
    save_state()  # Сохраняем в JSON файл
```

### Процесс проверки:
1. Worker получает список всех отзывов с сайта
2. Для каждого отзыва проверяет `state.is_review_processed(review.id)`
3. Если отзыв уже обработан — пропускает его
4. Если отзыв новый — обрабатывает и вызывает `state.mark_review_as_processed(review.id)`
5. Состояние сохраняется в `data/state.json` после каждой операции

### Формат state.json:
```json
{
  "processed_reviews": [1, 2, 3, 5, 7],
  "processed_notifications": [1, 2, 3],
  "last_updated": "2025-01-15T10:30:00Z"
}
```

Это гарантирует, что даже при перезапуске контейнера worker продолжит с того места, где остановился, и не создаст дубликаты ответов на одни и те же отзывы.

## 5. Итоговая схема потока данных

```
[Пользователь] 
     │
     ▼
[Сайт app_test] — POST /api/reviews → [PostgreSQL]
     │
     ▼ (REST API)
[Worker client.fetch_new_reviews()]
     │
     ▼
[Worker processor.detect_tone()] → определение тональности
     │
     ▼
[Worker processor.generate_response()] → OpenAI API или fallback
     │
     ▼
[Worker client.create_review()] → создаёт ответ в БД
     │
     ▼
[Worker client.update_review()] → помечает отзыв как обработанный
     │
     ▼
[Worker state.mark_review_as_processed()] → сохраняет в state.json
     │
     ▼ (опционально)
[Telegram bot] → уведомление в чат
```
