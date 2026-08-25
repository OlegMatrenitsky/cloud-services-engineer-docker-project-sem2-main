FROM node:18-alpine AS builder

WORKDIR /app

ARG VUE_APP_API_URL
ENV VUE_APP_API_URL=$VUE_APP_API_URL

ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


FROM nginx:1.27-alpine

RUN sed -i 's/listen[[:space:]]\+80;/listen 8080;/' \
    /etc/nginx/conf.d/default.conf

RUN sed -i 's|pid[[:space:]].*;|pid /tmp/nginx.pid;|' \
    /etc/nginx/nginx.conf

COPY frontend-nginx.conf /etc/nginx/conf.d/default.conf

RUN addgroup -S webapp && \
    adduser -S -G webapp -s /sbin/nologin webapp

RUN rm -rf /var/cache/nginx /var/log/nginx && \
    mkdir -p /tmp/nginx /tmp/nginx/client_temp /tmp/nginx/proxy_temp \
    /tmp/nginx/fastcgi_temp /tmp/nginx/uwsgi_temp /tmp/nginx/scgi_temp && \
    ln -s /tmp/nginx /var/cache/nginx && \
    ln -s /tmp/nginx /var/log/nginx

RUN mkdir -p /usr/share/nginx/html/momo-store && \
    chown -R webapp:webapp /usr/share/nginx/html

COPY --from=builder --chown=webapp:webapp \
    /app/dist /usr/share/nginx/html/momo-store

USER webapp

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget -q -O /dev/null http://127.0.0.1:8080/ || exit 1