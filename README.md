# Oil Spill AI — Комплекс для автоматического обнаружения разливов нефти

---

## Архитектура и взаимодействие компонентов

Oil Spill AI — это микросервисная система для автоматизированной обработки спутниковых изображений с целью обнаружения разливов нефти. Проект состоит из четырёх основных сервисов:

- **Frontend** — веб-интерфейс для пользователей (Next.js, React, Tailwind CSS, next-intl)
- **Backend** — REST API для загрузки архивов, интеграции с ML и управления задачами (FastAPI, Redis)
- **Celery worker** — отдельный сервис для асинхронной обработки задач, очередей и взаимодействия между backend и ML (Celery, Python)
- **ML сервис** — микросервис для сегментации изображений с помощью нейросети YOLO (FastAPI, Ultralytics YOLO, PyTorch)

### Взаимодействие компонентов

1. **Пользователь** загружает zip-архив с изображениями через веб-интерфейс (frontend).
2. **Frontend** отправляет архив на backend через API.
3. **Backend** сохраняет архив, извлекает изображения и ставит задачу в очередь Celery.
4. **Celery worker** асинхронно обрабатывает задачи: поочерёдно отправляет изображения в ML сервис для сегментации.
5. **ML сервис** возвращает сегментированные изображения Celery worker-у/backend-у.
6. **Backend** собирает результаты, формирует архив и предоставляет ссылку для скачивания пользователю.

Все временные и итоговые файлы хранятся в папке `results/` backend-а. Очередь задач и хранение статусов реализованы через Redis.

---

## Быстрый запуск всей системы через Docker Compose

