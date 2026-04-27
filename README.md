# Traefik reverse proxy

Traefik v3 + автоматические SSL-сертификаты Let's Encrypt.

## Структура

```
docker-compose.yml   — сервис traefik
traefik.yml          — основной конфиг Traefik
rules/               — дополнительные роутеры (file provider)
acme.json            — сертификаты Let's Encrypt (не в git)
```

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
