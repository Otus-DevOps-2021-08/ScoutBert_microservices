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

Docker4

Запустить контейнер с использованием none-драйвера
docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig

Запустить контейнер в сетевом пространстве docker-хоста
docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig

Создадь bridge-сеть в docker
docker network create reddit --driver bridge

Запустить наш проект reddit с использованием bridge-сети
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post scoutberty/post:1.0
docker run -d --network=reddit --network-alias=comment  scoutberty/comment:1.0
docker run -d --network=reddit -p 9292:9292 scoutberty/ui:1.0

Запустить  проект в 2-х bridge сетях.
Создать docker-сети
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24

Запустить контейнеры
docker run -d --network=front_net -p 9292:9292 --name ui  scoutberty/ui:1.0
docker run -d --network=back_net --name comment  scoutberty/comment:1.0
docker run -d --network=back_net --name post  scoutberty/post:1.0
docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest

Docker при инициализации контейнера может подключить к нему только 1 сеть
По этому необходимо подключить контейнеры ко второй сети
docker network connect front_net post
docker network connect front_net comment
