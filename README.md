# Проект Kittygram - это соцсеть для котоводов, в которой пользователи могут делиться с другими пользователями фотографиями своих питомцев.



### Деплой проекта на сервер:

Подключаемся к удаленному серверу:

ssh -i путь_до_файла_с_SSH_ключом/название_файла_закрытого_SSH-ключа login@ip

Например:

ssh -i D:/Dev/vm_access/yc-username yc-user@153.0.2.10

Клонируйте репозиторий:

git@github.com:v0vanjke/infra_sprint1.git

Установите на сервер пакетный менеджер и утилиту для создания виртуального окружени

sudo apt install python3-pip python3-venv -y

Перейдите в папку проекта:

cd infra_sprint1/backend

Создайте и активируйте виртуальное окружение:

python -m venv venv

source venv/bin/activate

python -m pip install --upgrade pip

Установите зависимости из файла requirements.txt:

pip install -r requirements.txt

Выполните миграции:

python manage.py migrate


### Настройка Gunicorn:

Создайте юнит для сервера Gunicorn:

sudo nano /etc/systemd/system/gunicorn_kittygram.service

Запустите процесс gunicorn_kittygram.service:

sudo systemctl start gunicorn_kittygram

Добавьте процесс в список автозапуска:

sudo systemctl enable gunicorn_kittygram

Проверьте работоспособность процесса:

sudo systemctl status gunicorn_kittygram



### Установка Nginx.

Находясь на удалённом сервере, из любой директории выполните команду:

sudo apt install nginx -y

Запустите Nginx командой:

sudo systemctl start nginx

Укажите файрволу, какие порты должны остаться открытыми. Для этого выполните на сервере две команды по очереди:

sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH

Включите файервол:

sudo ufw enable


### Настраиваем Nginx.

Перейдиет в директорию infra_sprint1/frontend и выполните команду:

npm run build

Скопируйте в системную директорию содержимое папки .../frontend/build/:

sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/ 

Через редактор Nano откройте файл конфигурации веб-сервера:

sudo nano /etc/nginx/sites-enabled/default

Заполните файл, указав ваш домен: 

server {
        server_name <ваш-домен>;

        location /admin/ {
               proxy_pass http://127.0.0.1:9000;
        }

        location /media/ {
               root /var/www/kittygram/;
        }

        location /api/ {
               proxy_pass http://127.0.0.1:9000;
        }
       
        location / {
               root /var/www/kittygram;
               index index.html index.htm;
               try_files $uri /index.html;
        }


Сохраните файл и выполните команду для проверки корректности файла:

sudo nginx -t

если ответ:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

то можно продолжать, в противном случае вернитесь на шаг назад и проверьте корректность составления файла.

Cоздайте директорию media в директории /var/www/kittygram/, находясь в директории /var/www/kittygram/ выполните команду:

```
mkdr media
```

Назначьте текущего пользователя владельцем директории media, чтобы Django-приложение могло сохранять картинки. Для этого используйте команду chown:

sudo chown -R <имя_пользователя> /var/www/kittygram/media/

Перезагрузите конфигурацию Nginx:

sudo systemctl reload nginx

Собираем статику для бэкенда:

Через редактор Nano откройте файл settings.py, укажите новое значение для константы STATIC_URL и создайте константу STATIC_ROOT:

STATIC_URL = '/static_backend/'
STATIC_ROOT = BASE_DIR / 'static_backend'

Сохраните изменения и закройте файл, при активированном виртуальном окружении перейдите в директорию с файлом manage.py и выполните команду:

python manage.py collectstatic

Скопируйте файлы статики бэкенда в системную директорию /var/www/kittygram/:

sudo cp -r /home/yc-user/infra_sprint1/backend/static_backend/ /var/www/kittygram/

Перезапустите Gunicorn:

sudo systemctl restart gunicorn


### Технологии:
Python3, Django==3.2.3, 

### Автор:
Raskin Vladimir


