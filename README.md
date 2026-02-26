# club-app-front

React + Vite SPA для клубного приложения.

## Прод-домены

- Frontend: `https://club.web.nikcnn.xyz`
- API: `https://club.api.nikcnn.xyz`

## Переменные окружения

Основная переменная для API:

- `VITE_API_BASE_URL` — базовый домен API, который вшивается на этапе `vite build`.

Шаблон переменных:

```bash
cp .env.example .env
```

`.env.example` и `.env.production` уже содержат:

```env
VITE_API_BASE_URL=https://club.api.nikcnn.xyz
```

## Локальный запуск без Docker

1. Установить зависимости:

```bash
npm ci
```

2. Запустить dev-сервер:

```bash
npm run dev
```

> В dev-режиме, если `VITE_API_BASE_URL` не задан, используется fallback `http://localhost:8000`.

## Сборка production-бандла

```bash
npm run build
```

## Docker (production)

В проекте используется multi-stage Dockerfile:

- Stage 1 (`node:20-alpine`): `npm ci` + `npm run build`
- Stage 2 (`nginx:alpine`): раздача `dist` и SPA fallback через `try_files ... /index.html`

### Сборка образа

```bash
docker build \
  --build-arg VITE_API_BASE_URL=https://club.api.nikcnn.xyz \
  -t club-app-front:latest .
```

### Запуск контейнера

```bash
docker run --rm -p 8080:80 club-app-front:latest
```

Проверка роутинга SPA (не должно быть 404 от nginx):

- `http://localhost:8080/news`
- `http://localhost:8080/clubs/1`

## Docker Compose

```bash
VITE_API_BASE_URL=https://club.api.nikcnn.xyz docker compose up --build -d
```

Остановка:

```bash
docker compose down
```

По умолчанию в `docker-compose.yml` используется:

- сервис `club-web`
- порт `8080:80`
- build arg `VITE_API_BASE_URL` (с дефолтом `https://club.api.nikcnn.xyz`)

## Важно про Vite env

`VITE_*` переменные доступны только на этапе сборки. Для production-деплоя обязательно передавать корректный `VITE_API_BASE_URL` во время `docker build` / `docker compose up --build`.


## Совместимость с backend docker-compose

Файл compose из вашего примера (с `postgres`, `qdrant`, `minio`, `app`) — это **backend-стек**. Для этого репозитория (frontend SPA) он не подходит «как есть», потому что фронтенду не нужны DB/Qdrant/MinIO-сервисы.

Для frontend нужен отдельный сервис (как в `docker-compose.yml` этого репо), который:

- собирает Vite-приложение на этапе build;
- принимает `VITE_API_BASE_URL` **на этапе сборки**;
- раздает статику через Nginx со SPA fallback.

Если хотите запускать backend и frontend в одном compose-проекте, просто добавьте сервис `club-web` из этого репо в ваш общий compose рядом с `app`.
