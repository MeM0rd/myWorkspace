## Введение

Проект разворачивает локальные зависимости для других проектов. Содержит в себе базы данных и кэш + прокси контейнер для
подключения нескольких проектов под докером на 80 порту.

Перед началом должен быть установлен docker и docker-compose. Так же должны быть свободны 80, 9000 и 9003 порты.

## Документация

### Быстрый старт

Копируем конфиг из шаблона

```bash
cp .env.example .env
```

Устанавливаем aws-cli

```bash
sudo apt install awscli
```
или для macOS
```bash
brew install awscli
```

Устанавливаем настраиваем aws

```bash
aws configure
```

```
AWS Access Key ID: 88815_dbstore
AWS Secret Access Key: O\~hU0;f5H
Default region name [None]: ru-1
Default output format [None]: Просто нажимаем enter
```

Устанавливаем настраиваем aws

```bash
aws configure --profile oarepo
```

```
AWS Access Key ID: 88815_oarepo
AWS Secret Access Key: dsfgst65_32344l|y[J&R5kjsdhi_dskjhfo
Default region name [None]: ru-1
Default output format [None]: Просто нажимаем enter
```

Добавляем сети

```bash
docker network create workspace && docker network create proxy
```

Запускаем контейнеры

```bash
docker-compose up -d
```

### Обновление базы данных

Если работаем из офиса, в `/etc/hosts` нужно добавить следущую запись (только для mariadb):

```
192.168.110.149 stage.napopravku.ru
```

1. Добавить ваш публичный ssh-ключ на сервере Stage в `/home/db-dump-viewer/.ssh/authorized_keys`
2. Подключиться первый раз по `ssh user@server` через консоль, нажать yes для добавления в known_hosts и можно
   отключаться

> База данных обновляется только на запущеный контейнер **postgres** и **maridb**

Для обновления только базы postgres:  `./update-db-postgres-only.sh`

Для обновления только базы oarepo:  `./update-db-oa.sh`

<s>Для обновления всех баз: `./update-db.sh`</s>

> Если вдруг, контейнер db будет постоянно перезапускаться либо что-то просто отвалится - без паники. Нузно удалить папки `.data/mariadb` и `.data/postgres`, перезапустить контейнеры и повторить обновление.

### Запуск sphinx

Клонируем конфиги и пересобираем контейнер

``` bash
git clone git@github.com:napopravku/np-sphinx-config.git sphinx/indexes
docker-compose up -d --build sphinx
```

### Авторизация в GitHub Packages

Это нужно для доступа к нашим базовым образам <a href="https://github.com/napopravku/docker-images">docker-images</a>. Данная манипуляция проводится один раз.

``` bash
docker login ghcr.io -u GITHUB_USERNAME
```

После этой команды в качестве пароля нужно ввести <a href="https://github.com/settings/tokens/new?scopes=repo,workflow,write:packages,delete:packages">GitHub Access Token</a>. Пароль от аккаунта больше не поддерживается всеми методами авторизации на GitHub.

## Добавление нового проекта

Для того что-бы новый проект видел бызы данных из **workspace** нужно добавить его в одну сеть.

Есть две сети **workplace** - нужна для подключения баз данных и кэша и **proxy** - добавляет проекты в сеть
прокси-сервиса для удоной работы над несколькими веб-сервисами.

Возьмем типовой проект с  **php-fpm** и **nginx**:

```yaml
# docker-compose.yml
version: '3.8'

services:
  php:
    image: php:7.4-fpm

  nginx:
    image: nginx:latest
    depends_on:
      - php
```

Композ запускает 2 сервиса. Для **php** нужна база данных, а в **nginx** нужно пробросить порт что бы можно было ходить
через браузер на сайт.

Для этого добавим 2-е сети, подключим их и обавим вспомогательные дерективы

``` yaml
# docker-compose.yml
version: '3.8'

networks:
  # сеть для прокси
  proxy:
    external:
      name: proxy 
  # сеть для баз данных
  workspace:
    external:
      name: workspace 
  # Тут немного подробнее, эта сеть нужна для того что бы nginx смог увидеть php во внутренней сети, так как при переопределении сети у нас пропадает дефолтная
  local:
    driver: bridge
    
services:
  php:
    image: php:7.4-fpm

    # сеть для баз данных и локальная
    networks:
      - workspace
      - local

  nginx:
    image: nginx:latest
    depends_on:
      - php
      
    # сеть для прокси и локальная
    networks:
      - proxy
      - local
      
    # метки позволяют прокси-сервису понять что этот контейнер при обрашении по хосту 
    # нужно показывать именно этот контейнер
    labels:
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.example.rule=Host(`example.loc`)"
      - "traefik.http.services.example.loadbalancer.server.port=80"
 
```

### Внимательно!

В метках нужно заменить не только хост на свой но и **example** в:

- traefik.http.routers.**example**
- traefik.http.services.**example**
