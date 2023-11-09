Как запустить проект:

Как запустить проект и перейти в него в командной строке:

git clone https://github.com/v0vanjke/infra_sprint1.git

cd infra_sprint1

Создать и активировать виртуально еокрудение:

python3 -m venv venv

source venv/bin/activate

python3 -m pip install --upgrade pip

Установить зависимости из файла requirements.txt:

pip install -r requirements.txt

Выполнить миграции:

python3 manage.py migrate

Запустить проект:

python3 manage.py runserver
