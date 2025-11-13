# Traefik (локально) — глобальный reverse-proxy с HTTPS

Этот набор файлов поднимает Traefik (v2) как глобальный reverse-proxy для локальной разработки с поддержкой HTTPS через локально подписанный сертификат (mkcert).

Ключевая идея:
- Traefik слушает порты 80/443 и проксирует контейнеры по правилам Host.
- Для HTTPS используем локально доверенный сертификат (сгенерировать с помощью mkcert) — он действует для `*.localhost`.

Подготовка (из корня репозитория):

1) Создать docker-сеть (один раз):

```fish
docker network create web
```

2) Установить mkcert и локально доверенный CA (инструкции: https://github.com/FiloSottile/mkcert). Пример генерации сертификата (fish shell):

```fish
# Установить локальный CA
mkcert -install

# Сгенерировать сертификат для всех поддоменов localhost
mkcert -cert-file traefik/certs/traefik.crt -key-file traefik/certs/traefik.key "*.localhost" localhost
```

После этого в `traefik/certs/` появятся `traefik.crt` и `traefik.key`.

3) Поставить права на приватный ключ и `acme.json` (рекомендуется):

```fish
chmod 600 traefik/acme.json traefik/certs/traefik.key
```

4) Поднять Traefik:

```fish
docker compose up -d
```

Проверки:
- Dashboard Traefik: http://localhost:8080
- Публичный HTTPS: любой сервис с правилом Host `*.localhost` будет доступен по https://<имя>.localhost если сервис настроен через Traefik (см. пример ниже).

Пример сервиса (docker-compose snippet) — используйте в проекте и подключайте к внешней сети `web`:

```yaml
services:
  myapp:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`project1.localhost`)"
      - "traefik.http.routers.myapp.entrypoints=websecure"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
    networks:
      - web

networks:
  web:
    external: true
```

Пояснения и подсказки:
- Я использовал `*.localhost` как удобный домен для множества локальных проектов (project1.localhost, project2.localhost и т.д.). `localhost` и его поддомены обычно резолвятся в loopback.
- Если вы предпочитаете другой домен (например, `.test`), нужно позаботиться о резолвинге этих имён в 127.0.0.1 (например, через /etc/hosts или локальный DNS), и сгенерировать сертификат для нужного домена.
- Если не хотите использовать mkcert, можно использовать самоподписанный сертификат, но тогда браузерам нужно будет вручную доверять CA.

Дальше (опционально):
- ограничить доступ к dashboard, добавить middleware аутентификации; включить ACME (Let's Encrypt) — для локальной машины это обычно не нужно.

Если хотите — могу добавить systemd unit или docker-compose override, чтобы Traefik автоматически запускался при старте машины.
