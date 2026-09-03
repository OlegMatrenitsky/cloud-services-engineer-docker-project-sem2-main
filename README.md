# Momo Store Project – Docker Deployment

Этот проект представляет собой веб-приложение **Momo Store** с backend на Go и frontend на Vue.js.

Приложение запускается в Docker-контейнерах и управляется с помощью Docker Compose.  
Для внешнего доступа используется Nginx в качестве reverse proxy.

## Структура проекта


- **backend/** – Go REST API сервер.
- **frontend/** – Vue.js одностраничное приложение (SPA).
- **docker-compose.yml** – оркестрация контейнеров (backend, frontend, proxy).
- **nginx.conf** – конфигурация Nginx для реверс-прокси.
- **.github/workflows/deploy.yaml** – CI/CD workflow для сборки, сканирования и деплоя Docker-образов.
- **README.md** – текущее описание и инструкции.

## Trivy — сканирование уязвимостей

Для проверки безопасности Docker-образов в проекте используется **Trivy**.

Trivy выполняет поиск известных уязвимостей в:

- базовых Docker-образах;
- системных пакетах;
- зависимостях приложения.

В CI/CD pipeline Trivy запускается в отдельной job `trivy_scan` после сборки и публикации Docker-образов.

Проверяются следующие образы:

```text
docker-project-backend:latest
docker-project-frontend:latest
