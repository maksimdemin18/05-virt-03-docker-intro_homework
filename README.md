Для удобства зададим переменные:

export DOCKERHUB_USER="maksidemon"
export FULLNAME="DeminMaksim"

export LOCAL_IMAGE="custom-nginx:1.0.0"
export HUB_IMAGE="${DOCKERHUB_USER}/custom-nginx:1.0.0"

Пояснение:

DOCKERHUB_USER — имя пользователя Docker Hub.
FULLNAME — ФИО
LOCAL_IMAGE — локальное имя образа.
HUB_IMAGE — имя образа для публикации в Docker Hub.

## Задача 1

Сценарий выполнения задачи:
- Установите docker и docker compose plugin на свою linux рабочую станцию или ВМ.
- Если dockerhub недоступен создайте файл /etc/docker/daemon.json с содержимым: ```{"registry-mirrors": ["https://mirror.gcr.io", "https://daocloud.io", "https://c.163.com/", "https://registry.docker-cn.com"]}```
- Зарегистрируйтесь и создайте публичный репозиторий  с именем "custom-nginx" на https://hub.docker.com (ТОЛЬКО ЕСЛИ У ВАС ЕСТЬ ДОСТУП);
- скачайте образ nginx:1.29.0;
- Создайте Dockerfile и реализуйте в нем замену дефолтной индекс-страницы(/usr/share/nginx/html/index.html), на файл index.html с содержимым:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```
- Соберите и отправьте созданный образ в свой dockerhub-репозитории c tag 1.0.0 (ТОЛЬКО ЕСЛИ ЕСТЬ ДОСТУП).
- Предоставьте ответ в виде ссылки на https://hub.docker.com/<username_repo>/custom-nginx/general .


============================================================================================================


## решение:

# 1. Установка Docker и Docker Compose Plugin
```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```
Добавляем официальный GPG-ключ Docker:
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
Добавляем репозиторий Docker:
```
. /etc/os-release
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Устанавливаем Docker Engine и Compose plugin:
```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
[Проверяем установку:](png/001.png)
```
docker --version
docker compose version
sudo systemctl status docker --no-pager
```
Если текущий пользователь не входит в группу docker, добавить его:
```
sudo usermod -aG docker "$USER"
newgrp docker
```
[Проверка работы Docker:](png/002.png)
```
docker run --rm hello-world
```

# 2. [Создаем рабочую директорию и скачиваем базовый образ nginx:1.29.0](png/003.png)
```
mkdir -p ~/011_Виртуализация_и_контейнеризация/docker/task1-custom-nginx
cd ~/011_Виртуализация_и_контейнеризация/docker/task1-custom-nginx
```
```
docker pull nginx:1.29.0
1.29.0: Pulling from library/nginx
b1badc6e5066: Pull complete
c5ada5e7d698: Pull complete
85003794a6a5: Pull complete
856c000ad0ec: Pull complete
fea7cebc499c: Pull complete
dea1652b095a: Pull complete
9dbfe0b105c9: Pull complete
946b359b3ef2: Download complete
23dc44e5f024: Download complete
Digest: sha256:3ab4ed065a1437cbbd45e65617b1285bdf6523c6bf56a121e00df41720e09a89
Status: Downloaded newer image for nginx:1.29.0
docker.io/library/nginx:1.29.0
```
[И проверяем:](png/004.png)
```
docker images nginx
```

# 3. [Создание файла index.html](png/005.png)
```
nano index.html
```
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```
Проверяем содержимое:
```
cat index.html
```

# 4. [Создание Dockerfile](png/006.png)
```
nano Dockerfile
```
```
FROM nginx:1.29.0
COPY index.html /usr/share/nginx/html/index.html
```
Проверяем:
```
cat Dockerfile
```

# 5. [Сборка образа](png/007.png)
```
docker build -t custom-nginx:1.0.0 .
```
Проверяем наличие образа:
```
docker images | grep custom-nginx
```

