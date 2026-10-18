DockerHub - https://hub.docker.com/repository/docker/bezculturshina/image-lab/tags

docker history 
```
IMAGE          CREATED             CREATED BY                                      SIZE      COMMENT
6dbfe5659b5f   About an hour ago   CMD ["python" "-m" "app.app"]                   0B        buildkit.dockerfile.v0
<missing>      About an hour ago   HEALTHCHECK {Test:[CMD-SHELL python -c "impo…   0B        buildkit.dockerfile.v0
<missing>      About an hour ago   EXPOSE [8000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      About an hour ago   USER appuser                                    0B        buildkit.dockerfile.v0
<missing>      About an hour ago   COPY --chown=appuser:appuser app ./app # bui…   1.07kB    buildkit.dockerfile.v0
<missing>      About an hour ago   COPY /opt/venv /opt/venv # buildkit             15.9MB    buildkit.dockerfile.v0
<missing>      2 hours ago         RUN |2 LAB_LOGIN=bezculturshina LAB_TOKEN=30…   4.37kB    buildkit.dockerfile.v0
<missing>      2 hours ago         ENV PATH=/opt/venv/bin:/usr/local/bin:/usr/l…   0B        buildkit.dockerfile.v0
<missing>      2 hours ago         LABEL org.lab.login=bezculturshina org.lab.t…   0B        buildkit.dockerfile.v0
<missing>      2 hours ago         ARG LAB_TOKEN=30ee96f018f6ac6e15a7da28031272…   0B        buildkit.dockerfile.v0
<missing>      2 hours ago         ARG LAB_LOGIN=bezculturshina                    0B        buildkit.dockerfile.v0
<missing>      2 hours ago         WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      12 days ago         CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      12 days ago         RUN /bin/sh -c set -eux;  for src in idle3 p…   36B       buildkit.dockerfile.v0
<missing>      12 days ago         RUN /bin/sh -c set -eux;   savedAptMark="$(a…   36.8MB    buildkit.dockerfile.v0
<missing>      12 days ago         ENV PYTHON_SHA256=c2c4321961fab0fb999d66e0ce…   0B        buildkit.dockerfile.v0
<missing>      12 days ago         ENV PYTHON_VERSION=3.12.15                      0B        buildkit.dockerfile.v0
<missing>      12 days ago         ENV GPG_KEY=7169605F62C751356D054A26A821E680…   0B        buildkit.dockerfile.v0
<missing>      12 days ago         RUN /bin/sh -c set -eux;  apt-get update;  a…   3.81MB    buildkit.dockerfile.v0
<missing>      12 days ago         ENV LANG=C.UTF-8                                0B        buildkit.dockerfile.v0
<missing>      12 days ago         ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      13 days ago         # debian.sh --arch 'amd64' out/ 'trixie' '@1…   78.8MB    debuerreotype 0.17
```


Преимущества перед наивный сборкой
1. Многоэтапная сборка
Использование multi-stage build позволяет устанавливать зависимости на отдельном этапе и переносить в итоговый образ только необходимое виртуальное окружение
```
FROM python:3.12-slim AS builder
...
FROM python:3.12-slim
...
COPY --from=builder /opt/venv /opt/venv
```
2. Использование облегчённого python:3.12-slim для минимизации размера
В качестве базового образа используется python:3.12-slim, что позволяет уменьшить итоговый размер образа 
3. Запуск приложения от некорневого пользователя
Приложение запускается от пользователя appuser, а не от root, что повышает безопасность контейнера
 ```
 RUN useradd -U -s /sbin/nologin --no-create-home appuser
```
4. Настройка переменной окружения без пересборки
Переменная ROCKET_SIZE позволяет изменять ответ приложения без пересборки Docker-образа. По умолчанию используется значение Small, а при запуске с -e ROCKET_SIZE=Medium приложение вернёт
```
Hello from Docker! Rocket=Medium
```
5. Самодиагностика через healthcheck
В Dockerfile настроена автоматическая проверка маршрута /health, позволяющая Docker определять работоспособность приложения.
```
   HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health', timeout=2)" || exit 1
```
