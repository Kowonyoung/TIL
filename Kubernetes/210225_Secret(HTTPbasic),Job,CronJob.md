# Secret - nginx 기본인증정보가 담긴 파일을 시크릿으로 관리 (p.254)

### 접근통제 3단계

- 식별 (identification)
- 인증 (authentication)
  - TYPE 1: 알고 있는 정보 (지식기반) -패스워드
  - TYPE 2: 가지고 있는 정보 (소유기반) - 주민등록증, OTP, 인증서, 스마트폰 등
  - TYPE 3: 특징 - 홍채, 지문, 성문, 정맥 (바이오 인증), 필기체 서명 (사인) 등
  - 2가지 이상을 혼용: 2-factor 인증, multi-factor 인증(다중인증방식)
  - 멀티 디바이스 인증 (멀티 채널 인증)
- 인가 (authorization) 

참고 : https://myanjini.tistory.com/51

<br>

### HTTP 기본인증

> 1. 웹 애플리케이션이 HTTP 요청을 받는다.
> 2. 서버는 사용자가 누구인지 식별하기 위해 개인정보를 요구하는 인증 요구로 응답한다.
> 3. 인증 요구를 받은 클라이언트는 사용자에게 인증 정보를 물어본다. 사용자는 인증 정보를 추가해 다시 요청을 보낸다.
> 4. 서버는 인증 정보를 확인하여 OK를 하거나, 인증 정보가 맞지 않다면 에러를 반환한다.
>
> **BASIC 인증** : 클라이언트에 대한 사용자 이름과 암호를 BASE64로 인코딩된 문자열 보낸다.

