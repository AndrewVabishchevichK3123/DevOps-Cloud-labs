# Лабораторная работа 2

В этой лабе будем создавать "хорошие" и "плохие" Dockerfiles, описывать плохие практики, исправлять их и обосновывать что к чему

## Пример "плохого" Dockerfile

```
# использование последней версии(!)
FROM ubuntu:latest

# установка всех(!) зависимостей в одном RUN
RUN apt-get update && apt-get install -y python3 python3-pip git

# копируем всю директорию, включая ненужные(!) файлы
COPY . /app

# используем права суперпользователя(!)
USER root

# запуск команды в фоновом(!) процессе
CMD python3 /app/app.py &

```

### Что плохого то?
* Использование образа latest: это может привести к непредсказуемым ошибкам и поведению, так как образ может измениться в будущем и негативно повлиять на работу контейнера

* Установка зависимостей в одном RUN: длинные команды сильно затрудняют отладку, поэтому в экстренной ситуации придется перезапускать всю команду

* Копирование всех файлов с помощью COPY: использование этой команды копирует ненужные файлы (в том числе), что увеличивает размер образа и усложняет сборку

## Пример "хорошего" Dockerfile

```
# использование конкретной весрии
FROM ubuntu:20.04

# обновление пакетов и установка зависимостей
RUN apt-get update && apt-get install -y python3 python3-pip

# установка необходимых зависимостей через pip
RUN pip3 install -r requirements.txt

# копируем только нужные файлы
COPY app /app

# создаем и используем непривилегированного пользователя
RUN useradd -m appuser
USER appuser

# запуск приложения
CMD ["python3", "/app/app.py"]

```
### Что улучшили?

* Убран образ latest: в "хорошем" Dockerfile указана конкретная версия образа (ubuntu:20.04), для обеспечения стабильности и предсказуемости

* RUN разделили на несколько команд, чтобы каждая выполнялась поэтапно и было легче отлавливать ошибки

* Копирование всех файлов с помощью COPY: в исправленном Dockerfile указано, какие файлы копировать, используя .dockerignore, чтобы исключить ненужные файлы


##  Плохие практики при работе с контейнерами

* Использование контейнеров с правами суперпользователя (root):
  
  Привилегированные контейнеры имеют полный доступ к системе хоста, что создает серьезные риски безопасности. Вместо этого лучше использовать минимальные права, необходимые для выполнения задачи
  

* Создание образов с большим количеством данных:

  Большой образ будет труднее распространить. Необходимо убедиться в наличии только необходимых файлов и библиотек для запуска приложения/процесса. Не устанавливать ненужные пакеты и не запускать обновления (yum update), которые загружают много файлов на новый слой образа
  

# Лабораторная работа 2 со звездочкой

## Пример "плохого" Docker Compose файла

```
version: '3'
services:
  app:
    build: .
    environment: 
      - DEBUG=true
    ports:
      - "80:8000" (-)
    volumes:
      - ./app:/app
    depends_on:
      - db
  db:
    image: postgres:latest (-)
    environment:
      - POSTGRES_PASSWORD=00000000 (-)

```

* Использование latest для базы данных: это приводит к нестабильности при обновлении образа

* Неуправляемое использование переменных окружения: переменные окружения, такие как пароли, не должны быть заданы напрямую в файле

* Прямое использование портов хоста: открытие порта напрямую на хосте увеличивает риски безопасности

## Пример "хорошего" Docker Compose файла

```
version: '3.8'
services:
  app:
    build: .
    environment:
      - DEBUG=false
    ports:
      - "8000:8000" (+)
    volumes:
      - ./app:/app
    depends_on:
      - db
  db:
    image: postgres:13 (+)
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password (+)
    secrets:
      - postgres_password
    networks:
      - internal

networks:
  internal:
    driver: bridge

secrets:
  postgres_password:
    file: ./secrets/postgres_password

```

* Исправлено использование версии образа: указана конкретная версия для обеспечения стабильности

* Исправлено управление переменными окружения: чувствительные данные вынесены в файл секретов для лучшей безопасности.

* Исправлено использование сети: введена внутренняя сеть, контейнеры не открыты напрямую для внешнего доступа


  Итак, в конечном счете мы создали внутреннюю сеть Docker для связи между сервисами и изолировали контейнеры.
  
  Для того чтобы контейнеры не "видели" друг друга по сети, была настроена внутренняя сеть internal, и контейнеры связываются только через неё. Это ограничивает доступ к контейнерам извне и между ними, улучшая безопасность и контроль сети
