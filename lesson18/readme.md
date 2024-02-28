# Урок 17 - etcd
# Урок 18 - Consul


# Домашнее задание

DCS

## Цель:

В результате выполнения ДЗ вы развернете отказоустойчивый DCS.


- Описание/Пошаговая инструкция выполнения домашнего задания:
- Разворачиваем кластер Etcd любым способом. Проверяем отказоустойчивость
- Разворачиваем кластер Consul любым способом. Проверяем отказоустойчивость


## Описание/Пошаговая инструкция выполнения домашнего задания:

## Consul

###Подготовительные работы

#### Устанавливаем docker
```bash
sudo snap install docker
```

#### Устанавливаем Consul
Инструкция: https://developer.hashicorp.com/consul/tutorials/docker/docker-compose-datacenter?in=consul%2Fdocker

```bash
mkdir consul
cd consul/
git clone https://github.com/hashicorp/learn-consul-docker.git
cd learn-consul-docker/datacenter-deploy-secure
```

Конфигурационные файлы:
```bash
docker-compose.yml
server1.json
server2.json
server3.json
client.json
```

Create the Consul datacenter
```bash
docker-compose up --detach
```

Результат:

```bash
vagrant@ubuntu2204:~/consul/learn-consul-docker/datacenter-deploy-secure$ sudo docker-compose up --detach
[+] Running 10/10
 ✔ consul-server3 6 layers [⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                      11.6s
   ✔ 5758d4e389a3 Pull complete                                                                                                                  1.5s
   ✔ 893bc6280f09 Pull complete                                                                                                                  1.8s
   ✔ a2874d28cf40 Pull complete                                                                                                                  3.4s
   ✔ 9f9456e01a30 Pull complete                                                                                                                  2.7s
   ✔ 763d944e188d Pull complete                                                                                                                  3.5s
   ✔ bca03003813c Pull complete                                                                                                                  3.7s
 ✔ consul-server1 Pulled                                                                                                                        11.7s
 ✔ consul-client Pulled                                                                                                                         11.7s
 ✔ consul-server2 Pulled                                                                                                                        11.7s
[+] Running 5/5
 ✔ Network datacenter-deploy-secure_consul  Created                                                                                              0.1s
 ✔ Container consul-server3                 Started                                                                                              1.6s
 ✔ Container consul-client                  Started                                                                                              1.6s
 ✔ Container consul-server1                 Started                                                                                              1.6s
 ✔ Container consul-server2                 Started                                                                                              1.6s
vagrant@ubuntu2204:~/consul/learn-consul-docker/datacenter-deploy-secure$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.4:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.2:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul/learn-consul-docker/datacenter-deploy-secure$
```

(Веб консоль: )[http://192.168.1.22:8500/ui/dc1/services]

Подключаемся к контейнеру и смотрим состояние:

```bash
sudo docker exec consul-client consul members
# or
sudo docker exec -it consul-client /bin/sh
consul members
```
