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

RUN apk add --no-cache wget

# Создаем системного пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Копируем только исполняемый файл
COPY --from=builder --chown=appuser:appgroup /app/bin/app ./app

# Используем непривилегированного пользователя
USER appuser

# Открываем порт приложения
EXPOSE 8081

# Healthcheck
# Проверено, что BusyBox в golang:1.20-alpine содержит nc и поддерживает
# параметр -z
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s \ 
    CMD sh -c "nc -z localhost 8081 || exit 1"


# Команда запуска контейнера
CMD ["./app"]