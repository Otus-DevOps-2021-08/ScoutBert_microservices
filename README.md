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
--generic-ip-address=51.250.14.51 \
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
