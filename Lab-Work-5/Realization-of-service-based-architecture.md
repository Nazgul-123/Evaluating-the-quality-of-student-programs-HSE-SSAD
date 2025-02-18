# Лабораторная работа №5

## Тема: Реализация архитектуры на основе сервисов (микросервисной архитектуры)

### Цель работы:
Получить опыт работы по организации взаимодействия сервисов с использованием контейнеров Docker.

## Выполненные задачи:

### 1. Упаковка сервисов в Docker-контейнеры (1 балл)
Для реализации микросервисной архитектуры было принято решение выделить два контейнера:
- **Контейнер с серверной частью** (Telegram-бот на базе `aiogram`)
- **Контейнер с базой данных** (`PostgreSQL`)

Для этого был создан `Dockerfile` для бота и `docker-compose.yml` для запуска обоих сервисов.

**Dockerfile:**
```dockerfile
FROM python:3.10
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  bot:
    build: .
    container_name: telegram_bot
    restart: always
    environment:
      - TELEGRAM_TOKEN=${TELEGRAM_TOKEN}
      - DATABASE_URL=postgresql://user:password@db:5432/code_evaluation
    depends_on:
      - db
    command: ["python", "main.py"]

  db:
    image: postgres:13
    container_name: postgres_db
    restart: always
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=code_evaluation
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### 2. Реализация микросервисной архитектуры и запуск контейнеров (3 балла)
Микросервисная архитектура включает в себя:
- **Сервис Telegram-бота**, который обрабатывает пользовательские запросы и взаимодействует с сервисами анализа кода.
- **Сервис базы данных**, который хранит информацию о пользователях, отправленных кодах и результатах анализа.

Контейнеры были успешно запущены с помощью команды:
```sh
docker-compose up --build -d
```
Работоспособность проверена с помощью отправки кода боту и получения отчета.

### 3. Настройка непрерывной интеграции (CI/CD) (2 балла)
Для обеспечения CI/CD был настроен **GitHub Actions**, который автоматически проверяет код и разворачивает сервис.

**Пример `.github/workflows/ci.yml`**
```yaml
name: CI/CD

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run tests
        run: pytest
```

### 4. Разработка интеграционных тестов (2 балла)
Для проверки взаимодействия сервисов были разработаны интеграционные тесты, которые запускаются в CI.

Пример теста взаимодействия с БД (pytest + asyncpg):
```python
import pytest
import asyncpg

@pytest.mark.asyncio
async def test_database_connection():
    conn = await asyncpg.connect(dsn="postgresql://user:password@localhost:5432/code_evaluation")
    assert conn is not None
    await conn.close()
```

### 5. Настройка непрерывного развертывания (2 балла)
Для деплоя на **Railway.app** настроено автоматическое развертывание контейнеров с серверной частью (ботом) и базой данных PostgreSQL при каждом пуше в `main`. Деплой выполняется с помощью GitHub Actions, который собирает Docker-образы и отправляет их в Railway для развертывания.

## Итог:
- Реализована микросервисная архитектура с использованием **Docker**.
- Настроен **CI/CD** с автоматическим тестированием и развертыванием.
- Разработаны интеграционные тесты для проверки работы сервисов.
- Сервис успешно запущен и протестирован, взаимодействие между ботом и базой данных работает корректно.

