# MQTT-Protocol-Implementation-and-Security Setup Environment

This project is to reproduce this vulnerability:
[Protocol fuzzing to find security vulnerabilities of RabbitMQ](https://onlinelibrary.wiley.com/doi/abs/10.1002/cpe.6012)


1. For RabbitMQ 3.6.10
```
docker run -it --network my_network --name rabbitmq_server -p 1883:1883 -p 5672:5672 -p 15672:15672 ubuntu:18.04
```
```
apt update
apt install wget -y
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v3_6_10/rabbitmq-server_3.6.10-1_all.deb
apt-get install -y erlang-nox
apt-get install -y logrotate socat
dpkg -i rabbitmq-server_3.6.10-1_all.deb
service rabbitmq-server start

rabbitmq-plugins enable rabbitmq_management
rabbitmq-plugins enable rabbitmq_mqtt
sudo rabbitmqctl add_user admin qwer1234
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

sudo rabbitmqctl add_user test1 1234
sudo rabbitmqctl set_permissions -p / test1 ".*" ".*" ".*"

sudo rabbitmqctl add_user test2 1234
sudo rabbitmqctl set_permissions -p / test2 ".*" ".*" ".*"

sudo rabbitmqctl add_user test3 1234
sudo rabbitmqctl set_permissions -p / test3 ".*" ".*" ".*"
```

2. For Fuzzer
```
docker run -it --network my_network --name mqtt_fuzzer ubuntu:18.04
```
```
apt update
apt install git -y
git clone https://gitlab.com/akihe/radamsa.git
cd radamsa
apt install make
apt install wget
apt install curl -y
apt-get install build-essential -y
sudo make OFLAGS=-01
gzip -d < ol.c.gz > ol.c
mkdir -p bin
cc -Wall -O3 -o bin/ol ol.c
./bin/ol -O1 -o radamsa.c rad/main.scm
sudo make install
cd ../
apt install python2.7
apt install python-pip
pip install Twisted==13.2.0
git clone https://github.com/F-secure/mqtt_fuzz.git
python2.7 mqtt_fuzz.py 172.18.0.2 1883 -ratio 3 -delay 100
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

