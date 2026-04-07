# Remnawave-Panel-Install
Подробное пошаговое руководство по развертыванию Remnawave. Это руководство содержит пошаговое описание настройки панели управления. Ваш VPS должен соответствовать минимальным требованиям Remnawave, прочитать их можно тут https://docs.rw/docs/overview/comparison-of-functions

⚠️Гайд полностью написан на Русском языке.
_____________________________________________________________

<a href="https://hostoff.link/invite/REMNAWAVE">Рекомендуемый и лучший хостинг для панели Remnawave</a>

Начнем с базовой настройки панели управления Remnawave.

# Этап 1 - Обновление системы и установка curl
`apt update && apt upgrade && apt install curl`

# Этап 2 - Установите Docker.
`curl -fsSL https://get.docker.com | sh`

# Этап 3 - Создайте каталог Remnawave и перейдите в него
`mkdir -p /opt/remnawave && cd /opt/remnawave`

# Этап 4 - Загрузите файл Docker Compose
`curl -o docker-compose.yml https://raw.githubusercontent.com/remnawave/backend/refs/heads/main/docker-compose-prod.yml`

# Этап 5 — Загрузите .env
`curl -o .env https://raw.githubusercontent.com/remnawave/backend/refs/heads/main/.env.sample`

# Этап 6 — Сгенерируйте секретных JWT-токенов.
`sed -i "s/^JWT_AUTH_SECRET=.*/JWT_AUTH_SECRET=$(openssl rand -hex 64)/" .env && sed -i "s/^JWT_API_TOKENS_SECRET=.*/JWT_API_TOKENS_SECRET=$(openssl rand -hex 64)/" .env`

# Этап 7 — Сгенерируйте дополнительных JWT-Токенов
`sed -i "s/^METRICS_PASS=.*/METRICS_PASS=$(openssl rand -hex 64)/" .env && sed -i "s/^WEBHOOK_SECRET_HEADER=.*/WEBHOOK_SECRET_HEADER=$(openssl rand -hex 64)/" .env`

# Этап 8 — Меняйте пароль Postgres и синхронизируйте DATABASE_URL
`pw=$(openssl rand -hex 24) && sed -i "s/^POSTGRES_PASSWORD=.*/POSTGRES_PASSWORD=$pw/" .env && sed -i "s|^\(DATABASE_URL=\"postgresql://postgres:\)[^\@]*\(@.*\)|\1$pw\2|" .env`

# Этап 9 — Настройка конфигурации .env
`nano .env`
Замените в .env эти значения на свои.

FRONT_END_DOMAIN="panel.whysead.com"
SUB_PUBLIC_DOMAIN="sub.whysead.com"

Поменяйте "panel.whysead.com" & "sub.whysead.com" на свои домены

И сохарните изменения.

# Этап 10 - Запуск контейнеров
`docker compose up -d && docker compose logs -f -t`

# Этап 11 - Создание файла Caddyfile
`mkdir -p /opt/remnawave/caddy && cd /opt/remnawave/caddy && nano Caddyfile`
Вставьте следующую конфигурацию.

```
https://REPLACE_WITH_YOUR_DOMAIN {
        reverse_proxy * http://remnawave:3000
}
:443 {
    tls internal
    respond 204
}

https://SUBSCRIPTION_PAGE_DOMAIN {
        reverse_proxy * http://remnawave-subscription-page:3010
}
```
Замените "https://REPLACE_WITH_YOUR_DOMAIN" и "https://SUBSCRIPTION_PAGE_DOMAIN" на свои значения.
# Этап 12 - Создание docker-compose.yml

`cd /opt/remnawave/caddy && nano docker-compose.yml`

Вставьте следующую конфигурацию.

```
services:
    caddy:
        image: caddy:2.9
        container_name: 'caddy'
        hostname: caddy
        restart: always
        ports:
            - '0.0.0.0:443:443'
            - '0.0.0.0:80:80'
        networks:
            - remnawave-network
        volumes:
            - ./Caddyfile:/etc/caddy/Caddyfile
            - caddy-ssl-data:/data

networks:
    remnawave-network:
        name: remnawave-network
        driver: bridge
        external: true

volumes:
    caddy-ssl-data:
        driver: local
        external: false
        name: caddy-ssl-data
```

# Этап 13 - Запустите контейнер

`docker compose up -d && docker compose logs -f -t`

_________________________________________________________________________________________________________________

# Настройка Remnawave Subscription Page

# Этап 1 - Создайте docker-compose.yml файл

`mkdir -p /opt/remnawave/subscription && cd /opt/remnawave/subscription && nano docker-compose.yml`

Вставьте в содержимое файла docker-compose.yml

```
services:
    remnawave-subscription-page:
        image: remnawave/subscription-page:latest
        container_name: remnawave-subscription-page
        hostname: remnawave-subscription-page
        restart: always
        env_file:
            - .env
        ports:
            - '127.0.0.1:3010:3010'
        networks:
            - remnawave-network

networks:
    remnawave-network:
        driver: bridge
        external: true
```

# Этап 2 - Создайте .env файл

`mkdir -p /opt/remnawave/subscription && cd /opt/remnawave/subscription && nano .env`

Вставьте следующую конфигурацию.

```
APP_PORT=3010
REMNAWAVE_PANEL_URL=https://panel.whysead.ru
REMNAWAVE_API_TOKEN=Токен_из_Remnawave
```
Замените "https://panel.whysead.ru" и "REMNAWAVE_API_TOKEN" на свои значения

# Этап 2 - Запустите контейнер.

`docker compose up -d && docker compose logs -f`

# Готово! Панель успешно установлена

Ваша панель доступа по домену - panel.yourdomain.ru
Ваш сайт с подписками доступен по домену - sub.yourdomain.ru




