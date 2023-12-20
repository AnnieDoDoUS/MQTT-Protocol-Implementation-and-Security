# MQTT-Protocol-Implementation-and-Security Setup Environment

This project is to reproduce this vulnerability:
[Protocol fuzzing to find security vulnerabilities of RabbitMQ](https://onlinelibrary.wiley.com/doi/abs/10.1002/cpe.6012)

1. For RabbitMQ 3.6.10


* Dockerfile:
```
FROM ubuntu:18.04

WORKDIR /app

RUN apt update
RUN apt install -y build-essential
RUN apt install wget -y
RUN wget https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v3_6_10/rabbitmq-server_3.6.10-1_all.deb
RUN apt-get install -y erlang-nox
RUN apt-get install -y logrotate socat
RUN dpkg -i rabbitmq-server_3.6.10-1_all.deb
RUN service rabbitmq-server start
```
* Build docker image:
```
docker build -t rabbitmq_server
```

* Build docker container:
```
docker run -it --network my_network --name rabbitmq_server -p 1883:1883 -p 5672:5672 -p 15672:15672 rabbitmq_server
```

* Add users
```
rabbitmq-plugins enable rabbitmq_management
rabbitmq-plugins enable rabbitmq_mqtt
rabbitmqctl add_user admin qwer1234
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

rabbitmqctl add_user test1 1234
rabbitmqctl set_permissions -p / test1 ".*" ".*" ".*"

rabbitmqctl add_user test2 1234
rabbitmqctl set_permissions -p / test2 ".*" ".*" ".*"

rabbitmqctl add_user test3 1234
rabbitmqctl set_permissions -p / test3 ".*" ".*" ".*"
```

2. For Fuzzer

* Dockerfile:
```
FROM ubuntu:18.04

WORKDIR /app

RUN apt update
RUN apt install git -y
RUN git clone https://gitlab.com/akihe/radamsa.git
RUN apt install make
RUN apt install wget
RUN apt install curl -y
RUN apt install sudo
RUN apt-get install build-essential -y
RUN (cd radamsa && make OFLAGS=-01) || true
RUN cd radamsa && gzip -d < ol.c.gz > ol.c && mkdir -p bin && ./bin/ol -O1 -o radamsa.c rad/main.scm && make install 
RUN apt install python2.7 -y
RUN apt install python-pip -y
RUN pip install Twisted==13.2.0
RUN git clone https://github.com/F-secure/mqtt_fuzz.git
```

* Build docker image:
```
docker build -t mqtt_fuzzer
```

* Build docker container:
```
docker run -it --network my_network --name mqtt_fuzzer mqtt_fuzzer
```

* Execute fuzzer:
Be sure to change broker_ip to real broker ip at first.
```
cd mqtt_fuzzer
python2.7 mqtt_fuzz.py [broker_ip] 1883 -ratio 3 -delay 100
```
3. For Publisher
```
docker run -it --network my_network --name publisher_ubuntu ubuntu:18.04
```
```
apt update
apt install mosquitto-clients -y
mosquitto_pub -h 172.18.0.2 -p 1883 -t key1 -u test1 -P 1234 -m "hello world!" -i producer
```

4. For Subscriber
```
docker run -it --network my_network --name subscriber_ubuntu ubuntu:18.04
```
```
apt update
apt install mosquitto-clients -y
mosquitto_sub -h 172.18.0.2 -p 1883 -t key1 -u test1 -P 1234 -i consumer
```

