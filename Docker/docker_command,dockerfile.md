# Docker 명령어

1. 컨테이너 목록 출력

```
$ docker ps [OPTIONS]
```
<br/>

2. 컨테이너 정지

```
$ docker stop [OPTIONS] CONTAINER [CONTAINER..]
ex. docker stop 68c3
```
<br/>

3. 컨테이너 삭제

```
$ docker rm [OPTIONS] CONTAINER [CONTAINER..]
ex. docker rm 68c3
```
<br/>

4. 이미지 목록 출력

```
$ docker images [OPTIONS] [REPOSITORY[:TAG]]
```
<br/>

5. 이미지 삭제

```
$ docker rmi [OPTIONS] IMAGE[IMAGE...]
```
<br/>

6. 이미지 받아오기 (이미지 데이터 저장)

```
$ docker pull [OPTIONS] NAME[:TAG|@DIGEST]
ex. docker pull ubuntu:16.04
```
<br/>

7. 컨테이너의 로그 확인 (컨테이너의 표준 출력 표시)

```
$ docker logs ${CONTAINER_ID}
```
<br/>

8. 외부에서 컨테이너 안의 명령 실행

```
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
ex. docker exec -it mynodejs /bin/bash
```
<br/>

9. 사용하지 않는 자원 (볼륨, 이미지, 컨테이너 등) 삭제

```
$ docker system prune -a
```
<br/>

10. 컨테이너와 이미지의 세부 정보를 JSON 형태로 출력

```
$ docker inspect ${CONTAINER/IMAGE_ID}
```
<br/>
<br/>


# Docker 이미지 생성

> 컨테이너의 상태를 그대로 이미지로 저장

애플리케이션을 이미지로 만들기 위해 리눅스만 설치된 컨테이너에 애플리케이션을 설치하고 그 상태를 그대로 이미지로 저장하는 단순한 방식이다.  Add files는 설치, 설정, 시작 파일 등으로 구성된다.

![image-20210208220220259](https://user-images.githubusercontent.com/77096463/107226061-f848fb00-6a5c-11eb-89a4-97f3eddb8930.png)

<br/>

**Application file + Dockerfile**
- 작업 프로젝트 데이터인 Application file과 이미지 빌드용 DSL인 Dockrfile이 더해져 Docker Image가 생성된다.
-  DSL : Domain Specific Language

![image-20210208220539349](https://user-images.githubusercontent.com/77096463/107226074-ff700900-6a5c-11eb-9a78-4357b29bda52.png)

<br/>
<br/>


# Dockerfile 작성하기

1. Dockerfile 생성

```
$ touch Dockerfile
```
<br/>

2. Dockerfile 작성

```
$ gedit Dockerfile
```
```
# Dockerfile
FROM ubuntu:latest		// 이미지 내려받기

RUN mkdir /mydata 	//디렉토리 생성
COPY test.sh /mydata/test.sh	//test.sh 파일 복사

ENTRYPOINT /mydata/test.sh	//test.sh 파일 실행
```
<br/>

3. Build

```
$ docker build -t fromtest:1.0 .
$ docker build --no-cache=true -t fromtest:1.0 .
```
<br/>

4. Run

```
$ docker container run -it fromtest:1.0
```
<br/>

5. (OPTIONAL) Copy

```
$ copy .\Dockerfile \Dockerfile_fromtest
```
<br/>







참고 : https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html