![HTTPAuth](https://user-images.githubusercontent.com/77096463/109166340-f3f04200-77bf-11eb-9164-7160636d9561.png)

1. openssl 모듈을 이용해서 id, pwd를 암호화한 후 BASE64로 인코딩

- **$(openssl passwd -quiet -crypt your_password)** : openssl을 이용하여 your_password 암호화
- "your_name:(암호화된 패스워드)" 문자열 생성
- 생성된 문자열을 base64로 인코딩

```
[vagrant@master ~]$ echo "your_name:$(openssl passwd -quiet -crypt your_password)" | base64
eW91cl9uYW1lOjNCaTNmUmZNaE5DM2cK
```

2. 시크릿을 생성하는 yaml 파일 작성

```
[vagrant@master ~]$ vi nginx-secret.yaml
```

```yaml
# nginx-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
data:
 .htpasswd: eW91cl9uYW1lOjNCaTNmUmZNaE5DM2cK
```

--dry-run -o yaml 옵션으로 시크릿을 생성하지 않고 yaml 파일 생성

```
[vagrant@master ~]$ kubectl create secret generic nginx-secret --from-literal .htpasswd=eW91cl9uYW1lOjNCaTNmUmZNaE5DM2cK --dry-run -o yaml > test.yaml

[vagrant@master ~]$ cat test.yaml
apiVersion: v1
data:
  .htpasswd: ZVc5MWNsOXVZVzFsT2pOQ2FUTm1VbVpOYUU1RE0yY0s=
kind: Secret
metadata:
  creationTimestamp: null
  name: nginx-secret
```


3. 시크릿 생성

```
[vagrant@master ~]$ kubectl apply -f nginx-secret.yaml
secret/nginx-secret created
```

```
[vagrant@master ~]$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-wqzv4   kubernetes.io/service-account-token   3      5d23h
my-password           Opaque                                1      17h
nginx-secret          Opaque                                1      26s
our-password          Opaque                                2      17h
```

4. 시크릿을 활용해 기본인증을 적용하는 nginx 파드를 구성

```
[vagrant@master ~]$ vi basic-auth.yaml
```

```yaml
# basic-auth.yaml

apiVersion: v1
kind: Service
metadata:
  name: basic-auth
spec:
  type: NodePort
  selector:
    app: basic-auth
  ports:
    - protocol : TCP
      port: 80
      targetPort: http
      nodePort: 30060
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: basic-auth
  labels: 
    app: basic-auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: basic-auth
  template:
    metadata:
      labels:
        app: basic-auth
    spec:
      containers:
        - name: nginx
          image: "gihyodocker/nginx:latest"
          imagePullPolicy: Always
          ports:
          - name: http
            containerPort: 80
          env:
          - name: BACKEND_HOST
            value: "localhost:8080"
          - name: BASIC_AUTH_FILE
            value: "/etc/nginx/secret/.htpasswd"
          volumeMounts:
            - mountPath: etc/nginx/secret
              name: nginx-secret
              readOnly: true
        - name: echo
          image: "gihyodocker/echo:latest"
          imagePullPolicy: Always
          ports: 
            - containerPort: 8080
          env:
            - name: HTTP_PORT
              value: "8080"
      volumes:
      - name: nginx-secret
        secret:
          secretName: nginx-secret
```

5. 생성 및 확인

```
[vagrant@master ~]$ kubectl apply -f basic-auth.yaml
service/basic-auth unchanged
deployment.apps/basic-auth created
```

```
[vagrant@master ~]$ kubectl get po,deployments,svc
NAME                                       READY   STATUS    RESTARTS   AGE
pod/basic-auth-6fdd6978b8-xj899            2/2     Running   0          17m
pod/configmap-volume-pod                   1/1     Running   1          19h
pod/container-env-example                  1/1     Running   1          20h
pod/hostname-deployment-6cd58767b4-2mtk8   1/1     Running   0          87m
pod/hostname-deployment-6cd58767b4-kgcbd   1/1     Running   0          87m
pod/hostname-deployment-6cd58767b4-njzsl   1/1     Running   0          87m
pod/secret-volume-example                  1/1     Running   1          17h
pod/selective-secret-volume-example        1/1     Running   1          17h

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/basic-auth            1/1     1            1           17m
deployment.extensions/hostname-deployment   3/3     3            3           24h

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/basic-auth              NodePort    10.105.56.146   <none>        80:30060/TCP     18m
service/hello-service           NodePort    10.100.84.147   <none>        8080:30221/TCP   5d16h
service/hostname-svc-nodeport   NodePort    10.99.101.47    <none>        8080:32516/TCP   24h
service/kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP          6d
service/nginx-test              NodePort    10.104.64.74    <none>        80:30403/TCP     5d21h
```

6. 인증 정보 없이 호출 시 401 오류 반환
```
[vagrant@master ~]$ curl -i http://127.0.0.1:30060
HTTP/1.1 401 Unauthorized					# 응답 시작 -> 인증 정보가 없는 경우 401 오류 반환
Server: nginx/1.13.12						# 응답 헤더 시작
Date: Thu, 25 Feb 2021 01:31:52 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 196
Connection: keep-alive
WWW-Authenticate: Basic realm="Restricted"	# 기본인증 방식으로 인증정보 요청
											# 응답 헤더 끝
<html>										# 응답 본문 (사용자 브라우저에 출력되는 내용)
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.13.12</center>
</body>
</html>
```
7. 인증정보와 함께 호출 시 정상 처리
- **--user your_name:your_password** : 사용자 인증 정보 전달

```
[vagrant@master ~]$ curl -i --user your_name:your_password http://127.0.0.1:30060
HTTP/1.1 200 OK
Server: nginx/1.13.12
Date: Thu, 25 Feb 2021 01:37:31 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 14
Connection: keep-alive
```

<br><br>

# Job (p.250)

하나 이상의 파드를 생성해 지정된 수의 파드가 정상 종료될 때까지 이를 관리하는 리소스

Job이 생성한 파드는 정상 종료 후에도 삭제되지 않고 남아 있어 로그나 실행 결과를 분석할 수 있다.

배치 작업 (일정한 형태의 작업이 반복) 형태에 적합

1. 매니페스트 파일 작성

```
[vagrant@master ~]$ vi simple-job.yaml
```

```yaml
# simple-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: pingpong
  labels:
    app: pingpong
spec:
  parallelism: 3		# 동시에 실행하는 파드의 수 (병렬)
  template:				# 파드 정의
    metadata:
      labels:
        app: pingpong
    spec:
      containers:
        - name: pingpong
          image: gihyodocker/alpine:bash
          command: ["/bin/sh"]
          args:
          - "c"
          - |
            echo [`date`] ping!
            sleep 10
            echo [`date`] pong!
      restartPolicy: Never			# 파드 종료 후 재실행 여부 설정 (Alaways, Never, OnFailure)
```

2. 생성 및 확인

```
[vagrant@master ~]$ kubectl apply -f simple-job.yaml
job.batch/pingpong created

[vagrant@master ~]$ kubectl logs -l app=pingpong
[Thu Feb 25 14:46:38 UTC 2021] ping!	# 38초
[Thu Feb 25 14:46:48 UTC 2021] pong!
[Thu Feb 25 14:46:37 UTC 2021] ping!	# 37초
[Thu Feb 25 14:46:48 UTC 2021] pong!
[Thu Feb 25 14:46:27 UTC 2021] ping!	# 27초
[Thu Feb 25 14:46:37 UTC 2021] pong!
```

STATUS가 completed라는 것은 작업이 모두 종료된 상태
```
[vagrant@master ~]$ kubectl get pods -l app=pingpong -o wide
NAME                        READY   STATUS      RESTARTS   AGE     IP                NODE    NOMINATED NODE   READINESS GATES
pingpong-5v8mh              0/1     Completed   0          3m23s   192.168.166.180   node1   <none>           <none>
pingpong-kl954              0/1     Completed   0          3m23s   192.168.166.181   node1   <none>           <none>
pingpong-pcfvx              0/1     Completed   0          3m23s   192.168.104.41    node2   <none>           <none>
```



<br>

<br>

# CronJob (p.252)

Job은 한 번만 실행되는 반면, CronJob은 스케줄을 지정해 정기적으로 파드 실행

cron 등을 사용해 정기적으로 실행하는 작업에 적합

<br>



![image](https://user-images.githubusercontent.com/77096463/109166868-81339680-77c0-11eb-9633-3daca0893bda.png)

<br>

1. 매니페스트 파일 작성

```
[vagrant@master ~]$ vi simple-cronjob.yaml
```

```yaml
# simple-cronjob.yaml

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: pingpong
spec:
  schedule: "*/1 * * * *"			# 파드를 실행할 스케줄 정의
  jobTemplate:						# 파드 정의
    spec:
      template:
        metadata:
          labels:
            app: pingpong
        spec: 
          containers:
            - name: pingpong
              image: gihyodocker/alpine:bash
              command: ["/bin/bash"]
              args: 
              - "-c"
              - | 
                echo [`date`] ping!
                sleep 10
                echo [`date`] pong!
          restartPolicy: OnFailure
```

2. 생성 및 적용
```
[vagrant@master ~]$ kubectl apply -f simple-cronjob.yaml
cronjob.batch/pingpong created
```

약 1분 단위로 작업이 수행됨을 확인 가능

```
[vagrant@master ~]$ kubectl get job -l app=pingpong
NAME                  COMPLETIONS   DURATION   AGE
pingpong-1614219180   1/1           12s        2m14s			# 134 초 전
pingpong-1614219240   1/1           12s        74s				# 74초 전
pingpong-1614219300   1/1           13s        14s				# 14초 전
```

3. 로그 확인 -> 여기서도 60초 간격임을 확인

```
[vagrant@master ~]$ kubectl logs -l app=pingpong
[Thu Feb 25 02:15:03 UTC 2021] ping!
[Thu Feb 25 02:15:13 UTC 2021] pong!
[Thu Feb 25 02:16:03 UTC 2021] ping!
[Thu Feb 25 02:16:13 UTC 2021] pong!
[Thu Feb 25 02:17:03 UTC 2021] ping!
[Thu Feb 25 02:17:13 UTC 2021] pong!
[Thu Feb 25 02:18:03 UTC 2021] ping!
[Thu Feb 25 02:18:13 UTC 2021] pong!
```

