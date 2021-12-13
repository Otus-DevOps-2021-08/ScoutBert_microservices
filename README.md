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

docker-compose

Выполнить:
export USERNAME=scoutberty
docker-compose up -d
docker-compose ps

CI-CD

Добавить remote
> git checkout -b gitlab-ci-1
> git remote add gitlab http://51.250.4.177/homework/example.git

Установить раннер внутри докера
> docker run -d --name gitlab-runner --restart always -v /srv/gitlabrunner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest

Зарегистрировать раннер
> docker exec -it gitlab-runner gitlab-runner register \
--url http://51.250.4.177/ \
--non-interactive \
--locked=false \
--name DockerRunner \
--executor docker \
--docker-image alpine:latest \
--registration-token r4cGznxa45P5wx_B9CHo \
--tag-list "linux,xenial,ubuntu,docker" \
--run-untagged

Добавить исходный код reddit в репозиторий:
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m "Add reddit app"
git push gitlab gitlab-ci-1

monitoring-1

Создадим Docker хост в Yandex Cloud

yc compute instance create \
  --name docker-host \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1604-lts,size=15 \
  --ssh-key ~/.ssh/yc-user.pub

docker-machine create \
  --driver generic \
  --generic-ip-address=51.250.9.91 \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/yc-user \
  docker-host

eval $(docker-machine env docker-host)

Запуск Prometheus
docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus

собираем Docker образ prometheus

export USER_NAME=username
docker build -t scoutberty/prometheus .

Запушить образы
docker push scoutberty/ui
docker push scoutberty/comment
docker push scoutberty/post
docker push scoutberty/prometheus

https://hub.docker.com/layers/scoutberty/comment/latest/images/sha256:082dfdbe719d232cd26b2d3c53b7f48a5f12fe0979a23c34690263ed2bddf301

https://hub.docker.com/layers/scoutberty/prometheus/latest/images/sha256:9506f46e09b01cac95cafeb16e0869529557007eaf3346a36b336a1e150ae456

https://hub.docker.com/layers/scoutberty/ui/latest/images/sha256:f226cc5613d2c2e3e992b6ca2f1f920d663633eea739db20ec8af28d74c3aa67

https://hub.docker.com/layers/scoutberty/post/latest/images/sha256:c636d0605418864908cb010615bc04f431f24aa39f5b6b17f187c9bec916a9f8
