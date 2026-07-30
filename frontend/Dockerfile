# Stage 1: Build the Vue app
FROM node:18-alpine AS builder

WORKDIR /app

# Устанавливаем переменные окружения для сборки
ARG VUE_APP_API_URL
ENV VUE_APP_API_URL=$VUE_APP_API_URL

# Копируем только файлы зависимостей для эффективного кэширования слоев Docker
COPY package*.json ./

# npm ci автоматически устанавливает только нужные для сборки пакеты без вызова python/make/g++
RUN npm ci

# Копируем исходный код и собираем проект
COPY . .
RUN npm run build

# Stage 2: Nginx stage to serve content
FROM nginx:1.27-alpine

# Переносим PID-файл в доступное для пользователя webapp место
RUN sed -i 's|pid[[:space:]].*;|pid /tmp/nginx.pid;|' /etc/nginx/nginx.conf

# Заменяем стандартный конфиг на свой
COPY frontend-nginx.conf /etc/nginx/conf.d/default.conf

# Создаем системного пользователя без привилегий
RUN addgroup -S webapp && adduser -S -G webapp -s /sbin/nologin webapp

# Переносим кэш и логи Nginx в /tmp, чтобы пользователь webapp мог туда писать
RUN rm -rf /var/cache/nginx /var/log/nginx && \
    ln -s /tmp /var/cache/nginx && \
    ln -s /tmp /var/log/nginx

# Создаем папку для статики и копируем файлы с правильными правами
RUN mkdir -p /usr/share/nginx/html/momo-store && \
    chown -R webapp:webapp /usr/share/nginx/html/momo-store

COPY --from=builder --chown=webapp:webapp /app/dist /usr/share/nginx/html/momo-store

# Переключаемся на непривилегированного пользователя
USER webapp

# Порт 8080 
EXPOSE 8080

# HEALTHCHECK 
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget -q -O /dev/null http://127.0.0 || exit 1
