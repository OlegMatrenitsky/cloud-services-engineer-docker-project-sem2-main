# Momo Store – Docker Deployment

Веб-приложение **Momo Store** с backend на Go и frontend на Vue.js, контейнеризированное с помощью Docker и Docker Compose.

## Структура проекта

```text
.
├── backend/              # Go REST API
├── frontend/             # Vue.js SPA
├── nginx.conf            # Nginx reverse proxy
├── docker-compose.yml    # Docker Compose
└── README.md
```

## Архитектура

```text
Internet
   │
   ▼
 Nginx :80
   ├──► Frontend :8080
   └──► Backend  :8081
```

Nginx — единственная точка доступа извне. Backend и frontend работают во внутренних Docker networks.

## Быстрый старт

Требования: Docker и Docker Compose.

```bash
docker compose build
docker compose up -d
```

Приложение:

```text
http://localhost
```

Проверка контейнеров:

```bash
docker compose ps
```

Логи:

```bash
docker compose logs -f
```

Остановка:

```bash
docker compose down
```

## Docker

Для backend и frontend используются **multi-stage builds** и лёгкие Alpine images.

Backend:

```text
golang:1.23-alpine → alpine:3.20
```

Frontend:

```text
node:18-alpine → nginx:1.27-alpine
```

В production images не попадают исходный код, Go compiler, Node.js и development dependencies.

## Безопасность

Используются:

* непривилегированные пользователи `appuser` и `webapp`;
* `read_only: true`;
* `cap_drop: ALL`;
* изолированные Docker networks;
* ограничения CPU и памяти;
* restart policies;
* healthchecks;
* только порт `80` открыт наружу.

Чувствительные данные не хранятся в Dockerfile или Git и передаются через переменные окружения / Docker Secrets.

## Масштабирование

Backend и frontend поддерживают горизонтальное масштабирование:

```bash
docker compose up -d --scale backend=3 --scale frontend=3
```

Docker Compose запускает несколько экземпляров сервисов, а Nginx распределяет запросы между ними.
