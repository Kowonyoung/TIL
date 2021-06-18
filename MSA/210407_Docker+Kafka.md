# Docker로 배포

1. order_ms.py
2. delivery_ms.py
3. kafka_consumer.py
4. (option) DB (Local DB or AWS RDS)
5. Zookeeper + Kafka 

<br>

![image](https://user-images.githubusercontent.com/77096463/113813271-8a922480-97aa-11eb-875c-4f58a2d79dbb.png)

<br>

### 0. my-coffee-network 생성

네트워크 생성

```
(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker network create --gateway 172.19.0.1 --subnet 172.19.0.0/24 my-coffee-network
704cce63302734e986f596141e03ed94e9a5589c54279ca2d9fc2d1bcc73390b
```

<br>

### 1. DB

mysql DB 띄우기

```
(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker run -d -p 13306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --network my-coffee-network --name mydb mysql:5.7

(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                                                NAMES
bfebf697b1ad   mysql:5.7                        "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    33060/tcp, 0.0.0.0:13306->3306/tcp                   mydb
```

```
(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker inspect mydb
```

"IPAddress": "172.19.0.3" 임을 확인

<br>

HeidiSQL에 localhost/13306으로 접속한 뒤 mydb 데이터베이스 생성 > delivery_status, orders table 생성

```sql
CREATE DATABASE mydb;
USE mydb;

CREATE TABLE delivery_status(
  `id` int NOT NULL AUTO_INCREMENT,
  `delivery_id` varchar(50) DEFAULT NULL,
  `order_json` text,
  `status` varchar(50) DEFAULT NULL,
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ;

CREATE TABLE orders(
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` varchar(100) NOT NULL,
  `order_id` varchar(100) NOT NULL,
  `coffee_name` varchar(100) NOT NULL,
  `coffee_price` int NOT NULL,
  `coffee_qty` int DEFAULT '1',
  `ordered_at` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ;
```

<br>

### 2. delivery_ms.py

(1) DB Config의 host 변경 (172.19.0.3)<br>
(2) mariadb > pymysql로 변경<br>
(3) localhost > kafka_server_ip로 변경 (172.19.0.101)<br>
(4) mysql로 변경했기 때문에 sql 쿼리문의 ?를 %s 로 변경 

```python
...

import flask_restful
#import mariadb
import pymysql
import json
import uuid

...

config = {
    'host': '172.19.0.3',
    'port': 3306,
    'user': 'root',
    'password': '',
    'database': 'mydb'
}

...
class Delivery(flask_restful.Resource):
    def __init__(self):
        self.conn = pymysql.connect(**config)
        self.cursor = self.conn.cursor()

 		...

# {"status" : "COMPLETED"}
class DeliveryStatus(flask_restful.Resource):
    def __init__(self):
        self.conn = pymysql.connect(**config)
        self.cursor = self.conn.cursor()

    def put(self,delivery_id):
        json_data = request.get_json()
        status = json_data['status']

        sql = "UPDATE delivery_status SET status = %s WHERE delivery_id=%s"

        ...
```

<br>

Dockerfile 생성 (Dockerfile_delivery)

```dockerfile
FROM python:3.7.9-stretch

WORKDIR /myflask

RUN pip install flask
RUN pip install flask_restful
RUN pip install pymysql

COPY ./delivery_ms.py /myflask/app.py

CMD ["flask", "run", "--host", "0.0.0.0", "--port", "6000"]
```

<br>

Dockerfile 빌드 및 실행

```
(msa) C:\cloud\FLASK_DEMO2>docker build -t mementohaeri/flask_delivery_ms -f Dockerfile_delivery .
```

```
(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker run -d -p 16000:6000 --network my-coffee-network --name delivery_ms mementohaeri/flask_delivery_ms

(msa) C:\cloud\FLASK_DEMO2>docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED             STATUS             PORTS                                                NAMES
7b39799857fb   mementohaeri/flask_delivery_ms   "flask run --host 0.…"   2 seconds ago       Up 2 seconds       0.0.0.0:16000->6000/tcp                              delivery_ms
bfebf697b1ad   mysql:5.7                        "docker-entrypoint.s…"   About an hour ago   Up About an hour   33060/tcp, 0.0.0.0:13306->3306/tcp                   mydb
```

<br>

16000번 포트로 HTTP GET 메서드 전송 시 STATUS가 200임을 확인 (아직 데이터를 입력하지 않아 빈 값을 가져온다.)

![image](https://user-images.githubusercontent.com/77096463/113803467-958f8980-9797-11eb-8933-6383c2215b6f.png)

<br>

### 3. Zookeeper + Kafka

git clone

```
(msa) C:\cloud\FLASK_DEMO2>git clone https://github.com/wurstmeister/kafka-docker.git
```

<br>


docker-compose-single-broker.yml 파일 수정 후 실행

- `depends_on` : zookeeper 먼저 기동 후 kafka 기동

```yaml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
    networks:
      my-network:
        ipv4_address: 172.19.0.100
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 172.19.0.101
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      my-network:
        ipv4_address: 172.19.0.101

networks:
  my-network:
    name: my-coffee-network
```

```
(msa) C:\cloud\FLASK_DEMO2\kafka-docker>docker-compose -f docker-compose-single-broker.yml up -d
```

```
(msa) C:\cloud\FLASK_DEMO2>docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED             STATUS             PORTS                                                NAMES
7b39799857fb   mementohaeri/flask_delivery_ms   "flask run --host 0.…"   2 seconds ago       Up 2 seconds       0.0.0.0:16000->6000/tcp                              delivery_ms
bfebf697b1ad   mysql:5.7                        "docker-entrypoint.s…"   About an hour ago   Up About an hour   33060/tcp, 0.0.0.0:13306->3306/tcp                   mydb
235ad27bc94f   wurstmeister/kafka               "start-kafka.sh"         2 hours ago         Up 2 hours         0.0.0.0:9092->9092/tcp                               kafka-docker_kafka_1
9f7b0c608044   wurstmeister/zookeeper           "/bin/sh -c '/usr/sb…"   2 hours ago         Up 2 hours         22/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp   kafka-docker_zookeeper_1
```

<br>

### 4. order_ms.py

Dockerfile_order 작성

```dockerfile
FROM python:3.7.9-stretch

WORKDIR /myflask

RUN pip install flask
RUN pip install flask_restful
RUN pip install pymysql
RUN pip install kafka-python

COPY ./order_ms.py /myflask/app.py

CMD ["flask", "run", "--host", "0.0.0.0", "--port", "5000"]
```

<br>

(1) DB Config의 host 변경 (172.19.0.3)<br>
(2) mariadb > pymysql로 변경<br>
(3) localhost > kafka_server_ip로 변경 (172.19.0.101)<br>
(4) mysql로 변경했기 때문에 sql 쿼리문의 ?를 %s 로 변경 

```python
...

import flask_restful
#import mariadb
import pymysql
import json
import uuid

...

config = {
    'host': '172.19.0.3',
    'port': 3306,
    'user': 'root',
    'password': '',
    'database': 'mydb'
}

...

class Order(flask_restful.Resource):
    def __init__(self):
        self.conn = pymysql.connect(**config)
        self.cursor = self.conn.cursor()
        # 1. KafkaProducer() -> 생성자에 추가
        self.producer = KafkaProducer(bootstrap_servers=['172.19.0.101:9092'])

    def get(self, user_id):
        sql = "select user_id, order_id, coffee_name, coffee_price, coffee_qty, ordered_at from orders where user_id=%s order by id desc"
        self.cursor.execute(sql, [user_id])
        result_set = self.cursor.fetchall()
        
		...

    def post(self, user_id):
        json_data = request.get_json()
        json_data['user_id'] = user_id
        json_data['order_id'] = str(uuid.uuid4()) 
        json_data['ordered_at'] = str(datetime.today())

        # DB insert
        sql = "INSERT INTO orders(user_id, order_id, coffee_name, coffee_price, coffee_qty, ordered_at) VALUES (%s,%s,%s,%s,%s,%s)"

		...
```

<br>

Dockerfile 빌드 및 실행

```
(msa) C:\cloud\FLASK_DEMO2>docker build -t mementohaeri/flask_order_ms -f Dockerfile_order .

(msa) C:\cloud\FLASK_DEMO2>docker run -d -p 15000:5000 --network my-coffee-network --name order_ms mementohaeri/flask_order_ms
```

<br>

현재 실행중인 컨테이너 확인

```
(msa) C:\cloud\FLASK_DEMO2>docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED             STATUS             PORTS                                                NAMES
7b39799857fb   mementohaeri/flask_delivery_ms   "flask run --host 0.…"   2 seconds ago       Up 2 seconds       0.0.0.0:16000->6000/tcp                              delivery_ms
705edeebf4e4   mementohaeri/flask_order_ms      "flask run --host 0.…"   4 minutes ago       Up 4 minutes       0.0.0.0:15000->5000/tcp                              order_ms
bfebf697b1ad   mysql:5.7                        "docker-entrypoint.s…"   About an hour ago   Up About an hour   33060/tcp, 0.0.0.0:13306->3306/tcp                   mydb
235ad27bc94f   wurstmeister/kafka               "start-kafka.sh"         2 hours ago         Up 2 hours         0.0.0.0:9092->9092/tcp                               kafka-docker_kafka_1
9f7b0c608044   wurstmeister/zookeeper           "/bin/sh -c '/usr/sb…"   2 hours ago         Up 2 hours         22/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp   kafka-docker_zookeeper_1
```

<br>

### 5. kafka_consumer.py

Dockerfile_consumer.py 작성

```dockerfile
FROM python:3.7.9-stretch

WORKDIR /mykafka

RUN pip install pymysql
RUN pip install kafka-python

COPY ./kafka_consumer.py /mykafka/app.py

CMD ["python", "/mykafka/app.py"]
```

<br>

(1) DB Config의 host 변경 (172.19.0.3)<br>
(2) mariadb > pymysql로 변경<br>
(3) localhost > kafka_server_ip로 변경 (172.19.0.101)

```python
...

#import mariadb
import pymysql
import uuid

config = {
    'host': '172.19.0.3',
    'port': 3306,
    'user': 'root',
    'password': '',
    'database': 'mydb'
}

consumer = KafkaConsumer('new_orders',
                        bootstrap_servers=['172.19.0.101:9092'],
                        auto_offset_reset='earliest',
                        enable_auto_commit=True,
                        auto_commit_interval_ms=1000,
                        consumer_timeout_ms=1000
                        )

conn = pymysql.connect(**config)
cursor= conn.cursor()
sql = "INSERT INTO delivery_status(delivery_id, order_json, status) VALUES (%s,%s,%s)"

...
```

<br>

Dockerfile 빌드 및 실행

```
(msa) C:\cloud\FLASK_DEMO2>docker build -t mementohaeri/kafka_consumer -f Dockerfile_consumer .

(msa) C:\cloud\FLASK_DEMO2>docker run -d -p 17000:7000 --network my-coffee-network --name kafka_consumer mementohaeri/kafka_consumer
bc9799040f13f10a595397974562fae113de87e0343f75b4d53b8e7a88c8e336
```

<br>

현재 실행중인 컨테이너 확인 (총 6개)

```
(msa) C:\cloud\FLASK_DEMO2>docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED             STATUS             PORTS                                                NAMES
7b39799857fb   mementohaeri/flask_delivery_ms   "flask run --host 0.…"   2 seconds ago       Up 2 seconds       0.0.0.0:16000->6000/tcp                              delivery_ms
705edeebf4e4   mementohaeri/flask_order_ms      "flask run --host 0.…"   4 minutes ago       Up 4 minutes       0.0.0.0:15000->5000/tcp                              order_ms
bc9799040f13   mementohaeri/kafka_consumer      "python /mykafka/app…"   13 minutes ago      Up 13 minutes      0.0.0.0:17000->7000/tcp                              kafka_consumer
bfebf697b1ad   mysql:5.7                        "docker-entrypoint.s…"   About an hour ago   Up About an hour   33060/tcp, 0.0.0.0:13306->3306/tcp                   mydb
235ad27bc94f   wurstmeister/kafka               "start-kafka.sh"         2 hours ago         Up 2 hours         0.0.0.0:9092->9092/tcp                               kafka-docker_kafka_1
9f7b0c608044   wurstmeister/zookeeper           "/bin/sh -c '/usr/sb…"   2 hours ago         Up 2 hours         22/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp   kafka-docker_zookeeper_1
```

<br>

### 6. 마지막 점검 및 HTTP 메서드 요청

my-coffee-network에 총 6개의 컨테이너가 할당 되어 있는 것과 6개의 컨테이너가 실행 중임을 확인

```
        "Containers": {
            "235ad27bc94f2959f13c4854b80addbf3b1fae6db751812d8a6771acb6e3d3f0": {
                "Name": "kafka-docker_kafka_1",
                "EndpointID": "d56beeb4b76ba91c3cbbccbaa50cd43b3c1c4609d376231cc4189a363cae6704",
                "MacAddress": "02:42:ac:13:00:65",
                "IPv4Address": "172.19.0.101/24",
                "IPv6Address": ""
            },
            "705edeebf4e4bdc08a705a42afdb221c19b5e384d330b90d3ebba3b4e271f069": {
                "Name": "order_ms",
                "EndpointID": "a31197787865b8939d94a2815bf2ac6a43cadbff9e0a5a96756d3cea43dc8841",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/24",
                "IPv6Address": ""
            },
            "7b39799857fbe6897143a12d5b28480fb56cd78e99f79c888eed97fa68bbc919": {
                "Name": "delivery_ms",
                "EndpointID": "bcc3b96de9e6481f625fc37a76b69a2135eb83fd726a1501007fdb0345d96a4b",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/24",
                "IPv6Address": ""
            },
            "9f7b0c608044c32349c0c9afb69515b5cc5bf0249c0d506252dab8c4389b9c62": {
                "Name": "kafka-docker_zookeeper_1",
                "EndpointID": "8e1cbe6bab91a40850941379fe3fced8a51fb3dec5694271e71e2482ed299d57",
                "MacAddress": "02:42:ac:13:00:64",
                "IPv4Address": "172.19.0.100/24",
                "IPv6Address": ""
            },
            "bc9799040f13f10a595397974562fae113de87e0343f75b4d53b8e7a88c8e336": {
                "Name": "kafka_consumer",
                "EndpointID": "14564a01c624acb4a269fb88416beb3b6c0a66b6058d0a0ef644c4f0ec9d5a19",
                "MacAddress": "02:42:ac:13:00:05",
                "IPv4Address": "172.19.0.5/24",
                "IPv6Address": ""
            },
            "bfebf697b1ad389ee719af6f40b6271f039a9cf6765c156264766abbbd5beef7": {
                "Name": "mydb",
                "EndpointID": "129e6556e58c152dd090217f627bf2f95c800f9922914fca441f7ad44afd43a3",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/24",
                "IPv6Address": ""
            }
        },
```

<br>

이후 아래의 다이어그램처럼 도커로 컨테이너를 배포하여 flask, kafka를 이용한다.

![image](https://user-images.githubusercontent.com/77096463/113813271-8a922480-97aa-11eb-875c-4f58a2d79dbb.png)

<br>

**order_ms.py** > HTTP GET 메서드를 통해 orders 테이블의 데이터를 가져온다. (처음에는 아무 데이터도 없으므로 빈 값 가져온다.)

![image](https://user-images.githubusercontent.com/77096463/113820574-d8f8f080-97b5-11eb-879b-e2f0037f75aa.png)

<br>

**order_ms.py** > HTTP POST 메서드로 orders 테이블에 데이터를 추가한다. 

![image](https://user-images.githubusercontent.com/77096463/113820601-e01ffe80-97b5-11eb-8663-d76d0bfb258d.png)

<br>

orders 테이블에 데이터가 입력된 것을 확인

![image](https://user-images.githubusercontent.com/77096463/113820668-f928af80-97b5-11eb-9663-094acdaba689.png)

<br>

**delivery_ms.py** > HTTP GET 메서드 통해 delivery_status 테이블의 데이터를 가져온다. (kafka_consumer.py에 의해 orders 테이블의 데이터가 delivery_status 테이블에 저장됨)

![image](https://user-images.githubusercontent.com/77096463/113820633-e9a96680-97b5-11eb-8e06-a3f878ba2c9e.png)

<br>

**delivery_ms.py** > HTTP PUT 메서드를 통해 주문의 상태(Status)를 변경

![image](https://user-images.githubusercontent.com/77096463/113820652-f201a180-97b5-11eb-88ea-4c27f64d3e3a.png)

<br>

주문의 상태 변경 후 delivery_status 테이블에서 데이터의 상태가 변경되었는지 확인

![image](https://user-images.githubusercontent.com/77096463/113820685-00e85400-97b6-11eb-965f-ab5eea9d141c.png)

<br>

