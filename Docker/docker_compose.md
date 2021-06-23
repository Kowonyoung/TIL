# Docker Compose

Docker Compose는 Docker Command 혹은 복잡한 설정을 쉽게 관리하기 위한 Tool이다.

- YAML 형식으로 Docker를 생성하며 settings 관련 작업을 작성한 script 파일
- 컨테이너별로 별도의 Docker 명령어를 실행하지 않고 **한 번에 여러 개의 컨테이너를 동시에 실행한다.**

<br/>
<br/>
[기존의 방식]

![image](https://user-images.githubusercontent.com/77096463/107904120-3a5acb00-6f8e-11eb-8af4-d7315f9191fe.png)

<br/>
<br/>
[Docker Compose 방식]

![image](https://user-images.githubusercontent.com/77096463/107903987-f071e500-6f8d-11eb-8afa-ab0af7ce325a.png)

<br/>
<br/>

[Docker Compose 형식]

- 같은 depth를 가진다면 똑같은 command line으로 인식한다.
- 아래의 경우 총 2개의 컨테이너 (servicename , servicename2 )를 실행시키고자 한다.

![image](https://user-images.githubusercontent.com/77096463/107907330-c1ac3c80-6f96-11eb-99bb-9fe888ce2e57.png)

<br/>
<br/>

### 1. Docker Compose 실행

```
$ docker-compose up
$ docker-compose up --build 	//dockerfile 다시 빌드
$ docker-compose --file docker-compose.yml up 	//docker-compose.yml 파일 시작
```
<br/>
<br/>

### 2. Docker Compose 종료

```
$ docker-compose down
```
<br/>
<br/>

### 3. Docker Compose 실습

1. docker compose 파일 작성

- my-mysql 컨테이너와 my-django 컨테이너 설정
- **depends_on** : 서비스 종속성 순서대로 서비스 시작 (my-mysql 서비스 시작 -> my-django 서비스 시작)
- **networks** : 2개의 서비스가 모두 한 네트워크에서 실행되게끔 지정

```yaml
# Docker-compose.yml
version: "3.9"
services: 
  my-mysql:
    container_name: mysql_server2
    image: mementohaeri/mymysql:3.0
    volumes:
      - ./mysql-data2:/var/lib/mysql
    ports: 
      - 23306:3306
    environment: 
      MYSQL_DATABASE: mydb
      MYSQL_ALLOW_EMPTY_PASSWORD: "true"
    networks:
      - my-network
  my-django:
    container_name: django_server
    image: mementohaeri/mydjango:3.0
    ports:
      - 8000:8000
    depends_on: 
      - my-mysql
    networks:
      - my-network
networks: 
  my-network:
    driver: bridge
```
<br/>
<br/>

2. docker-compose.yml 파일 작성 후, `$ docker-compose up` 명령어 실행하기

```
$ docker-compose up
$ docker ps
```

![image](https://user-images.githubusercontent.com/77096463/107906500-a7715f00-6f94-11eb-9878-1aa5f868a084.png)
<br/>
<br/>

3. 다음과 같은 오류가 나타날 때 (db는 존재하나 table을 받아오지 못하는 경우)

![image](https://user-images.githubusercontent.com/77096463/107906641-edc6be00-6f94-11eb-9a4c-c80ef3badc46.png)

```
$ docker exec -it django_server /bin/bash
# python manage.py migrate
```
<br/>

다시 서버에 접속하면 table 데이터를 가져왔음을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/77096463/107906790-5d3cad80-6f95-11eb-8a88-c65474109943.png)

<br/>
<br/>

4. 각 컨테이너가 제대로 작동하는지 로그 확인

```
$ docker logs django_server
$ docker logs mysql_server2
```

![image](https://user-images.githubusercontent.com/77096463/107907023-eeac1f80-6f95-11eb-9b7e-08335728fb03.png)

![image](https://user-images.githubusercontent.com/77096463/107907052-01beef80-6f96-11eb-8c84-cfb4174ab4f8.png)
<br/>
<br/>

5. mysql_server2 컨테이너와 django_server 컨테이너가 서로 통신하는지 확인

```
$ docker exec -it django_server /bin/bash
# ping mysql_server2
```

![image](https://user-images.githubusercontent.com/77096463/107907165-4480c780-6f96-11eb-8eea-3f1359dae7d1.png)