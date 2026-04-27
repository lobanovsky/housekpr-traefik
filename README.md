# Traefik reverse proxy

Traefik v3 + автоматические SSL-сертификаты Let's Encrypt.

## Структура

```
docker-compose.yml        — два сервиса: docker-proxy и traefik
traefik.yml               — основной конфиг Traefik
nginx-docker-proxy.conf   — конфиг nginx-прокси для Docker socket
rules/                    — дополнительные роутеры (file provider)
acme.json                 — сертификаты Let's Encrypt (не в git)
```

## Почему docker-proxy

Docker Engine 27+ поднял минимальную поддерживаемую версию API с 1.24 до 1.40.
Traefik жёстко зашивает версию 1.24 в Go SDK при первом подключении к Docker — это не
меняется настройками или переменными окружения.

Решение: nginx-контейнер `docker-proxy` стоит между Traefik и Docker socket и
переписывает `/v1.24/` → `/v1.45/` в URL каждого запроса. Traefik подключается
по `tcp://docker-proxy:2375` вместо прямого монтирования сокета.

Это также рекомендованная Traefik практика безопасности: прямой доступ к Docker socket
из Traefik опасен — компрометация Traefik даёт полный доступ к Docker daemon.

## Деплой

```bash
# создать сеть если не существует
docker network create housekpr-network

# создать пустой acme.json с правильными правами
touch acme.json && chmod 600 acme.json

docker compose up -d
```

## Добавить новый сервис

Добавить лейблы в `docker-compose.yml` нужного сервиса:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.housekpr.ru`)"
  - "traefik.http.routers.myapp.entrypoints=https"
  - "traefik.http.routers.myapp.tls=true"
  - "traefik.http.routers.myapp.tls.certresolver=letsEncrypt"
  - "traefik.http.services.myapp.loadbalancer.server.port=3000"
networks:
  - housekpr-network
```

Сервис должен быть в сети `housekpr-network`.
