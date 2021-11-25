# ScoutBert_microservices
ScoutBert microservices repository

Docker2

Docker хост в Yandex Cloud

yc compute instance create \
--name docker-host \
--zone ru-central1-a \
--network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
--create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1604-lts,size=15 \
--ssh-key ~/.ssh/yc-user.pub

docker-machine create \
--driver generic \
--generic-ip-address=51.250.11.187 \
--generic-ssh-user yc-user \
--generic-ssh-key ~/.ssh/yc-user \
docker-host

eval $(docker-machine env docker-host)

собрать свой образ
docker build -t reddit:latest .

запустить контейнер
docker run --name reddit -d --network=host reddit:latest

загрузить на docker hub
docker tag reddit:latest scoutberty/otus-reddit:1.0
docker push scoutberty/otus-reddit:1.0

запустить
docker run --name reddit -d -p 9292:9292 scoutberty/otus-reddit:1.0


Docker3

Скачать образ MongoDB
docker pull mongo:latest

Собрать образы
docker build -t scoutberty/post:1.0 ./post-py
docker build -t scoutberty/comment:1.0 ./comment
docker build -t scoutberty/ui:1.0 ./ui

Создать спец сеть
docker network create reddit

запустить контейнеры
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post scoutberty/post:1.0
docker run -d --network=reddit --network-alias=comment scoutberty/comment:1.0
docker run -d --network=reddit -p 9292:9292 scoutberty/ui:1.0

Создадm Docker volume:
docker volume create reddit_db

Удалить старые копии контейнеров
docker kill $(docker ps -q)

Запустить контейнеры с volume
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post scoutberty/post:1.0
docker run -d --network=reddit --network-alias=comment scoutberty/comment:1.0
docker run -d --network=reddit -p 9292:9292 scoutberty/ui:2.0
