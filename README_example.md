<div align="center">

# 🔍 AI Search & Analytics Platform

![Python](https://img.shields.io/badge/Python-3.12+-3776AB?style=flat-square&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=flat-square&logo=fastapi&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-3-4FC08D?style=flat-square&logo=vue.js&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791?style=flat-square&logo=postgresql&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-0.5+-563d7c?style=flat-square)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4.1--mini-412991?style=flat-square&logo=openai&logoColor=white)

**B2B-лидогенерация с ИИ-поиском, скорингом и аналитикой**

[Описание](#-описание-проекта) • [Технологии](#-технологии) • [Архитектура](#-архитектура) • [Скриншоты](#-скриншоты) • [Установка](#-установка-и-настройка) • [Структура](#-структура-проекта) • [Компоненты](#-детальное-описание-компонентов) • [Масштабирование](#-масштабирование-проекта) • [Лицензия](#-лицензия) • [Заключение](#-заключение) • [Контакты](#-контакты)

</div>

---

## 📋 Описание проекта

**AI Search & Analytics Platform** — это fullstack-платформа для автоматизации поиска и квалификации B2B-лидов в инженерных и промышленных секторах. Платформа объединяет многоканальный поиск компаний, ИИ-скоринг, генерацию вопросов для звонков и RAG-ассистента по корпоративной базе знаний.

> **Примечание:** База знаний (`rag_docs/`) сгенерирована LLM на основании открытых источников компании НЕВАТОМ и используется исключительно для демонстрации возможностей RAG-системы.

### Основные возможности

- 🔍 **Многоканальный поиск** — агрегация данных из госзакупок, ФНС, 2ГИС, DaData и отраслевых каталогов
- 🤖 **ИИ-скоринг** — рейтинг лидов по 25+ факторам (релевантность, тендеры, СРО, оборот, возраст)
- 📊 **Аналитика** — автоматическая генерация вопросов для звонков на основе профиля компании
- 💬 **Чат-виджет** — RAG-ассистент с двумя режимами («Поиск» и «Справка»)
- 📤 **Экспорт** — CSV/XLSX для импорта в CRM
- 🔧 **Расширяемость** — модульная архитектура коннекторов

### Бизнес-цель

**Проблема:** Сотрудники отдела продаж тратят до 120 минут на ручной поиск компаний по разрозненным каталогам, тендерным площадкам и реестрам. Данные устаревают, контакты теряются, скоринг — интуитивный.

**Решение:** Платформа сокращает время на поиск и подготовку к звонку со 120 минут до 3–5 минут на лид за счёт автоматической агрегации, скоринга и генерации скриптов.

---

## 🛠 Технологии

| Категория | Технологии |
|-----------|------------|
| **Язык программирования** | Python 3.12+ |
| **Backend API** | FastAPI 0.115+, Uvicorn, Pydantic v2 |
| **Frontend** | Vue 3 + TypeScript + Vite |
| **LLM & Embeddings** | OpenAI GPT-4.1-mini, text-embedding-3-small (ProxyAPI) |
| **Векторная БД** | ChromaDB 0.5.23 |
| **Основная БД** | PostgreSQL 15+ (asyncpg, SQLAlchemy 2.0) |
| **Парсинг** | httpx, selectolax, BeautifulSoup4 |
| **Коннекторы** | 2GIS API, DaData API, Zakupki.gov.ru RSS/API, ФНС ЕГРУЛ |
| **Тестирование** | pytest, pytest-asyncio |
| **Инфраструктура** | Docker, Docker Compose |

---

## 🏗 Архитектура

```
┌─────────────────────────────────────────────────────────────────┐
│                      Frontend (Vue 3 + Vite)                    │
│   Поиск лидов │ Аналитика │ Чат-виджет │ Экспорт │ Настройки    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Backend (FastAPI + Python 3.12)              │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ RAG Engine   │  │ Search       │  │ Analytics    │           │
│  │ (retrieval)  │  │ Service      │  │ Engine       │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│         │                 │                  │                  │
│         ▼                 ▼                  ▼                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ Chat Widget  │  │ Connectors   │  │ Lead Scorer  │           │
│  │ (modes:      │  │ Zakupki│ФНС  │  │ (rule-based) │           │
│  │  search/help)│  │ 2GIS│DaData  │  └──────────────┘           │
│  └──────────────┘  └──────────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  PostgreSQL   │    │   ChromaDB    │    │   Disk Cache  │
│  (leads,      │    │  (embeddings, │    │  (retrieval   │
│   sessions,   │    │   RAG docs)   │    │   cache)      │
│   analytics)  │    └───────────────┘    └───────────────┘
└───────────────┘
```

---

## 📸 Скриншоты

### Главная страница — вкладка «Поиск»

![Search Tab](docs/images/search_tab.png)

### Результаты поиска с группировкой

![Search Results](docs/images/search_results.png)

### Карточка лида со скорингом

![Lead Card](docs/images/lead_card.png)

### Аналитика — сгенерированные вопросы для звонка

![Analytics Modal](docs/images/analytics_modal.png)

### Чат-виджет с RAG-ответом в рехиме «Поиск»

![Widget Search](docs/images/widget_search.png)

### Чат-виджет с RAG-ответом в рехиме «Справка»

![Widget Help](docs/images/widget_help.png)

---

## 🚀 Установка и настройка

### Предварительные требования

- Python 3.12+
- Node.js 18+
- Docker Desktop
- API-ключ OpenAI (ProxyAPI)

### Клонирование репозитория

```bash
git clone <repository-url>
cd ai-search-analytics-web-app
```

### Создание виртуального окружения

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
source .venv/bin/activate     # Linux/macOS
```

### Установка зависимостей

```bash
pip install -e ".[dev]"
```

### Настройка переменных окружения

```bash
copy .env.example .env
```

Отредактируйте `.env`, внеся свои ключи

### Запуск базы данных (Docker)

```bash
docker compose up -d
```

### Создание таблиц в базе данных (миграции)

**venv НЕ нужен** — миграции выполняются внутри Docker-контейнера PostgreSQL.

```powershell
# Скопируйте файлы миграций в контейнер и примените
docker compose cp migrations/001_initial.sql postgres:/tmp/001_initial.sql
docker compose cp migrations/002_analytics.sql postgres:/tmp/002_analytics.sql
docker compose cp migrations/003_chat.sql postgres:/tmp/003_chat.sql

# Примените миграции
docker compose exec -T postgres psql -U nevatom -d lead_search -f /tmp/001_initial.sql
docker compose exec -T postgres psql -U nevatom -d lead_search -f /tmp/002_analytics.sql
docker compose exec -T postgres psql -U nevatom -d lead_search -f /tmp/003_chat.sql

# Проверьте, что таблицы созданы
docker compose exec -T postgres psql -U nevatom -d lead_search -c "\dt"
```

### Запуск Backend

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

API Docs: http://localhost:8000/docs

### Индексация RAG (первый запуск)

```bash
curl -X POST http://localhost:8000/rag/ingest \
  -H "Content-Type: application/json" \
  -d '{"path":"","force_rebuild":true}'
```

### Запуск Frontend

```bash
cd frontend
npm install
npm run dev
```

Приложение: http://localhost:5173

> **Примечание:** Полная инструкция по запуску проекта находится в файле [START_GUIDE.md](START_GUIDE).

---

## 📁 Структура проекта

```
ai-search-analytics-web-app/
├── main.py                          # Точка входа FastAPI
├── pyproject.toml                   # Зависимости и конфигурация
├── .env.example                     # Шаблон переменных окружения
├── .gitignore                       # Исключения для Git
├── docker-compose.yml               # Запуск базы данных (Docker)
├── LICENSE                          # Лицензия MIT
├── START_GUIDE.md                   # Полная инструкция по запуску проект
├── README.md                        # Документация проекта
│
├── frontend/                        # Vue 3 SPA
│   ├── src/
│   │   ├── components/              # search/, analytics/, chat/
│   │   ├── api/                     # Клиенты HTTP
│   │   ├── types/                   # TypeScript интерфейсы
│   │   └── App.vue
│   └── package.json
│
├── core/                            # Ядро бизнес-логики
│   ├── config/
│   │   └── settings.py              # Pydantic Settings (env)
│   ├── db/
│   │   ├── models.py                # SQLAlchemy ORM
│   │   └── session.py               # Async engine + session
│   ├── search/
│   │   ├── connectors/              # Zakupki, FNS, 2GIS, DaData, ABOK
│   │   ├── schemas.py               # Pydantic модели поиска
│   │   ├── scoring.py               # LeadScorer (правила)
│   │   └── okved_mapping.py         # Сопоставление ОКВЭД
│   ├── rag/
│   │   ├── document_loader.py       # Парсинг Markdown + YAML frontmatter
│   │   ├── chunker.py               # Разбиение на чанки
│   │   ├── embeddings.py            # OpenAI embeddings
│   │   ├── chroma_store.py          # ChromaDB wrapper
│   │   ├── intent_classifier.py     # Классификация намерений
│   │   ├── retrieval.py             # RetrievalPipeline
│   │   └── cache.py                 # LRU + disk cache
│   ├── chat/
│   │   ├── schemas.py               # ChatMode, ChatContext
│   │   ├── modes.py                 # resolve_categories
│   │   ├── suggestions.py           # Подсказки по режиму/роли
│   │   ├── quick_actions.py         # Быстрые действия
│   │   └── session_memory.py        # Память сессии
│   ├── analytics/
│   │   ├── schemas.py               # Модели отчётов
│   │   ├── report_builder.py        # Формирование блоков отчёта
│   │   └── question_generator.py    # Генерация вопросов через LLM
│   └── dependencies.py              # FastAPI Depends фабрики
│
├── services/                        # Сервисный слой
│   ├── search_service.py            # Логика поиска + скоринг + пагинация
│   ├── analytics_service.py         # Аналитика + экспорт XLSX/CSV
│   ├── chat_service.py              # Обработка сообщений чата
│   └── rag_service.py               # RAG ingestion + search
│
├── handlers/                        # FastAPI роутеры
│   ├── search_router.py
│   ├── analytics_router.py
│   ├── chat_router.py
│   └── rag_router.py
│
├── rag_docs/                        # База знаний (Markdown)
│   ├── system/                      # Справка по интерфейсу
│   ├── sales/                       # Скрипты, возражения, методологии
│   ├── segments/                    # Описание сегментов клиентов
│   ├── products/                    # Технические спецификации
│   └── company/                     # Логистика, гарантия, сертификация
│
├── migrations/                      # SQL-миграции
│   ├── 001_initial.sql
│   ├── 002_analytics.sql
│   └── 003_chat.sql
│
├── tests/                           # Pytest
│   ├── rag/
│   ├── search/
│   ├── analytics/
│   └── chat/
│
└── docs/
    └── images/                      # Скриншоты
```

---

## 📖 Детальное описание компонентов

### Ядро системы (Core)

**RAG Engine** (`core/rag/`)
- **Document Loader** — парсит Markdown-файлы из `rag_docs/`, извлекает YAML frontmatter (`doc_id`, `category`, `audience`, `tags`) и тело документа
- **Chunker** — разбивает документы на чанки по заголовкам h2-h4 с перекрытием 120 токенов, максимум 900 токенов на чанк
- **Embeddings** — генерирует векторы через OpenAI `text-embedding-3-small` (1536 измерений)
- **Chroma Store** — хранит чанки с метаданными (`category`, `audience`, `priority`, `section_title`), поиск по косинусному сходству
- **Intent Classifier** — определяет категорию запроса (`system`, `product`, `sales`, `segments`, `company`) и намерение (`search`, `help`, `technical`) по ключевым словам или через LLM
- **Retrieval Pipeline** — выполняет векторный поиск с фильтрацией по категории и аудитории, пост-фильтрацию по порогу сходства (0.35), дедупликацию по `doc_id`, форматирование контекста для LLM
- **Cache** — двухуровневый кэш (LRU в памяти + diskcache на диске) с TTL для FAQ (48 ч) и технических вопросов (24 ч)

### Базы данных

**PostgreSQL** (`core/db/`)
- Хранит сессии поиска, лиды (компания, ИНН, сегмент, регион, скоринг, контакты), сессии чата, историю взаимодействий, отчёты аналитики, шаблоны вопросов
- Асинхронный доступ через `asyncpg` + SQLAlchemy 2.0

**ChromaDB** (`data/chroma/`)
- Persistent-режим, коллекция `nevatom_kb_v1`
- Метаданные каждого чанка: `doc_id`, `category`, `audience`, `tags`, `section_title`, `priority`, `is_table`
- Индексирование полей `category`, `audience`, `tags` для быстрой фильтрации

### Поиск лидов (Search Service)

**Архитектура поиска**
1. Пользователь задаёт фильтры (сегменты, регион, ключевые слова)
2. `SearchService` параллельно запрашивает все активные коннекторы
3. Результаты мержатся по ИНН или названию компании
4. `LeadScorer` оценивает каждого кандидата по 15+ факторам
5. Отфильтрованные и отсортированные лиды сохраняются в PostgreSQL
6. Пагинация, группировка (по сегменту/региону) и экспорт

**Коннекторы** (`core/search/connectors/`)

| Коннектор | Источник | Что возвращает | Особенности |
|-----------|----------|----------------|-------------|
| **ZakupkiConnector** | zakupki.gov.ru | Заказчики тендеров по ОКПД2/ОКПД | RSS + Open API, извлечение ИНН из описания |
| **FNSConnector** | egrul.nalog.ru | Юрлица по ключевым словам | Public API ФНС, двухэтапный поиск (запрос → результат) |
| **DGISConnector** | 2GIS API | Компании по рубрикам | API v3.0, free tier 1000 запросов/день, поиск по рубрикам и адресам |
| **DaDataConnector** | DaData API | Организации по названию | Suggest API, обогащение данных (адрес, руководитель, статус) |
| **RegistryConnector** | abok.ru | Отраслевой каталог | Парсинг HTML через selectolax |

**Парсинг ответов**
- Каждый коннектор преобразует сырой ответ API в `RawLeadCandidate` (унифицированная структура)
- Поля: `company_name`, `inn`, `segment`, `region`, `okved_codes`, `turnover_mln`, `employees`, `website`, `contact_*`, `has_tenders_12m`, `has_sro`, `vent_mention`
- `SearchService._merge_candidates()` объединяет дубли по ИНН, дополняя пропущенные поля из разных источников

**Скоринг** (`core/search/scoring.py`)
- Правиловая система (не ML), веса настраиваются через `.env`
- Ключевые факторы: ОКВЭД (+20), тендеры за 12м (+15), сайт активен (+10), оборот >200 млн (+5), СРО (+10), упоминание вентиляции (+10), возраст ≥2 лет (+5)
- Порог попадания в результаты: 10+ баллов

### Аналитика лидов (Analytics Engine)

**Процесс**
1. Пользователь выбирает 1–5 лидов → запускает аналитику
2. `AnalyticsService` формирует контекст из профиля лида (сегмент, регион, скоринг, ОКВЭД)
3. LLM генерирует структурированный отчёт:
   - Обязательные блоки: обзор компании, боли и потребности, гипотезы по продуктам, конкурентный контекст, чек-лист подготовки к звонку
   - Опциональные блоки: рекомендуемые вопросы (из банка + сгенерированные LLM)
4. Отчёт сохраняется в PostgreSQL, вопросы — в `question_sets`
5. Менеджер отмечает использованные вопросы → система обучается

**Question Generator** (`core/analytics/question_generator.py`)
- Сначала ищет релевантные вопросы в `sales_questions_bank.md` через RAG
- Если недостаточно — генерирует дополнительные через LLM с привязкой к методологии (SPIN, BANT, MEDDIC)
- Каждый вопрос содержит: `text`, `methodology_tag`, `category`, `source`

### Чат-виджет (Chat Widget)

**Два режима работы**

| Режим | Контекст | RAG-категории | Тип вопросов |
|-------|----------|---------------|--------------|
| 🔍 **Поиск** | Текущий лид (lead_id, segment, score) | `sales`, `segments`, `product` | «Какие вопросы задать монтажнику?», «Как отработать возражение?» |
| ❓ **Справка** | Интерфейс системы | `system` | «Как экспортировать в Excel?», «Что означает скоринг 65?» |

**Архитектура обработки сообщения**
1. Frontend отправляет: `query`, `mode`, `user_role`, `context` (lead_id, segment, score)
2. `ChatService` определяет категории RAG через `resolve_categories(mode, query, context)`
3. `RetrievalPipeline` ищет релевантные чанки в ChromaDB с фильтром по категории
4. Если `max_score < 0.5` — fallback («К сожалению, я не нашёл точного ответа...»)
5. Иначе — LLM генерирует ответ с переданным контекстом и системным промптом
6. Ответ стримится через SSE (Server-Sent Events), сохраняется в БД

**Функционал**
- Плавающая кнопка 🤖 (горячая клавиша Ctrl+K)
- История сообщений в рамках сессии
- Контекстные подсказки по режиму и роли
- Обратная связь 👍/👎 с выбором причины
- Быстрые действия (quick actions): «Запустить аналитику», «Скопировать вопросы», «Экспортировать»

### Сервисный слой (Services)

| Сервис | Ответственность |
|--------|-----------------|
| **SearchService** | Оркестрация коннекторов, мерж кандидатов, скоринг, пагинация, группировка, экспорт CSV/XLSX |
| **AnalyticsService** | Генерация отчётов через LLM, управление вопросами, экспорт аналитики, логирование использования |
| **ChatService** | Обработка сообщений чата, вызов RAG, fallback-логика, стриминг SSE, сохранение сессий |
| **RAGService** | Ingestion документов, поиск по базе знаний, управление коллекциями ChromaDB |

### Утилиты

- **Ingest Documents** — CLI/API для загрузки Markdown в ChromaDB с перестроением индекса
- **Logging Config** — структурированное логирование, подавление шума от внешних библиотек
- **Disk Cache** — кэширование результатов retrieval для снижения нагрузки на OpenAI и ChromaDB

---

## 📡 API Endpoints (краткий справочник)

### Поиск

| Метод | Путь | Описание |
|-------|------|----------|
| `GET` | `/search/filters` | Метаданные фильтров |
| `POST` | `/search/run` | Запуск поиска |
| `POST` | `/search/more` | Следующая страница |
| `POST` | `/search/export` | Экспорт CSV/XLSX |

### Аналитика

| Метод | Путь | Описание |
|-------|------|----------|
| `POST` | `/analytics/run` | Аналитика по лидам |
| `GET` | `/analytics/report/{id}` | Отчёт |
| `POST` | `/analytics/export` | Экспорт отчётов |

### Чат

| Метод | Путь | Описание |
|-------|------|----------|
| `GET` | `/chat/suggestions` | Подсказки |
| `POST` | `/chat/message` | Сообщение (JSON) |
| `POST` | `/chat/stream` | Стриминг (SSE) |
| `POST` | `/chat/feedback` | 👍/👎 |

### RAG

| Метод | Путь | Описание |
|-------|------|----------|
| `POST` | `/rag/ingest` | Индексация |
| `POST` | `/rag/search` | Поиск по базе знаний |

---

## 🏗 Масштабирование проекта

### Горизонтальное масштабирование

| Уровень | Решение |
|---------|---------|
| **Backend** | Несколько экземпляров FastAPI за Nginx / load balancer |
| **Database** | PostgreSQL репликация (master-slave) |
| **Cache** | Redis для кэширования запросов к LLM и поисковым API |
| **Queue** | Celery для фоновых задач (ингестия, массовая аналитика) |

### Вертикальное масштабирование

| Компонент | Улучшение |
|-----------|-----------|
| **LLM** | Переход на gpt-4-turbo для сложной аналитики |
| **RAG** | Multi-hop retrieval, гибридный поиск (keyword + semantic) |
| **Скоринг** | ML-модель на исторических данных вместо правил |

### Новые источники данных

| Источник | Сложность | Стоимость |
|----------|-----------|-----------|
| **Rusprofile API** | ⭐⭐ | ~5000 ₽/мес |
| **Checko API** | ⭐⭐ | ~3000 ₽/мес |
| **Kontur.Focus** | ⭐⭐⭐ | ~10000 ₽/мес |

### Новые фичи

- Интеграция с CRM (Bitrix24, AmoCRM)
- Авто-обновление лидов (еженедельный ре-скрейпинг)
- Email-рассылка на основе аналитики
- Дашборд с метриками (воронка, конверсия, ROI)
- Multi-tenant (SaaS для нескольких компаний)

---

## 📄 Лицензия

MIT License — подробности в файле [LICENSE](LICENSE)

---

## 🎯 Заключение

**AI Search & Analytics Platform** — это готовое к production решение для B2B-лидогенерации в инженерных секторах. Платформа объединяет многоканальный поиск, ИИ-скоринг, генерацию скриптов звонков и RAG-ассистента в едином интерфейсе.

**Ключевые достижения:**
- Сокращение времени на подготовку к звонку со 120 до 3–5 минут
- Агрегация 4+ источников данных в единую базу лидов
- Автоматическая генерация вопросов для звонков с привязкой к методологиям продаж
- RAG-ассистент с контекстной памятью и двумя режимами работы

---

## 📞 Контакты

**Автор:** Ivan P  
**Telegram:** [@nonoyessure](https://t.me/nonoyessure)  

---

<div align="center">

**⭐ Ставьте звезду, если проект полезен!**

🚀

</div>
