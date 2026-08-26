# Kittygram

Kittygram — веб-сервис для публикации фотографий котиков. Пользователи могут
зарегистрироваться, добавлять карточки питомцев, указывать их имя, год рождения
и цвет, загружать фотографии и присваивать котикам достижения.

Проект упакован в Docker-контейнеры. GitHub Actions автоматически проверяет
backend и frontend, собирает образы, публикует их на Docker Hub и запускает
контейнерную smoke-проверку приложения.

## Технологии

- Python 3.9, Django 3.2 и Django REST Framework;
- PostgreSQL 13;
- React;
- Gunicorn и Nginx;
- Docker и Docker Compose;
- GitHub Actions, Docker Hub;
- Ruff и pytest.

## Структура проекта

- `backend/` — REST API и административная панель Django;
- `frontend/` — клиентское React-приложение;
- `nginx/` — gateway, раздача статики и пользовательских изображений;
- `docker-compose.yml` — локальная сборка контейнеров;
- `docker-compose.production.yml` — запуск готовых образов из Docker Hub;
- `.github/workflows/main.yml` — CI/CD workflow.

## Локальный запуск в Docker

Требуются Git, Docker и Docker Compose.

1. Клонируйте репозиторий и перейдите в его каталог:

   ```bash
   git clone https://github.com/dariiyak/kittygram_final.git
   cd kittygram_final
   ```

2. Создайте файл `.env` из примера:

   ```bash
   cp .env.example .env
   ```

3. Замените в `.env` тестовые значения. Обязательные переменные:

   ```dotenv
   POSTGRES_DB=kittygram
   POSTGRES_USER=kittygram_user
   POSTGRES_PASSWORD=strong-database-password
   DB_NAME=kittygram
   DB_HOST=db
   DB_PORT=5432
   SECRET_KEY=long-random-django-secret-key
   DEBUG=False
   ALLOWED_HOSTS=localhost,127.0.0.1
   TIME_ZONE=Europe/Moscow
   ```

   В `ALLOWED_HOSTS` можно через запятую добавить доменное имя или IP сервера.

4. Соберите и запустите контейнеры:

   ```bash
   docker compose up -d --build
   ```

   При старте backend автоматически применит миграции, соберёт статику и
   запустит Gunicorn.

5. При необходимости создайте администратора:

   ```bash
   docker compose exec backend python manage.py createsuperuser
   ```

Приложение будет доступно по адресу <http://localhost:9000/>, API — по адресу
<http://localhost:9000/api/>, административная панель — по адресу
<http://localhost:9000/admin/>.

Остановить проект:

```bash
docker compose down
```

Для удаления контейнеров вместе с базой данных и загруженными файлами можно
выполнить `docker compose down --volumes`. Эта команда удаляет данные без
возможности восстановления.

## Запуск готовых образов

После создания `.env` запустите опубликованные образы из Docker Hub:

```bash
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
```

Используются образы:

- `dariiyak/kittygram_backend:latest`;
- `dariiyak/kittygram_frontend:latest`;
- `dariiyak/kittygram_gateway:latest`.

## Проверки

Проверить backend локально можно командами:

```bash
python -m pip install -r backend/requirements.txt
python -m ruff check backend/
python backend/manage.py test cats
```

Проверки frontend:

```bash
cd frontend
npm ci
npm test -- --watchAll=false
npm run build
```

## CI/CD

Workflow запускается при push в ветку `main`. В настройках GitHub-репозитория
должны быть добавлены секреты:

- `DOCKER_USERNAME` — логин Docker Hub;
- `DOCKER_PASSWORD` — токен Docker Hub с правом записи;
- `TELEGRAM_TO` и `TELEGRAM_TOKEN` — необязательные секреты для уведомлений.

После успешных тестов workflow собирает и публикует три Docker-образа, затем
поднимает приложение через `docker-compose.production.yml` и проверяет ответ
gateway по HTTP.
