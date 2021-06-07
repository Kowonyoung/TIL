1. Mysql 컨테이너 추가

```
$ docker run -d -p 23306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true -v C:\cloud\finance_project\db_mount:/var/lib/mysql --network my-network --name mysql_server2 mysql:5.7
```
<br/>
2. mydb 데이터베이스 생성

```
$ docker exec -it mysql_server2 /bin/bash
# mysql -h127.0.0.1 -uroot
> create database mydb;
> show databases;
```

<br/>

3. django project의 database 변경

```python
#settings.py
DATABASES = {
    'default': {
        # 'ENGINE': 'django.db.backends.sqlite3',
        # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        'ENGINE':'django.db.backends.mysql',
        'NAME':'mydb',
        'USER':'root',
        'PASSWORD':'',
        'HOST':'127.0.0.1',
        'PORT':'23306'
    }
}
```

```
$ python .\manage.py makemigrations
$ python .\manage.py migrate
$ python .\manage.py createsuperuser
$ python .\manage.py runserver	//서버 구동 확인
```

도커파일을 수정한 후 새로운 이미지 빌드

```
$ docker build -t mydjango:mysql . 
$ docker run -d -it -p 8000:8000 --network my-network --name mydjango2 mydjango:mysql 
```

명령어 수행 후 http:127.0.0.1/8000 서버 접속하기 

![image-20210214003147644](https://user-images.githubusercontent.com/77096463/108019030-a9065a00-705c-11eb-8c43-8282396cef70.png)

4. database를 환경변수로 설정하여 컨테이너 생성 및 실행

```
$ docker run -d -it -p 8000:8000 -e MYSQL_ALLOW_EMPTY_PASSWORD=true -e MYSQL_DATABASE=mydb --network my-network --name mydjango2 mydjango:mysql
```

5. mydjango 이미지 tag 변경

```
$ docker tag mydjango:mysql mementohaeri/mydjango:1.0
```

hub에 tag 변경한 이미지 업로드

```
$ docker push mementohaeri/mydjango:1.0
```





6. MYSQL 용 도커파일 생성

도커파일 수정

```dockerfile
# Dockerfile
FROM mysql:5.7

ENV MYSQL_ALLOW_EMPTY_PASSWORD true
ENV MYSQL_DATABASE mydb
EXPOSE 3306

COPY db_mount /var/lib/mysql 

CMD ["mysqld"]
```

이미지 빌드

```
$ docker build --no-cache=true -t mymysql . 
```

컨테이너 생성 및 실행

```
$ docker run -d -p 23306:3306 --network my-network --name mysql_server2 mymysql:latest
$ docker exec -it mysql_server2 /bin/bash
# mysql -h127.0.0.1 -uroot
> show databases;
> use mydb;
> show tables;
```

위 명령어를 통해 기존 장고 프로젝트에서 생성된 테이블을 확인할 수 있다.

![image-20210214004402994](https://user-images.githubusercontent.com/77096463/108019031-aa378700-705c-11eb-9dc6-7fb333dee14b.png)



