# Этап 1: Сборка Go-приложения
FROM golang:1.23-alpine AS builder
WORKDIR /app

# Кэшируем зависимости
COPY go.mod go.sum ./
RUN go mod download

# Копируем исходный код
COPY . .

# Сборка с оптимизацией размера бинарника
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-w -s" \
    -o /app/bin/app ./cmd/api

# Этап 2: Финальный образ
FROM alpine:3.20

# Создаем системного пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Копируем только исполняемый файл
COPY --from=builder --chown=appuser:appgroup /app/bin/app ./app

# Используем непривилегированного пользователя
USER appuser

# Открываем порт приложения
EXPOSE 8081

# HEALTHCHECK
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8081/health || exit 1

# Команда запуска контейнера
CMD ["./app"]