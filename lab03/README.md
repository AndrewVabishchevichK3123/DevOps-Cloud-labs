# Лабораторная работа 3
В данной лабораторной необходимо для начала узнать что же такое CI/CD; почему это очень полезно, а также проанализировать плохой CI/CD файл, написать хороший с исправленными ошибками, и описать все это

## CI/CD (непрерывная интеграция и непрерывное развертывание/поставка)
Это набор подходов, направленных на автоматизацию и улучшение процессов разработки, тестирования и развертывания ПО, что позволяет быстро и удобно выпускать код в продакшен

## Начнем с плохого CI/CD файла
```

name: Bad CI/CD

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        run: git clone https://github.com/AndrewVabishchevichK3123/DevOps-Cloud-labs
      
      - name: Install dependencies
        run: |
          echo "Installing dependencies..."
          sudo apt-get update
          sudo apt-get install -y python3 python3-pip
          
      - name: Run tests
        run: |
          echo "Skipping tests..."
      
      - name: Deploy
        run: |
           echo "Deploying..."
      
      - name: Cleanup
        run: |
          echo "Cleaning up..."
          rm -rf /build
```

* Неопределенность версий (использование latest):

Использование тега latest в Docker-образах может привести к разным результатам при каждом запуске пайплайна, так как образ будет меняться, и это может вызвать нестабильность.

* Устанавливаются Python и pip, но непонятно, какие зависимости требуются для проекта:

Нет информации о файле requirements.txt или другом способе указания зависимостей. Установка лишних пакетов (например, python3, если проект не использует Python) тратит ресурсы.

* Этап Run tests фактически пропущен:

Тесты — важная часть CI/CD. Без их выполнения нельзя гарантировать, что приложение работает корректно. Если тесты пропущены, в продакшен может быть выложен неработающий или нестабильный код.

* Этап Deploy состоит только из строки: echo "Deploying...":

Никакой реальной логики деплоя нет. Код не перемещается на сервер или в облачное хранилище. Не указаны параметры доступа (например, ключи SSH, адрес сервера, директория деплоя).

* Отсутствие проверки веток при деплое:Этап Cleanup удаляет содержимое директории /build:

Директория /build может быть системной и не связанной с проектом. Команда rm -rf в корневых директориях крайне опасна: можно случайно удалить критически важные файлы.

## Скрины

![image](https://github.com/user-attachments/assets/5887738e-1483-4c8d-8231-65783abe5502)

![image](https://github.com/user-attachments/assets/8826b37f-a2f1-4395-a281-340e79fca411)


Попробуем все исправить

```

name: Good CI/CD

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-22.04
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.9
      
      - name: Install dependencies
        run: |
          echo "Installing dependencies..."
          pip install -r lab03/requirements.txt

      - name: Run tests
        run: |
          echo "Running tests..."
          if [ -d tests ]; then
            pytest tests
          else
            echo "No tests found. Skipping test step."
          fi
      
      - name: Build application
        run: |
          echo "Building application..."
          if [ -d src ]; then
            mkdir -p build
            cp -r src/* build/
            echo "Build complete."
          else
            echo "Directory 'src' does not exist. Skipping build step."
          fi

      - name: Deploy to production
        env:
          DEPLOY_SERVER: ${{ secrets.DEPLOY_SERVER }}
          DEPLOY_USER: ${{ secrets.DEPLOY_USER }}
          DEPLOY_PATH: /var/www/my-app
        run: |
          echo "Deploying to production server..."
          if [ "$(ls -A build/)" ]; then
            scp -o StrictHostKeyChecking=no -r build/* $DEPLOY_USER@$DEPLOY_SERVER:$DEPLOY_PATH
            echo "Deployment complete."
          else
            echo "No files to deploy. Skipping deployment."
          fi

      - name: Cleanup
        run: |
          echo "Cleaning up local build files..."
          rm -rf build


```

* Исправление неопределенности версий:

В "хорошем" CI/CD файле вместо тега latest используется ubuntu-22.04, что позволит избежать нежелательных ошибок, связанных с несовместимостью версий.

* Проверка наличия необходимых файлов и директорий:
  
Прежде чем выполнить важные операции, такие как установка зависимостей, запуск тестов или деплой, файл проверяет, существует ли нужная директория или файл.

* Использование секретов для деплоя:

В файле используется GitHub Secrets для безопасного хранения данных, таких как данные для подключения к серверу (например, DEPLOY_SERVER, DEPLOY_USER), что предотвращает утечку чувствительной информации

* Контроль над зависимостями:
  
Зависимости устанавливаются только из файла requirements.txt, что помогает поддерживать четкость и контроль над тем, какие зависимости должны быть установлены

* Использование конкретной версии Python:

Для стабильности работы пайплайна используется конкретная версия Python (3.9)

## Скрины

![image](https://github.com/user-attachments/assets/ce498863-bbc0-4774-a2e3-fa8a66a52dd0)

![image](https://github.com/user-attachments/assets/27b4347f-e856-441c-8bc8-ec18df227ac1)


## Заключение

CI/CD дейстивтельно интересная и полезная методика. Сатира про Васю хоть и пассивно-агрессивная, но актуальна, и, судя по всему, основана на реальной истории...
