# Лаборатоная работа №6 "Взаимодействие контейнеров"

## Цель работы

Научиться управлять взаимодействием нескольких контейнеров.

## Задание

Создать php приложение на базе двух контейнеров: nginx, php-fpm.

### Описание выполнения работы с ответами на вопросы

1. Создаем репозиторий ```containers06``` и копируем его на компьютер.
2. В директории ```containers06``` создаем директорию ```mounts/site```. В данную директорию pfgbcsdfvt сайт на php.
3. Создаем файл ```.gitignore``` в корне проекта и добавляем в него строки:

   ```git
   # Ignore files and directories
   mounts/site/*
   ```

4. Создаем в директории ```containers06``` файл ```nginx/default.conf``` со следующим содержимым:

```shell
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
      try_files $uri $uri/ /index.php?$args;
    }
    
    location ~ \.php$ {
      fastcgi_pass backend:9000;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
    }
}
```

5. Создаем сеть ```internal``` для контейнеров.

   ```shell
   docker network create internal
    ```

![Img-1](https://imgur.com/LAxnfHo.png)

6. Создаем контейнер ```backend``` со следующими свойствами:

- на базе образа php:7.4-fpm;
- к контейнеру примонтирована директория mounts/site в /var/www/html;
- работает в сети internal.

  ```shell
  docker run -d --name backend --network internal -v D:\Container\containers06\mounts\site:/var/www/html php:7.4-fpm
  ```

![Img-2](https://imgur.com/QFZoW2R.png)

7. Создаем контейнер ```frontend``` со следующими свойствами:

- на базе образа nginx:1.23-alpine;
- с примонтированной директорией mounts/site в /var/www/html;
- с примонтированным файлом nginx/default.conf в /etc/nginx/conf.d/default.conf;
- порт 80 контейнера проброшен на порт 80 хоста;
- работает в сети internal.

  ```shell
  docker run -d --name frontend --network internal -p 80:80 -v D:\Container\containers06\mounts\site:/var/www/html -v D:\Container\containers06\nginx\default.conf:/etc/nginx/conf.d/default.conf nginx:1.23-alpine
  ```

  ![Img-3](https://imgur.com/5fnQpsU.png)

8. Результат выполнения программы:
   ![Img-4](https://imgur.com/2pyh5zx.png)

9. Ответы на вопросы:
    - Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?
      Контейнеры взаимодействуют через созданную сеть ```internal```. Оба контейнера (backend и frontend) подключены к этой сети, поэтому могут обращаться друг к другу по именам контейнеров.
    - Как видят контейнеры друг друга в рамках сети ```internal```?
       - Внутри сети ```internal``` контейнеры могут обращаться друг к другу по именам.
       - Контейнер ```frontend (nginx)``` может обращаться к ```backend (php-fpm)``` по имени ```backend```, потому что Docker автоматически создаёт DNS-имена для контейнеров внутри одной сети.
       - В конфигурации Nginx указан ```fastcgi_pass backend:9000;```, что означает, что nginx будет искать сервер PHP по имени backend и порту 9000.
    - Почему необходимо было переопределять конфигурацию ```nginx```?
        По умолчанию Nginx не настроен для работы с PHP. Обычный Nginx обрабатывает только статические файлы (HTML, CSS, JS). Чтобы Nginx мог передавать запросы PHP в php-fpm, нужно:
         - Указать корневую директорию.
         - Добавить обработку PHP-файлов.

### Выводы

  В ходе выполенния лабораторной работы был освоен навык управлять взаимодействием нескольких контейнеров, а также было создано php приложение на базе двух контейнеров: nginx, php-fpm.
