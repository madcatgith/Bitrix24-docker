# Bitrix24 docker-compose

### Настройка окружения:

Настройки можно изменить в файле ```.env```.


```
BX_PUBLIC_HTML_PATH     # путь к директории public_html, в которой содержатся директории хостов
BX_MODULES_PATH         # путь к репозиторию modules
BX_LOGS_PATH            # путь к директории в которой контейнеры должны хранить логи
BX_MYSQL_PATH           # путь DB volume
BX_PHP_VERSION          # версия php 7|8

```

### Как запустить?
```
docker-compose up -d
```

## Как заполнять подключение к БД?
- DB-HOST: mysql
- DB-USER: bitrix
- DB-NAME: sitemanager
- DB-PASS: находится в docker-compose файле(не секьюрно, но для локальной разработки пойдет)

## Примечание:
- По умолчанию стоит папка ```var/www/public_html``` но это легко поменять
- не забудьте подпихнуть файлик cron_events.php из корня проекта в ./var/www/public_html/bitrix/php_interface/  - чтобы завелась почта на cron
- Почтовый клииент доступен по адресу: (ваш ip):8025

## Push & Pull
Проверку системы не проходит но работает.
```
Путь для публикации команд: http://push-server-pub/bitrix/pub/
Для всего остального домен: bitrix24-sub.docker.localhost
```

## Если захотите подключать папки BITRIX и UPLOADS по simlink:
- На мастер-сайте необходимо настроить nfs-server и дать доступ к данным папкам
- В конец docker-compose добавьте блок с mount папками(ip- мастер-сайта)

```
 upload-mount:
    driver: local
    driver_opts:
      type: nfs4
      o: nfsvers=4,addr=192.168.0.100,rw
      device: ":/public_html/upload"
  bitrix-mount:
    driver: local
    driver_opts:
      type: nfs4
      o: nfsvers=4,addr=192.168.0.100,rw
      device: ":/public_html/bitrix"

```
- подключите их в php и nginx контейнерах как volumes

      - "upload-mount:/var/www/public_html/upload"
      - "bitrix-mount:/var/www/public_html/bitrix"

## Если вдруг захотите маунтить папки с другого докера:
-разместите в нем контейнер c nfs сервером

```
  nfs:
    image: itsthenetwork/nfs-server-alpine:12
    container_name: nfs
    restart: unless-stopped
    privileged: true
    environment:
      - SHARED_DIRECTORY=/data
    volumes:

      - ./var/www:/data

    ports:
      - 2049:2049
    networks:
      bx:
        aliases:
          - nfs

```
