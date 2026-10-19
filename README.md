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



Команда запуска с --read-only и проверки эндпоинтов
```
PS C:\Users\user\PycharmProjects\PythonProject3> docker run -d --name flask-lab --read-only `
>>   -e ROCKET_SIZE=Big `
>>   -e "LAB_LOGIN=$LOGIN" `
>>   -e "LAB_TOKEN=$TOKEN" `
>>   -e "SECRET=$SECRET" `
>>   -p 8080:8000 `
>>   bezculturshina/image-lab:30ee96f0
23da4cfa5210c3bc1db4dafee4b5e9683b08a6779f439c31efc0e28d4c851376
PS C:\Users\user\PycharmProjects\PythonProject3> curl.exe -s http://localhost:8080/
Hello from Docker! Rocket=Big
PS C:\Users\user\PycharmProjects\PythonProject3> curl.exe -s http://localhost:8080/health
{"status":"ok","uptime":31.58}
PS C:\Users\user\PycharmProjects\PythonProject3> curl.exe -s http://localhost:8080/proof
{"login":"bezculturshina","ok":true,"proof":"0c2cff1b5223fa9df526d93e262a3e91d99f0bc378dad018e42d14e6d48156c6"}
PS C:\Users\user\PycharmProjects\PythonProject3> docker inspect --format '{{.HostConfig.ReadonlyRootfs}}' flask-lab
true
PS C:\Users\user\PycharmProjects\PythonProject3> docker inspect --format '{{.State.Health.Status}}' flask-lab
healthy
```


Проверка SIGTERM
```
PS C:\Users\user\PycharmProjects\PythonProject3> docker stop -t 10 flask-lab
flask-lab
PS C:\Users\user\PycharmProjects\PythonProject3> docker inspect --format '{{.State.ExitCode}}' flask-lab
0
PS C:\Users\user\PycharmProjects\PythonProject3> docker logs flask-lab
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://172.17.0.2:8000
Press CTRL+C to quit
127.0.0.1 - - [19/Oct/2026 17:04:22] "GET /health HTTP/1.1" 200 -
172.17.0.1 - - [19/Oct/2026 17:04:41] "GET / HTTP/1.1" 200 -
172.17.0.1 - - [19/Oct/2026 17:04:50] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:04:52] "GET /health HTTP/1.1" 200 -
172.17.0.1 - - [19/Oct/2026 17:04:59] "GET /proof HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:05:22] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:05:52] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:06:23] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:06:53] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:07:23] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [19/Oct/2026 17:07:53] "GET /health HTTP/1.1" 200 -
Shutting down...
```

Вес образа в битах:
```
PS C:\Users\user\PycharmProjects\PythonProject3> docker image inspect bezculturshina/image-lab:30ee96f0 --format '{{.Size}}'
135313962
```
135313962 байт примерно равно 135 Мб
<img width="2219" height="99" alt="{BC0B826A-C10D-4F47-8937-A02637301BAD}" src="https://github.com/user-attachments/assets/00fa00b1-f493-4685-b651-4dd81d118e0e" />


Для выполнения заданий был выбран встроенный сервер Flask так как в ходе работы нужно было проводить разработку и тестирование (многоэтапная сборка, healthcheck, режим --read-only, запуск от непревилигированного пользователя, корректное завершение контейнера), для этих проверок встроенного сервера достаточно. 
Gunicorn больше предназначен для развёртывания. Отказ от Gunicorn уменьшает количество зависимостей, что уменьшает размер итогового образа.

ARG - переменная для передачи значений во время сборки образа
ENV - переменная окружения контейнера, её значения сохраняются в образе и доступны во время работы контейнера
LABEL - дополнительная информация, описывающая образ

/proof для вывода использует переменные окружения (env)

