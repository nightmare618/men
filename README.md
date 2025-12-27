Лабораторная работа №3: Docker и контейнеризация
Выполнил: Лаптий Никита Витальевич
Группа: ИЭ-23
Дата: 27.12.2025

Задача 1: Создание кастомного образа nginx
Цель:
Создать кастомный образ nginx с изменённой индексной страницей и загрузить его в Docker Hub.
Выполненные действия:
1.	Установлены Docker и Docker Compose на ВМ Debian:
sudo apt update
sudo apt install docker.io docker-compose-plugin -y
2.	Создан файл /etc/docker/daemon.json для настройки зеркал репозиториев.
3.	Зарегистрирован репозиторий custom-nginx на Docker Hub.
4.	Скачан образ nginx:1.21.1:
docker pull nginx:1.21.1
5.	Создан Dockerfile:
FROM nginx:1.21.1
COPY index.html /usr/share/nginx/html/index.html
6.	Создан файл index.html с содержимым:
<html>
<head>Hey, ZGU!</head>
<body><p>I will be IT Engineer!</p></body>
</html>
7.	Образ собран и загружен в Docker Hub:

docker build -t l3n1n414/custom-nginx:1.0.0 .
docker push l3n1n414/custom-nginx:1.0.0
Результат:
•	Ссылка на репозиторий: https://hub.docker.com/r/l3n1n414/custom-nginx

Задача 2: Запуск и проверка контейнера
Цель:
Запустить контейнер из созданного образа, переименовать его и проверить работоспособность.
Выполненные команды:
# Запуск контейнера
docker run -d --name LaptiyNikita-custom-nginx-t2 -p 127.0.0.1:8080:80 l3n1n414/custom-nginx:1.0.0

# Переименование
docker rename LaptiyNikita-custom-nginx-t2 custom-nginx-t2

# Проверка работы
date +"%d-%m-%Y %T.%N %Z"
sleep 0.150
docker ps
ss -tlpn | grep 127.0.0.1:8080
docker logs custom-nginx-t2 -n1
docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html

# Проверка через curl
curl http://127.0.0.1:8080




Результат:
•	Контейнер запущен, переименован, работает на порту 8080.
•	Веб-страница доступна по http://127.0.0.1:8080.

Задача 3: Работа с запущенным контейнером
Цель:
Изучить управление контейнером, изменить его конфигурацию и проанализировать проблемы.
Выполненные действия:
1.	Подключение к потоку вывода контейнера:
docker attach custom-nginx-t2
Нажатие Ctrl+C остановило контейнер, потому что сигнал прерывания был передан процессу nginx.
2.	Перезапуск контейнера:
docker start custom-nginx-t2
3.	Вход в контейнер и изменение конфигурации nginx:
docker exec -it custom-nginx-t2 bash
apt update && apt install nano -y
nano /etc/nginx/conf.d/default.conf
# Замена порта 80 на 81
nginx -s reload
curl http://127.0.0.1:80
curl http://127.0.0.1:81
exit
4.	Проверка доступности снаружи:
ss -tlpn | grep 127.0.0.1:8080
docker port custom-nginx-t2
curl http://127.0.0.1:8080
Проблема: После изменения порта nginx на 81, проброс порта с хоста 8080 на 80 перестал работать, потому что nginx слушает теперь только порт 81.
5.	Удаление контейнера без остановки:
docker rm -f custom-nginx-t2

Задача 4: Работа с volume
Цель:
Продемонстрировать работу с томами Docker, создав общую папку между контейнерами и хостом.
Выполненные команды:
# Запуск контейнеров с volume
docker run -d -v $(pwd):/data --name centos-container centos:7 sleep infinity
docker run -d -v $(pwd):/data --name debian-container debian:11 sleep infinity

# Создание файла в контейнере CentOS
docker exec centos-container touch /data/from-centos.txt

# Создание файла на хосте
echo "Hello from host" > host-file.txt

# Проверка файлов в контейнере Debian
docker exec debian-container ls -la /data/
docker exec debian-container cat /data/host-file.txt
Результат:
•	Файлы, созданные в контейнерах и на хосте, доступны в обоих контейнерах через общую папку /data.





Задача 5: Docker Compose и Portainer
Цель:
Настроить Docker Compose, локальный реестр образов и управление через Portainer.
Выполненные действия:
1.	Созданы два файла композиции:
o	compose.yaml (Portainer)
o	docker-compose.yaml (Registry)
2.	Запуск docker compose up -d запустил compose.yaml, потому что Docker Compose отдаёт приоритет этому имени файла.
3.	Редактирование compose.yaml для включения обоих сервисов:
version: "3"
include:
  - docker-compose.yaml
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
4.	Загрузка образа в локальный registry:
docker tag yourusername/custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx:latest
docker push 127.0.0.1:5000/custom-nginx:latest
5.	Настройка Portainer через веб-интерфейс https://127.0.0.1:9000.
6.	Деплой стека через Portainer:
version: '3'
services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
7.	Проверка контейнера через Portainer (скриншот вкладки "Inspect").
8.	Удаление одного из файлов композиции и повторный запуск:
docker compose up -d
Предупреждение: Docker Compose обнаружил, что файл compose.yaml включает docker-compose.yaml, но один из них отсутствует.
Решение: восстановить удалённый файл или обновить compose.yaml.
9.	Остановка проекта одной командой:
docker compose down
Результат:
•	Локальный registry работает на порту 5000.
•	Portainer управляет окружением через веб-интерфейс.
•	Стек nginx развёрнут и доступен на порту 9090.

Выводы:
В ходе работы были освоены:
•	Создание кастомных Docker-образов.
•	Управление контейнерами (запуск, переименование, изменение конфигурации).
•	Работа с томами для обмена данными.
•	Использование Docker Compose для оркестрации.
•	Настройка локального registry и Portainer для управления контейнерами.