# 6. [Локальная проверка образа](png/008.png)
```
docker run --rm -d --name test-custom-nginx -p 127.0.0.1:8080:80 custom-nginx:1.0.0
```
[Проверка страницы:](png/009.png)
```
curl http://127.0.0.1:8080
```
Удаляем тестовый контейнер:
```
docker rm -f test-custom-nginx
```

# 7. [Публикация в Docker Hub](png/015.png)
[Авторизация:](png/011.png)
```
docker login
```
Тегируем образ:
```
docker tag custom-nginx:1.0.0 "${HUB_IMAGE}"
```
[Публикуем:](png/013.png)
```
docker push "${HUB_IMAGE}"
```
[Проверяем:](png/014.png)
```
docker pull "${HUB_IMAGE}"
```


============================================================================================================


## Задача 2
1. Запустите ваш образ custom-nginx:1.0.0 командой docker run в соответвии с требованиями:
- имя контейнера "ФИО-custom-nginx-t2"
- контейнер работает в фоне
- контейнер опубликован на порту хост системы 127.0.0.1:8080
2. Не удаляя, переименуйте контейнер в "custom-nginx-t2"
3. Выполните команду ```date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; docker ps ; ss -tlpn | grep 127.0.0.1:8080  ; docker logs custom-nginx-t2 -n1 ; docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html```
4. Убедитесь с помощью curl или веб браузера, что индекс-страница доступна.

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.


============================================================================================================


## решение:

# 1. [Удалим старые контейнеры, если они остались после предыдущих попыток:](png/101.png)
```
docker rm -f "${FULLNAME}-custom-nginx-t2" custom-nginx-t2 2>/dev/null || true
```
Запускаем контейнер:
```
docker run -d --name "${FULLNAME}-custom-nginx-t2" -p 127.0.0.1:8080:80 custom-nginx:1.0.0
```

# 2. [Переименование контейнера](png/101.png)
```
docker rename "${FULLNAME}-custom-nginx-t2" custom-nginx-t2
```
Проверка:
```
docker ps
```

# 3. [Выполнение команды из задания](png/102.png)
```
date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; docker ps ; ss -tlpn | grep 127.0.0.1:8080 ; docker logs custom-nginx-t2 -n1 ; docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html
```
Ожидаемый base64 для правильного index.html:
```
28-05-2026 18:59:09.036760005 MSK
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                    NAMES
bb2b0ef78f30   custom-nginx:1.0.0   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   127.0.0.1:8080->80/tcp   custom-nginx-t2
LISTEN 0      4096       127.0.0.1:8080      0.0.0.0:*
2026/05/28 15:56:46 [notice] 1#1: start worker process 44
PGh0bWw+CjxoZWFkPgpIZXksIE5ldG9sb2d5CjwvaGVhZD4KPGJvZHk+CjxoMT5JIHdpbGwgYmUg
RGV2T3BzIEVuZ2luZWVyITwvaDE+CjwvYm9keT4KPC9odG1sPgo=
```

# 4. [Проверка через curl](png/102.png)
```
curl -i http://127.0.0.1:8080
```


============================================================================================================