1. Убедитесь, что у вас установлен [Docker](https://www.docker.com/) и [Docker Compose](https://docs.docker.com/compose/).
2. В корне репозитория выполните:
   ```bash
   docker-compose up --build
   ```
3. Сервисы будут доступны по адресам:
   - **Frontend:** http://localhost:3000
   - **Backend API:** http://localhost:8000
   - **ML сервис:** http://localhost:8002
   - **Celery worker:** внутренний сервис (нет отдельного порта, логируется в консоль)
   - **Redis:** localhost:6379 (внутренний сервис)

> **Примечание:** Для работы ML сервиса требуется файл весов модели: `ml/weights/best.pt`. Убедитесь, что он присутствует до сборки контейнера.

---

## Модули проекта

### 1. Frontend (`frontend/`)
- **Назначение:** Веб-интерфейс для загрузки архивов, отслеживания статуса задач, скачивания результатов, просмотра документации и информации о проекте.
- **Технологии:** Next.js, React, Tailwind CSS, next-intl, Three.js
- **Запуск отдельно:**
  ```bash
  cd frontend
  npm install
  npm run dev
  ```
- **Документация:** [frontend/README.md](frontend/README.md)

### 2. Backend (`backend/`)
- **Назначение:** REST API, постановка задач в очередь Celery, интеграция с ML сервисом, управление задачами и результатами.
- **Технологии:** FastAPI, Redis, Python
- **Запуск отдельно:**
  ```bash
  cd backend
  pip install -r requirements.txt
  uvicorn app.main:app --host 0.0.0.0 --port 8000
  # Redis должен быть запущен отдельно
  ```
- **Документация:** [backend/README.md](backend/README.md)

### 3. Celery worker (`backend/`)
- **Назначение:** Асинхронная обработка задач, взаимодействие между backend и ML сервисом.
- **Технологии:** Celery, Python
- **Запуск отдельно:**
  ```bash
  cd backend
  celery -A app worker --loglevel=info
  # Redis должен быть запущен отдельно
  ```
- **Документация:** [backend/README.md](backend/README.md)

### 4. ML сервис (`ml/`)
- **Назначение:** Сегментация изображений с помощью нейросети YOLO, REST API для получения результатов.
- **Технологии:** FastAPI, Ultralytics YOLO, PyTorch
- **Запуск отдельно:**
  ```bash
  cd ml
  pip install -r requirements.txt
  uvicorn app.main:app --host 0.0.0.0 --port 8002
  # или ./run.sh
  ```
- **Документация:** [ml/README.md](ml/README.md)

---

## Переменные окружения и конфигурация

- **Backend:**
  - `CELERY_BROKER_URL`, `CELERY_RESULT_BACKEND` — адрес Redis (по умолчанию: `redis://localhost:6379/0`)
  - `ML_SERVICE_URL` — адрес ML сервиса (по умолчанию: `http://ml:8002/segment` в Docker Compose)
  - `.env` файл для локальной настройки
- **Frontend:**
  - `NEXT_PUBLIC_API_URL` — адрес backend API (если не localhost)
  - `.env.local` для локальной настройки
- **ML сервис:**
  - Путь к весам модели: `ml/weights/best.pt`

---

## Расширение и поддержка

- **Добавить язык:**
  - Frontend: добавьте JSON-файл перевода в `frontend/messages/` и зарегистрируйте язык в `src/i18n/routing.ts`.
- **Добавить страницу или компонент:**
  - Frontend: создайте файл в `src/app/[locale]/` или компонент в `src/components/`.
- **Добавить новый API endpoint:**
  - Backend: реализуйте endpoint в `backend/app/main.py` и добавьте обработку в Celery/ML при необходимости.
- **Обновить модель:**
  - ML сервис: замените файл весов в `ml/weights/best.pt`.

---

## Полезные ссылки

- [Next.js Docs](https://nextjs.org/docs)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [next-intl Docs](https://next-intl-docs.vercel.app/)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Ultralytics YOLO Docs](https://docs.ultralytics.com/)
- [Three.js Docs](https://threejs.org/docs/)

---

## Работа с репозиторием

1. **Клонирование:**
   ```bash
   git clone https://github.com/oil-spill-ai/oil-spill-devops.git
   cd oil-spill-devops
   ```
2. **Ветки:**
   - Основная ветка: `main`
   - Для новых фич создавайте отдельные ветки от main: `feature/your-feature-name`
3. **Коммиты:**
   - Пишите осмысленные сообщения к коммитам на английском или русском
   - Группируйте связанные изменения в один коммит
4. **Пулл-реквесты:**
   - Перед созданием PR убедитесь, что код проходит тесты и не содержит конфликтов
   - Описывайте суть изменений в описании PR
5. **Code style:**
   - Соблюдайте принятый стиль кода (ESLint, Prettier, Black для Python)
6. **Документация:**
   - Обновляйте документацию при изменениях в архитектуре или API

---

# 📘 Инструкция по работе с репозиторием, содержащим Git-сабмодули

---

## 🔽 1. Клонирование репозитория с сабмодулями

Обычный `git clone` не подтягивает содержимое сабмодулей — только ссылки на них. Чтобы сразу получить всё:

```bash
git clone --recurse-submodules <URL_репозитория>
```

Если репозиторий уже был клонирован без параметра `--recurse-submodules`, добавь сабмодули вручную:

```bash
git submodule update --init --recursive
```

---

## 💻 2. Работа локально с репозиторием и сабмодулями

Предположим, структура проекта следующая:

```
main-repo/
├── .gitmodules
└── submodules/
    └── frontend/  <-- сабмодуль
```

### 🔁 Обновление изменений из GitHub

#### A. Обновить основной репозиторий:
```bash
git pull
```

#### B. Обновить все сабмодули:
```bash
git submodule update --init --recursive
```

> ⚠️ Это подтянет сабмодули на те коммиты, которые указаны в основном репозитории.

---

### ✏️ Внесение изменений в сабмодуль

1. Перейди в директорию сабмодуля:
    ```bash
    cd submodules/frontend
    ```

2. Сделай изменения:
    ```bash
    git checkout -b feature/new-ui   # по желанию
    git add .
    git commit -m "Изменил UI"
    git push origin feature/new-ui
    ```

3. Вернись в основной репозиторий:
    ```bash
    cd ../..
    ```

4. Зафиксируй обновлённую ссылку на коммит сабмодуля:
    ```bash
    git add submodules/frontend
    git commit -m "Обновил коммит подмодуля frontend"
    git push
    ```

---

### ⬆️ Публикация изменений

Чтобы изменения полностью отразились в удалённом репозитории:

1. Пушим изменения в сабмодуле:
    ```bash
    cd submodules/frontend
    git push
    ```

2. Пушим обновлённую ссылку из основного репозитория:
    ```bash
    cd ../..
    git push
    ```

---

## 🛠 Полезные команды

### Проверить статус сабмодулей:
```bash
git submodule status
```

### Обновить сабмодули до последних коммитов из их веток:
```bash
git submodule update --remote --merge
```
> ⚠️ Используй только при необходимости вручную "освежить" сабмодули.

---

## 💡 Советы

- Не коммить изменения сабмодуля и основного репозитория в одном `git commit` — это отдельные действия.
- Сабмодуль — это полноценный Git-репозиторий со своим `origin`, ветками и `git push`.
- В команде обязательно делитесь обновлённой ссылкой на коммит сабмодуля через `git add`, `commit` и `push` в основном репозитории.
