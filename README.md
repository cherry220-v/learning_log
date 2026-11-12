# Learning Log

Учебный проект по материалам учебника «Python & Django» (автор Каламбет В.Б., раздел «Проект learning_log»).

Learning Log — сетевой журнальный сервис: пользователь создаёт темы и ведёт по ним записи о том, что он изучил. Каждый пользователь видит и редактирует только свои собственные данные.

## Возможности

- Домашняя страница, список тем, страница отдельной темы со списком записей.
- Регистрация (`/users/register/`), вход (`/users/login/`), выход (POST-форма в шапке сайта).
- Добавление тем (`/new_topic/`), добавление записей (`/new_entry/<id>/`), редактирование записей (`/edit_entry/<id>/`).
- Доступ к списку тем, темам, формам ограничен декоратором `@login_required`; чужие темы и записи дают ответ 404.
- Оформление — Bootstrap 4 через пакет `django-bootstrap4`.
- Автоматические тесты: `python manage.py test` (4 теста на проверку доступа к темам).
- Развёртывание — Docker Compose: Gunicorn + WhiteNoise, SQLite в именованном томе.

## Технологии

| Компонент | Версия |
| --- | --- |
| Python | 3.12 |
| Django | 5.2 LTS |
| django-bootstrap4 | 26.3 |
| Gunicorn | 27.0 |
| WhiteNoise | 6.12 |

Точный список зависимостей — в `requirements.txt`.

## Структура проекта

```
learning_log/
├── manage.py
├── requirements.txt
├── Dockerfile
├── compose.yaml
├── .env                 # создаётся на сервере, в Git не попадает
├── learning_log/        # настройки проекта (settings.py, urls.py, wsgi.py)
├── learning_logs/       # основное приложение: модели, формы, представления, шаблоны
└── users/               # приложение пользователей: регистрация, шаблоны входа/выхода
```

## Запуск локально (без Docker)

1. Python 3.12. Перейдите в папку проекта (рядом с `manage.py`).
2. Создайте и активируйте виртуальное окружение:

   ```bash
   python3.12 -m venv ll_env
   source ll_env/bin/activate          # Windows PowerShell: .\ll_env\Scripts\Activate.ps1
   ```

3. Установите зависимости:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Примените миграции и создайте суперпользователя:

   ```bash
   python manage.py migrate
   python manage.py createsuperuser
   ```

5. Запустите сервер разработки и откройте http://127.0.0.1:8000/ :

   ```bash
   python manage.py runserver
   ```

6. Автотесты:

   ```bash
   python manage.py test
   ```

## Запуск через Docker Compose

1. Создайте рядом с `manage.py` файл `.env` (список переменных — ниже).
2. Соберите и запустите:

   ```bash
   docker compose up -d --build
   ```

Миграции и сборка статики (`collectstatic`) выполняются автоматически при старте контейнера, SQLite-база хранится в именованном томе `learning_log_data`. Приложение доступно по адресу из `APP_BIND_IP:APP_PORT`.

## Переменные окружения (.env)

| Переменная | Назначение | Пример |
| --- | --- | --- |
| `DJANGO_SECRET_KEY` | Секретный ключ Django. На сервере задать свой длинный случайный ключ. | `replace-with-a-long-random-secret` |
| `DJANGO_DEBUG` | Режим отладки. На сервере — `False`. | `False` |
| `DJANGO_ALLOWED_HOSTS` | Список адресов через запятую, по которым открывается приложение. | `192.168.1.11,localhost,127.0.0.1` |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | Адреса источников для защиты CSRF (схема + адрес + порт). | `http://192.168.1.11:20010` |
| `DJANGO_USE_HTTPS` | `True`, если приложение открыто по HTTPS (включает безопасные cookie). | `False` |
| `SQLITE_PATH` | Путь к базе SQLite внутри контейнера. | `/app/data/db.sqlite3` |
| `APP_BIND_IP` | IP, на котором публикуется порт (для учебного сервера — `0.0.0.0`). | `0.0.0.0` |
| `APP_PORT` | Внешний порт, выданный проекту командой `newproject`. | `20010` |

Файл `.env` содержит секретный ключ и в репозиторий не загружается — он есть в `.gitignore`.

## Учётные данные учебной базы

В архиве приложен файл `db.sqlite3` с учебными данными (в Git он не попадает — запись `db.sqlite3` в `.gitignore`). Для входа:

| Пользователь | Пароль | Роль |
| --- | --- | --- |
| `admin` | `AdminLL_2026` | суперпользователь (админка `/admin/`), владелец тем Chess и Rock Climbing |
| `user1` | `User1LL_2026` | обычный пользователь (список тем пуст — видно изоляцию данных) |

Темы: Chess (2 записи), Rock Climbing (1 запись). Пароли только для локальной учёбы; при развёртывании создайте своих пользователей.

Если база не нужна — удалите `db.sqlite3` и создайте заново командами `python manage.py migrate` и `python manage.py createsuperuser`.