## Задача 3
1. Воспользуйтесь docker help или google, чтобы узнать как подключиться к стандартному потоку ввода/вывода/ошибок контейнера "custom-nginx-t2".
2. Подключитесь к контейнеру и нажмите комбинацию Ctrl-C.
3. Выполните ```docker ps -a``` и объясните своими словами почему контейнер остановился.
4. Перезапустите контейнер
5. Зайдите в интерактивный терминал контейнера "custom-nginx-t2" с оболочкой bash.
6. Установите любимый текстовый редактор(vim, nano итд) с помощью apt-get.
7. Отредактируйте файл "/etc/nginx/conf.d/default.conf", заменив порт "listen 80" на "listen 81".
8. Запомните(!) и выполните команду ```nginx -s reload```, а затем внутри контейнера ```curl http://127.0.0.1:80 ; curl http://127.0.0.1:81```.
9. Выйдите из контейнера, набрав в консоли  ```exit``` или Ctrl-D.
10. Проверьте вывод команд: ```ss -tlpn | grep 127.0.0.1:8080``` , ```docker port custom-nginx-t2```, ```curl http://127.0.0.1:8080```. Кратко объясните суть возникшей проблемы.
11. * Это дополнительное, необязательное задание. Попробуйте самостоятельно исправить конфигурацию контейнера, используя доступные источники в интернете. Не изменяйте конфигурацию nginx и не удаляйте контейнер. Останавливать контейнер можно. [пример источника](https://www.baeldung.com/linux/assign-port-docker-container)
12. Удалите запущенный контейнер "custom-nginx-t2", не останавливая его.(воспользуйтесь --help или google)

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.


============================================================================================================


## решение:


# 1. [Подключение к стандартным потокам контейнера](png/201.png)
```
docker attach custom-nginx-t2
```
После подключения нажать:
```
Ctrl-C
```
docker attach подключает терминал к стандартным потокам ввода, вывода и ошибок основного процесса контейнера. При нажатии Ctrl-C сигнал передаётся процессу контейнера, из-за чего контейнер может завершиться. Официальная документация Docker описывает это поведение для docker attach.

# 2. [Проверка, что контейнер остановился](png/202.png)
```
docker ps -a
```

# 3. [Перезапуск контейнера](png/203.png)
```
docker start custom-nginx-t2
```
Проверка:
```
docker ps
```

# 4. [Вход в интерактивный терминал контейнера](png/204.png)
```
docker exec -it custom-nginx-t2 bash
```
Установка текстового редактора и curl
```
apt-get update
apt-get install -y nano curl
```

# 5. [Замена порта listen 80 на listen 81](png/205.png)
```
/etc/nginx/conf.d/default.conf
```
[Перезагрузка конфигурации Nginx](png/206.png)
```
nginx -s reload
```

# 6. [Проверка портов внутри контейнера](png/206.png)
```
curl http://127.0.0.1:80
```

# 7.[Проверка с хостовой ОС](png/207.png)
```
ss -tlpn | grep 127.0.0.1:8080
docker port custom-nginx-t2
curl http://127.0.0.1:8080
```

# 7.1 [Дополнительное исправление без изменения Nginx и без удаления контейнера](png/210.png)

Без изменения конфигурации Nginx можно запустить внутри контейнера TCP-прокси с порта 80 на порт 81.
```
docker exec -it custom-nginx-t2 bash
```
Внутри контейнера:
```
apt-get update
apt-get install -y socat
nohup socat TCP-LISTEN:80,fork,reuseaddr TCP:127.0.0.1:81 >/tmp/socat-80-to-81.log 2>&1 &
exit
```
[Проверка с хоста:](png/211.png)
```
curl http://127.0.0.1:8080
```

# 8 Удаление запущенного контейнера без предварительной остановки
```
docker rm -f custom-nginx-t2
```


============================================================================================================


## Задача 4


- Запустите первый контейнер из образа ***centos*** c любым тегом в фоновом режиме, подключив папку  текущий рабочий каталог ```$(pwd)``` на хостовой машине в ```/data``` контейнера, используя ключ -v.
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив текущий рабочий каталог ```$(pwd)``` в ```/data``` контейнера.
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```.
- Добавьте ещё один файл в текущий каталог ```$(pwd)``` на хостовой машине.
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.


В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод.


============================================================================================================


## решение:


# 1. [Создание рабочей директории](png/301.png)
```
mkdir -p task4
cd task4
```
# 2. [Запуск контейнера CentOS](png/302.png)
```
docker run -d --name centos-t4 -v "$(pwd)":/data centos:7 sleep infinity
```
Проверяем
```
docker ps
CONTAINER ID   IMAGE      COMMAND            CREATED          STATUS          PORTS     NAMES
390fc7874a41   centos:7   "sleep infinity"   22 seconds ago   Up 21 seconds             centos-t4

```
# 3. [Запуск контейнера Debian](png/305.png)
```
docker run -d --name debian-t4 -v "$(pwd)":/data debian:12-slim sleep infinity
```
Проверяем
```
docker ps
CONTAINER ID   IMAGE            COMMAND            CREATED         STATUS         PORTS     NAMES
a180a54cbb6c   debian:12-slim   "sleep infinity"   6 seconds ago   Up 5 seconds             debian-t4
390fc7874a41   centos:7         "sleep infinity"   6 minutes ago   Up 6 minutes             centos-t4
```

# 4. [Создание файла из контейнера CentOS](png/303.png)
```
docker exec -it centos-t4 bash
nano /data/from_centos.txt
cat /data/from_centos.txt
exit
```

# 5. [Создание файла на хостовой машине](png/304.png)
```
echo "File created on host system" > from_host.txt
```
Проверка на хосте:
```
ls -la
cat from_centos.txt
cat from_host.txt
```

# 6. [Проверка файлов из контейнера Debian](png/305.png)
```
docker exec -it debian-t4 bash
ls -la /data
cat /data/from_centos.txt
cat /data/from_host.txt
exit
```
Оба контейнера используют один и тот же каталог хостовой системы, подключённый в /data.
Поэтому файл, созданный в контейнере CentOS, виден на хосте и в контейнере Debian.
Файл, созданный на хосте, также виден в обоих контейнерах.

# 7. Очистка после задачи
```
docker rm -f centos-t4 debian-t4
```


============================================================================================================


## Задача 5

1. Создайте отдельную директорию(например /tmp/netology/docker/task5) и 2 файла внутри него.
"compose.yaml" с содержимым:
```
version: "3"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
"docker-compose.yaml" с содержимым:
```
version: "3"
services:
  registry:
    image: registry:2

    ports:
    - "5000:5000"
```

И выполните команду "docker compose up -d". Какой из файлов был запущен и почему? (подсказка: https://docs.docker.com/compose/compose-application-model/#the-compose-file )

2. Отредактируйте файл compose.yaml так, чтобы были запущенны оба файла. (подсказка: https://docs.docker.com/compose/compose-file/14-include/)

3. Выполните в консоли вашей хостовой ОС необходимые команды чтобы залить образ custom-nginx как custom-nginx:latest в запущенное вами, локальное registry. Дополнительная документация: https://distribution.github.io/distribution/about/deploying/
4. Откройте страницу "https://127.0.0.1:9000" и произведите начальную настройку portainer.(логин и пароль адмнистратора)
5. Откройте страницу "http://127.0.0.1:9000/#!/home", выберите ваше local  окружение. Перейдите на вкладку "stacks" и в "web editor" задеплойте следующий компоуз:

```
version: '3'

services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
```
6. Перейдите на страницу "http://127.0.0.1:9000/#!/2/docker/containers", выберите контейнер с nginx и нажмите на кнопку "inspect". В представлении <> Tree разверните поле "Config" и сделайте скриншот от поля "AppArmorProfile" до "Driver".

7. Удалите любой из манифестов компоуза(например compose.yaml).  Выполните команду "docker compose up -d". Прочитайте warning, объясните суть предупреждения и выполните предложенное действие. Погасите compose-проект ОДНОЙ(обязательно!!) командой.

В качестве ответа приложите скриншоты консоли, где видно все введенные команды и их вывод, файл compose.yaml , скриншот portainer c задеплоенным компоузом.


============================================================================================================


## решение:


# 1. [Создание директории](png/401.png)
```
mkdir -p /tmp/netology/docker/task5
cd /tmp/netology/docker/task5
```

# 2. [Создание compose.yaml](png/401.png)
```
nano compose.yaml
version: "3"

services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

# 3. [Создание docker-compose.yaml](png/402.png)
```
nano docker-compose.yaml
version: "3"

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
```

# 4. [Первый запуск Compose](png/403.png)
```
docker compose up -d
```
Проверка:
```
docker compose ps
```
Docker Compose выбирает compose.yaml, потому что это каноническое и предпочтительное имя файла.
Поэтому на первом запуске будет запущен только сервис portainer, а сервис registry из docker-compose.yaml не будет запущен.

# 5. Изменение compose.yaml, чтобы запускались оба файла

[Редактируем compose.yaml и добавляем include:](png/404.png)
```
nano compose.yaml
include:
  - docker-compose.yaml

services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
Проверяем итоговую конфигурацию:
```
docker compose config
```
[Запускаем:](png/405.png)
```
docker compose up -d
```
Проверяем:
```
docker compose ps
NAME                IMAGE                           COMMAND                  SERVICE     CREATED         STATUS         PORTS
task5-portainer-1   portainer/portainer-ce:latest   "/portainer"             portainer   2 minutes ago   Up 2 minutes
task5-registry-1    registry:2                      "/entrypoint.sh /etc…"   registry    6 seconds ago   Up 6 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp
```
Директива include подключает внешний Compose-файл к основному compose.yaml.

# 6. Загрузка образа custom-nginx в локальный registry

[Проверяем, что registry доступен:](png/406.png)
```
curl http://127.0.0.1:5000/v2/
```
Тегируем локальный образ:
```
docker tag custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx:latest
```
Отправляем образ в локальный registry:
```
docker push 127.0.0.1:5000/custom-nginx:latest
```
Проверяем каталог registry:
```
curl http://127.0.0.1:5000/v2/_catalog
{"repositories":["custom-nginx"]}
```
Проверяем теги:
```
curl http://127.0.0.1:5000/v2/custom-nginx/tags/list
{"name":"custom-nginx","tags":["latest"]}
```
# 7. Настройка Portainer

[Открыть в браузере:](png/407.png)
```
https://127.0.0.1:9443
```
Далее:

Создать администратора.
Задать логин.
Задать пароль.
Выбрать local environment.

# 8. Деплой Stack в Portainer

Открыть:
```
http://127.0.0.1:9000/#!/home
```
Далее:

Выбрать local environment.
Перейти в Stacks.
Нажать Add stack.
Выбрать Web editor.
[Вставить Compose:](png/408.png)
```
version: '3'

services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
```
Назвать stack, например:
custom-nginx-stack
Нажать Deploy the stack.

[Проверка с хоста:](png/410.png)
```
curl http://127.0.0.1:9090
```
Вывод:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```
9. [Inspect в Portainer](png/410.png)

Открыть:
```
http://127.0.0.1:9000/#!/2/docker/containers
```
Далее:

Выбрать контейнер с Nginx.
Нажать Inspect.
Переключиться в режим Tree.
Развернуть поле Config.
Сделать скриншот от поля AppArmorProfile до поля Driver.

10. Удаление одного из Compose-манифестов и анализ warning

[Удалим compose.yaml:](png/413.png)
```
rm compose.yaml
```
[Выполним:](png/414.png)
```
docker compose up -d
```
Получим
```
Found orphan containers ...
```

Предупреждение появляется потому, что ранее Compose-проект запускался по конфигурации, где были сервисы portainer и registry.
После удаления compose.yaml Docker Compose видит только оставшийся файл docker-compose.yaml, где описан только сервис registry.
Контейнер portainer, созданный прежней конфигурацией, больше не описан в текущей Compose-модели, поэтому Docker Compose считает его orphan container, то есть «осиротевшим» контейнером.
Выполняем предложенное действие:
```
docker compose up -d --remove-orphans
```
11. [Погасить Compose-проект одной командой](png/415.png)
```
docker compose down --remove-orphans
```
