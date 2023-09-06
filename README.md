##  Tasking
### Описание
Приложение для планирования задач. Есть фронтенд (SPA-приложение на React)
и бэкенд (Django-приложение). Задачи можно добавлять, изменять, удалять
и переводить из группы "незавершённые" в "завершённые".
Проект запущен на виртуальном удалённом сервере в трёх контейнерах: nginx, PostgreSQL и Django+Gunicorn. Заготовленный контейнер с фронтендом используется для сборки файлов. Контейнер с проектом обновляется на Docker Hub.

### Технологический стек
[![Python](https://img.shields.io/badge/-Python-464646?style=flat&logo=Python&logoColor=56C0C0&color=cd5c5c)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-464646?style=flat&logo=Django&logoColor=56C0C0&color=0095b6)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/-Django%20REST%20Framework-464646?style=flat&logo=Django%20REST%20Framework&logoColor=56C0C0&color=cd5c5c)](https://www.django-rest-framework.org/)
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-464646?style=flat&logo=PostgreSQL&logoColor=56C0C0&color=0095b6)](https://www.postgresql.org/)
[![Nginx](https://img.shields.io/badge/-NGINX-464646?style=flat&logo=NGINX&logoColor=56C0C0&color=cd5c5c)](https://nginx.org/ru/)
[![gunicorn](https://img.shields.io/badge/-gunicorn-464646?style=flat&logo=gunicorn&logoColor=56C0C0&color=0095b6)](https://gunicorn.org/)
[![Docker](https://img.shields.io/badge/-Docker-464646?style=flat&logo=Docker&logoColor=56C0C0&color=cd5c5c)](https://www.docker.com/)
[![Docker-compose](https://img.shields.io/badge/-Docker%20compose-464646?style=flat&logo=Docker&logoColor=56C0C0&color=0095b6)](https://www.docker.com/)
[![Docker Hub](https://img.shields.io/badge/-Docker%20Hub-464646?style=flat&logo=Docker&logoColor=56C0C0&color=cd5c5c)](https://www.docker.com/products/docker-hub)
[![GitHub%20Actions](https://img.shields.io/badge/-GitHub%20Actions-464646?style=flat&logo=GitHub%20actions&logoColor=56C0C0&color=0095b6)](https://github.com/features/actions)

### Как запущен
В файле settings.py в разрешенные хосты добавим id удаленного сервера и доменное имя проекта
```
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'tasking.sytes.net']
```
В корневой директории создан файл .env с переменными окружения
```
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=
DB_HOST=
DB_PORT=
TOKEN=
```
Собраны образы из нужных директорий для frontend, backend  и nginx. Образы проверены, загружены на Docker Hub и будут использоваться для запуска контейнеров в файле конфигурации docker-compose.production.yml

На удаленном сервере создала директорию taski/, в неe скопировала файлы docker-compose.production.yml и .env командой из корневой директории на локальном компе
```
scp -i path_to_SSH/SSH_name docker-compose.production.yml \
    username@server_ip:/home/username/taski/docker-compose.production.yml
scp -i path_to_SSH/SSH_name .env\
    username@server_ip:/home/username/taski/.env 
```

Для запуска Docker Compose в режиме демона выполнила команду на сервере в папке taski/ с флагом -d:
```
sudo docker compose -f docker-compose.production.yml up -d
```
Выполнила миграции, собрала статику, создала суперпользователя
```
docker compose -f docker-compose.production.yml exec backend python manage.py makemigrations
docker compose -f docker-compose.production.yml exec backend python manage.py migrate
docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /app/backend_static/static/
docker compose -f docker-compose.production.yml exec backend python manage.py createsuperuser
```
Настроила внешний Nginx так, чтобы он отправлял в докер все запросы без исключения — и запросы к приложению, и запросы к статике. На сервере в редакторе nano открыла конфиг Nginx: nano /etc/nginx/sites-enabled/default. Изменила настройки location в секции server.
```
location / {
        proxy_pass http://127.0.0.1:8000;
}
```
Выполнила команду проверки конфигурации:
```
sudo nginx -t
```
Перезагрузила конфиг Nginx
```
sudo service nginx reload
```
## Автоматизация деплоя: CI/CD
Workflow написан так, что каждый git push в ветку main является событием-триггером для запуска тестирования и деплоя.
При возникновении события-триггера GitHub Actions читает файл с описанием workflow и для каждой отдельной задачи-job этого workflow выделяет отдельный раннер, который будет выполнять эту задачу.
На раннере GitHub Actions, надо развернуть докер. Код проекта на раннере есть, докерфайлы тоже есть — надо выполнить билд и прямо с раннера запушить образы на Docker Hub.
На боевом сервере надо перезапустить контейнеры, чтобы они создались из обновлённых образов. Раннер должен аутентифицироваться от моего имени на докерхабе — чтобы запушить образы. А потом аутентифицироваться и на боевом сервере — чтобы перезапустить контейнеры.
Секретные данные можно спрятать на платформе GitHub Actions, в специальном хранилище. Значения из этого хранилища будут переданы в переменные, доступ к которым будет только у раннера при запуске воркфлоу.  
Переменные c токенами, паролями и другими приватными данными на платформе GitHub Actions называют secrets, хранят их в разделе Secrets.
Для отслеживания выполнения workflow в GitHub Actions, можно подключить к работе Telegram-бота.

### Развёрнутый проект:
https://tasking.sytes.net

### Над проектом работала (бэкенда и деплой)
[AnnaMihailovna](https://github.com/AnnaMihailovna/)
