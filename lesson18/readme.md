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

## etcd

Устанавливаем etcd с исходников

```bash
sudo apt-get update
sudo apt install golang-go
git clone -b v3.5.11 https://github.com/etcd-io/etcd.git
cd etcd
./build.sh
```
 **Устанровка завершилась с ошибкой**

Пробуем установить из пакета

```bash
sudo apt install etcd-server etcd-client

```

Установка завершена успешно

т.к. к уроку не приложена ни презентация, ни выполняемые скрипты для примера (о чем писал в чате, но ситуация не изменилась), то на этом этапе все. Далее Consul.



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
consul operator raft list-peers
```

#### Настраиваем агента

на 1-3-м сервере (рабочий вариант)
```bash
echo '{
        "service": {
            "id": "web",
            "name": "web",
            "tags": [ "consul" ],
            "port": 80,
            "enable_tag_override": false,
            "check": {
                "id": "web_up",
                "name": "consul healthcheck",
                "tcp": "localhost:8500",
                "interval": "10s",
                "timeout": "2s"
            }
        },
       "enable_script_checks": true,
       "enable_local_script_checks": true
    }' > /consul/config/web.json

consul validate /consul/config/
consul reload
##consul services register -name=web
```

### Тестируем отказоустойчивость

```bash
sudo docker exec consul-client consul members
sudo docker exec consul-client consul operator raft list-peers
sudo docker stop consul-server3
sudo docker stop consul-server1

```

```bash
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.4:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.2:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Node            ID                                    Address          State     Voter  RaftProtocol
consul-server3  691589e9-212b-790e-2539-eca14d55bcf9  172.18.0.2:8300  follower  true   3
consul-server1  b0e3d1ad-c9b5-ec0c-c15e-9de351a101bd  172.18.0.4:8300  follower  true   3
consul-server2  1aece920-2abe-bf97-afb9-93106baf619b  172.18.0.5:8300  leader    true   3
vagrant@ubuntu2204:~/consul$ sudo docker stop consul-server3
consul-server3
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Node            ID                                    Address          State     Voter  RaftProtocol
consul-server1  b0e3d1ad-c9b5-ec0c-c15e-9de351a101bd  172.18.0.4:8300  follower  true   3
consul-server2  1aece920-2abe-bf97-afb9-93106baf619b  172.18.0.5:8300  leader    true   3
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.4:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.2:8301  left    server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul$ sudo docker stop consul-server1
consul-server1
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.4:8301  failed  server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.2:8301  left    server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Error getting peers: Failed to retrieve raft configuration: Unexpected response code: 500 (rpc error making call: No cluster leader)
vagrant@ubuntu2204:~/consul$ sudo docker start consul-server1
consul-server1
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Node            ID                                    Address          State     Voter  RaftProtocol
(unknown)       b0e3d1ad-c9b5-ec0c-c15e-9de351a101bd  172.18.0.4:8300  follower  true   unknown
consul-server2  1aece920-2abe-bf97-afb9-93106baf619b  172.18.0.5:8300  leader    true   3
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Node            ID                                    Address          State     Voter  RaftProtocol
consul-server1  b0e3d1ad-c9b5-ec0c-c15e-9de351a101bd  172.18.0.2:8300  follower  true   3
consul-server2  1aece920-2abe-bf97-afb9-93106baf619b  172.18.0.5:8300  leader    true   3
vagrant@ubuntu2204:~/consul$ sudo docker start consul-server3
consul-server3
vagrant@ubuntu2204:~/consul$ sudo docker stop consul-server2
consul-server2
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Error getting peers: Failed to retrieve raft configuration: Unexpected response code: 500 (rpc error making call: Raft leader not found in server lookup mapping)
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Error getting peers: Failed to retrieve raft configuration: Unexpected response code: 500 (rpc error making call: No cluster leader)
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Error getting peers: Failed to retrieve raft configuration: Unexpected response code: 500 (rpc error making call: No cluster leader)
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.2:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  failed  server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.4:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul$ sudo docker start consul-server2
consul-server2
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul members
Node            Address          Status  Type    Build   Protocol  DC   Partition  Segment
consul-server1  172.18.0.2:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server2  172.18.0.5:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-server3  172.18.0.4:8301  alive   server  1.11.2  2         dc1  default    <all>
consul-client   172.18.0.3:8301  alive   client  1.11.2  2         dc1  default    <default>
vagrant@ubuntu2204:~/consul$ sudo docker exec consul-client consul operator raft list-peers
Node            ID                                    Address          State     Voter  RaftProtocol
consul-server1  b0e3d1ad-c9b5-ec0c-c15e-9de351a101bd  172.18.0.2:8300  leader    true   3
consul-server2  1aece920-2abe-bf97-afb9-93106baf619b  172.18.0.5:8300  follower  true   3
consul-server3  691589e9-212b-790e-2539-eca14d55bcf9  172.18.0.4:8300  follower  true   3
vagrant@ubuntu2204:~/consul$
```

### Вывод

Может чего и не донастроил, однако при остановке хоста, который был лидером, новый не выбирается пока хост не поднимется.

### Остановка всех контейнеров
```bash
vagrant@ubuntu2204:~/consul/learn-consul-docker/datacenter-deploy-secure$ sudo docker-compose stop
[+] Stopping 4/4
 ✔ Container consul-server1  Stopped                                                                                                             0.6s
 ✔ Container consul-server2  Stopped                                                                                                             0.4s
 ✔ Container consul-server3  Stopped                                                                                                             0.5s
 ✔ Container consul-client   Stopped
```
