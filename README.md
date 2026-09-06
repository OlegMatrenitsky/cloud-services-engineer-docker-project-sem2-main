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

## Размеры Docker-образов

Используются lightweight Alpine-образы и multi-stage builds.

| Компонент     | Базовый образ                          |     Размер |
| ------------- | -------------------------------------- | ---------: |
| Frontend      | `node:18-alpine` → `nginx:1.27-alpine` | **37.1 MB**|
| Backend       | `golang:1.23-alpine` → `alpine:3.20`   |  **139 MB**|
| Reverse Proxy | `nginx:1.27-alpine`                    | **~20 MB** |


Размер репозиториев проекта:

| Repository                |      Размер |
| ------------------------- | ----------: |
| `docker-project-frontend` | **37.1 MB** |
| `docker-project-backend`  |  **139 MB** |

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

### Секреты

Секретные значения не хранятся непосредственно в `docker-compose.yml`.

Для локальной разработки можно использовать `.env`:

```env
DB_PASSWORD=your-secret
```

Для более безопасного варианта Docker Compose поддерживает secrets:

```yaml
services:
  backend:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

Секрет доступен контейнеру через `/run/secrets/db_password`.

Файлы `.env` и `secrets/` не должны попадать в Git.


## Масштабирование

Backend и frontend поддерживают горизонтальное масштабирование:

```bash
docker compose up -d --scale backend=3 --scale frontend=3
```

Docker Compose запускает несколько экземпляров сервисов, а Nginx распределяет запросы между ними.
