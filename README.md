# Kittygram 
Kittygram создан, чтобы хозяева могли делиться своими котиками.

<img src="https://otkritkis.com/wp-content/uploads/2022/06/m7wxf.gif" width="385px" align="center">

### Функционал сайта 

После регистрации у вас есть возможность:

- Просматривать, добавлять и удалять своих котиков;
- Присваивать своим котикам различные достижения; 
- Просматривать чужих котиков и их достижения;

### Сайт
Рабочая версия сайта доступна по ссылке: https://infra.servebeer.com/

## Установка 

1. Форкните репозиторий, клонируйте его, и перейдите в него:

    ```bash
    git clone ...
    ```
    ```bash
    cd kittygram_final
    ```
2. Создайте файл .env и заполните его своими данными:

    ```bash
    POSTGRES_DB=...
    POSTGRES_USER=...
    POSTGRES_PASSWORD=...
    DB_HOST=...
    DB_PORT=...
    ```

### Создание Docker-образов

1. Для создания Docker-образов последовательно выполните следующие команды, замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram

    ```bash
    mkdir kittygram
    ```

3. Для установки docker compose на сервер, поочередно выполните следующие команды:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. Скопируйте на сервер в директорию kittygram/ файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```

5. Переходите в директорию kittygram/ и запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:

    ```bash
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
    ```

9. Проверьте работоспособность конфига и перезапустите Nginx:

    ```bash
    sudo nginx -t 
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории

    ```bash
    kittygram/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_PASSWORD - пароль от аккаунта DockerHub
    DOCKER_USERNAME - логин DockerHub
    HOST - IP адресс сервера
    USER - логин на сервере
    SSH_KEY - SSH ключ
    SSH_PASSPHRASE - пароль от сервера
    TELEGRAM_TO - ваш Telegram ID
    TELEGRAM_TOKEN - токена вашего Telegram бота
    ```
