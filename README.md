# Проект Kittygram - это соцсеть для котоводов, в которой пользователи могут делиться с другими пользователями фотографиями своих питомцев.



### Деплой проекта на сервер:

Подключаемся к удаленному серверу:
```
ssh -i путь_до_файла_с_SSH_ключом/название_файла_закрытого_SSH-ключа login@ip
```
Например:
```
ssh -i D:/Dev/vm_access/yc-username yc-user@153.0.2.10
```
Клонируйте репозиторий:
```
git@github.com:v0vanjke/infra_sprint1.git
```
Установите на сервер пакетный менеджер и утилиту для создания виртуального окружени
```
sudo apt install python3-pip python3-venv -y
```
Перейдите в папку проекта:
```
cd infra_sprint1/backend
```
Создайте и активируйте виртуальное окружение:
```
python -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
```
Установите зависимости из файла requirements.txt:
```
pip install -r requirements.txt
```
Выполните миграции:
```
python manage.py migrate
```
### Настройка Gunicorn:

Создайте юнит для сервера Gunicorn:
```
sudo nano /etc/systemd/system/gunicorn_kittygram.service
```
Заполните файл:
```
[Unit]
# Это текстовое описание юнита, пояснение для разработчика.
Description=gunicorn daemon 

# Условие: при старте операционной системы запускать процесс только после того, 
# как операционная система загрузится и настроит подключение к сети.
# Ссылка на документацию с возможными вариантами значений 
# https://systemd.io/NETWORK_ONLINE/
After=network.target 

[Service]
# От чьего имени будет происходить запуск:
# укажите имя, под которым вы подключались к серверу.
User=yc-user 

# Путь к директории проекта:
# /home/<имя-пользователя-в-системе>/
# <директория-с-проектом>/<директория-с-файлом-manage.py>/.
# Например:
WorkingDirectory=/home/yc-user/infra_sprint1/backend/

# Команду, которую вы запускали руками, теперь будет запускать systemd:
# /home/<имя-пользователя-в-системе>/
# <директория-с-проектом>/<путь-до-gunicorn-в-виртуальном-окружении> --bind 0.0.0.0:8000 backend.wsgi
ExecStart=/home/yc-user/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:9000 kittygram_backend.wsgi

[Install]
# В этом параметре указывается вариант запуска процесса.
# Значение <multi-user.target> указывают, чтобы systemd запустил процесс,
# доступный всем пользователям и без графического интерфейса.
WantedBy=multi-user.target
```

Запустите процесс gunicorn_kittygram.service:
```
sudo systemctl start gunicorn_kittygram
```
Добавьте процесс в список автозапуска:
```
sudo systemctl enable gunicorn_kittygram
```
Проверьте работоспособность процесса:
```
sudo systemctl status gunicorn_kittygram
```

### Установка Nginx.

Находясь на удалённом сервере, из любой директории выполните команду:
```
sudo apt install nginx -y
```
Запустите Nginx командой:
```
sudo systemctl start nginx
```
Укажите файрволу, какие порты должны остаться открытыми. Для этого выполните на сервере две команды по очереди:
```
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
```
Включите файервол:
```
sudo ufw enable
```

### Настраиваем Nginx.

Перейдиет в директорию infra_sprint1/frontend и выполните команду:
```
npm run build
```
Скопируйте в системную директорию содержимое папки .../frontend/build/:
```
sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/ 
```
Через редактор Nano откройте файл конфигурации веб-сервера:
```
sudo nano /etc/nginx/sites-enabled/default
```
Заполните файл, указав ваш домен: 
```
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
```

Сохраните файл и выполните команду для проверки корректности файла:
```
sudo nginx -t
```
если ответ:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
то можно продолжать, в противном случае вернитесь на шаг назад и проверьте корректность составления файла.

Cоздайте директорию media в директории /var/www/kittygram/, находясь в директории /var/www/kittygram/ выполните команду:
```
mkdr media
```
Назначьте текущего пользователя владельцем директории media, чтобы Django-приложение могло сохранять картинки. Для этого используйте команду chown:
```
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```
Перезагрузите конфигурацию Nginx:
```
sudo systemctl reload nginx
```
Собираем статику для бэкенда:

Выполните команду:
```
python manage.py collectstatic
```
Скопируйте файлы статики бэкенда в системную директорию /var/www/kittygram/:
```
sudo cp -r /home/yc-user/infra_sprint1/backend/static_backend/ /var/www/kittygram/
```
Перезапустите Gunicorn:
```
sudo systemctl restart gunicorn
```

### Технологии:
Python3, Django==3.2.3

### Автор:
[Raskin Vladimir](https://github.com/v0vanjke)


