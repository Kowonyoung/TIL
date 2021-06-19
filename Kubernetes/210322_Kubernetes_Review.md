2024~2025년 기점으로 클라우드를 전면적으로 사용할 예정<br>
-> 로컬에 사용하고 있던 미들웨어나 운영체제 등을 클라우드로 이전할 계획<br>
-> 외산 브랜드보다는 국산 브랜드 위주<br>
-> 이유 : 비용, 속도 측면에서 유리 / 수작업의 한계 / 시스템 재사용 가능
<br>
운영체제, 데이터베이스, 미들웨어 등의 리소스를 클라우드를 이용해 재분할<br>
네이버, LG 등이 클라우드 컴퓨팅이 가능하게끔 컴퓨팅 파워를 할당시켜줌<br>

도커, K8S -> 가상화 솔루션으로 미들웨어 성격을 가짐<br>
클라우드 서비스라기 보다는 클라우드에서 운영할 수 있는 가상화 기술을 관리하는 역할<br>
도커 : 컨테이너 가상화의 한 종류 -> 유료화될 가능성<br>
- crio-o<br>
- containerd<br>
- AWS, Naver, LG, KT(public cloud) 에서도 도커 서비스 제공 가능<br>
	- Database<br>
	- IaaS : 가상의 HW 구축<br>
	- PaaS : 플랫폼<br>
	- SaaS : SW (Gmail, ERP, Office365, Adobe Photoshop..)<br>
	- PreaaS ...<br>

쿠버네티스 : 도커, crio-o, containerd를 관리 (컨테이너 가상화를 관리)<br>
- pod : 컨테이너가 포함되어 있는데, 이는 도커, crio-o, containerd로 만들 수 있음<br>
- 즉, 쿠버네티스가 없어도 도커를 사용할 수 있음<br>
- 쿠버네티스는 도커, crio-o, containerd 중 하나가 있어야 사용 가능<br>

오픈스택 : 하이브리드 클라우드, 프라이빗 클라우드 구축 시 사용될 수 있는 솔루션<br>

VPC : Virtual Private Cloud(Network) > aws에서 구축하는 네트워크 <br>
- 서브넷 : 같은 네트워크에 묶여져 있음 > 통신할 때 제약이 없음<br>
- IPv4<br>
    - 0~255.0~255.0~255.0~255<br>
    - 256*256*256*256<br>
    - 가상 네트워크 private network<br>
    - 내부적으로 통신이 가능<br>
    - 외부와 통신하기 위해서는 Gateway 필요 <br>

-------------

<br>

# k8s 실습 (pod, svc)

pod 생성 
```
[root@master vagrant]# vi my_hello_pod.yml 

[root@master vagrant]# kubectl apply -f my_hello_pod.yml 
pod/hello-pod created
```
```yaml
[root@master vagrant]# cat my_hello_pod.yml
apiVersion: v1
kind: Pod
metadata:
 name: hello-pod
 labels:
  app: hello
spec:
 containers:
 - name: hello-container
   image: edowon0623/hello:2.0 
   ports:
   - containerPort: 8000
```

pod 상세 정보 확인

```
[root@master vagrant]# kubectl get pods -o wide
NAME                                   READY   STATUS      RESTARTS   AGE    IP                NODE    NOMINATED NODE   READINESS GATES
hello-pod                              1/1     Running     0          2m8s   192.168.104.47    node2   <none>           <none>
```

hello-pod가 실행 중인 node2로 가서 정보 확인

```
[root@node2 vagrant]# docker ps | grep hello-pod
60a5e71d5c9d   edowon0623/hello       "docker-entrypoint.s…"   About a minute ago   Up About a minute             k8s_hello-container_hello-pod_default_6bd3f9cf-8645-4bab-b8b2-8cc1a3c086f0_0
5f771298912f   k8s.gcr.io/pause:3.1   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_hello-pod_default_6bd3f9cf-8645-4bab-b8b2-8cc1a3c086f0_0
```

hello-pod가 잘 실행 중인지 확인 (내부 클러스터)

```
[root@master vagrant]# curl 192.168.104.47:8000
Hello, World (on K8S)!
```

<br>

service yaml 파일 생성 (외부에서도 접근하고 사용할 수 있게끔)

```yaml
[root@master vagrant]# cat my_hello_svc.yml 
apiVersion: v1
kind: Service
metadata:
 name: hello-service
spec:
 selector:
  app: hello
 ports:
  - port: 8001
    targetPort: 8000
 type: NodePort
```

service 생성 및 확인

- 외부에서 사용할 때는 31827번으로 접근

```
[root@master vagrant]# kubectl apply -f my_hello_svc.yml
service/hello-service configured

[root@master vagrant]# kubectl get svc
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-service           NodePort    10.100.84.147   <none>        8001:31827/TCP   30d
```

service 테스트 

```
//서비스 접속
[root@master vagrant]# curl 127.0.0.1:31827
Hello, World (on K8S)!
```

service 상세 정보 확인

```
[root@master vagrant]# kubectl describe svc hello-service
Name:                     hello-service
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"hello-service","namespace":"default"},"spec":{"ports":[{"port":80...
Selector:                 app=hello
Type:                     NodePort
IP:                       10.100.84.147
Port:                     <unset>  8001/TCP
TargetPort:               8000/TCP
NodePort:                 <unset>  31827/TCP
Endpoints:                192.168.104.47:8000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

service 내용 변경

```
[root@master vagrant]# kubectl edit svc hello-service
```

윈도우에서 127.0.0.1:31827를 통해 접속하고 싶을 때 vagrant에 포트 포워딩 필요

```
C:\Users\Lenovo>cd C:\cloud\vagrant
C:\cloud\vagrant>code Vagrantfile
```

vagrantFile을 보면 30403포트가 열려있기 때문에 svc의 포트를 변경할 예정

- hello-pod가 실행 중인 노드를 확인 후 해당 노드의 포트포워딩 확인 및 설정

```
   # Node2
  config.vm.define:"node-2" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="node-2"
        vb.customize ["modifyvm", :id, "--cpus", 1]
        vb.customize ["modifyvm", :id, "--memory", 1024]
    end
    cfg.vm.host_name="node2"
    # cfg.vm.synced_folder ".", "/vagrant", type: "nfs"
    cfg.vm.network "private_network", ip: "192.168.56.12"
    cfg.vm.network "forwarded_port", guest: 22, host: 19212, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 28080
    cfg.vm.network "forwarded_port", guest: 30403, host: 30403
    cfg.vm.network "forwarded_port", guest: 30404, host: 30404
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOS.sh"
  end
```
```
[root@master vagrant]# kubectl edit svc hello-service
```

![image](https://user-images.githubusercontent.com/77096463/111943247-67177a80-8b18-11eb-8346-6a886fd8aa80.png)

<br>

이후 127.0.0.1:30403 접속 시 윈도우에서 접속 가능

![image](https://user-images.githubusercontent.com/77096463/111943203-4c450600-8b18-11eb-8dae-d0648e0e6380.png)

<br>

# nginx-proxy, echo 애플리케이션을 pod로 배포

이미지 가져오기

```
[root@master vagrant]# docker pull gihyodocker/nginx:latest
[root@master vagrant]# docker pull gihyodocker/echo:latest

[root@master vagrant]# docker images | grep gihyodocker
gihyodocker/nginx                    latest     88f12faf6a8a   2 years ago     155MB
gihyodocker/echo                     latest     3dbbae6eb30d   3 years ago     733MB
```

yaml 파일 작성

```yaml
[root@master vagrant]# vi simple-pod.yaml
[root@master vagrant]# cat simple-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
 name: simple-echo
spec:
 containers:
 - name: nginx
   image: gihyodocker/nginx:latest
   env: 
   - name: BACKEND_HOST
     value: localhost:8080
   ports:
   - containerPort: 80
 - name: echo
   image: gihyodocker/echo:latest
   ports:
   - containerPort: 8080
```

pod 생성

```
[root@master vagrant]# kubectl apply -f simple-pod.yaml
pod/simple-echo created

[root@master vagrant]# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
basic-auth-6fdd6978b8-mwk99   2/2     Running   0          5h
hello-pod                     1/1     Running   0          22m
simple-echo                   2/2     Running   0          18s
```

ip주소 호출

```
[root@master vagrant]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE    IP                NODE    NOMINATED NODE   READINESS GATES
simple-echo                   2/2     Running   0          45s    192.168.104.50    node2   <none>           <none>

[root@master vagrant]# curl 192.168.104.50:8080
Hello Docker!!
[root@master vagrant]# curl 192.168.104.50:80
Hello Docker!! 
```

simple-echo 파드로 접속
```
[root@master vagrant]# kubectl exec -it simple-echo sh -c nginx 
# hostname
simple-echo
# exit
```
로그 확인
```
[root@master vagrant]# kubectl logs simple-echo -c nginx
[root@master vagrant]# kubectl logs simple-echo -c echo
2021/03/22 06:47:23 start server
2021/03/22 06:55:49 received request
2021/03/22 06:55:55 received request
```



<br>

# ReplicaSet

yaml 파일 작성 -> 1개의 pod 당 2개의 컨테이너 생성 -> 총 6개의 컨테이너 생성됨

```yaml
[root@master vagrant]# vi simple-replicas.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: echo
 labels:
  app: echo	//replica sets의 자체 레이블
spec:
 replicas: 3
 selector:
  matchLabels:
   app: simple-echo
 template:
  metadata:
   labels:
    app: simple-echo
  spec:
   containers:
   - name: nginx
     image: gihyodocker/nginx:latest
     env: 
     - name: BACKEND_HOST
       value: localhost:8080
     ports:
     - containerPort: 80
   - name: echo
     image: gihyodocker/echo:latest
     ports:
     - containerPort: 8080
```

replica set 생성 및 확인
```
[root@master vagrant]# kubectl apply -f simple-replicas.yaml
replicaset.apps/echo created

[root@master vagrant]# kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
basic-auth-6fdd6978b8   1         1         1       25d
echo                    3         3         0       7s
```

replica set ip 주소 확인 및 접속

```
[root@master vagrant]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP                NODE    NOMINATED NODE   READINESS GATES
echo-b22m6                    2/2     Running   0          43s     192.168.104.54    node2   <none>           <none>
echo-cnfnr                    2/2     Running   0          43s     192.168.104.52    node2   <none>           <none>
echo-slr46                    2/2     Running   0          43s     192.168.166.190   node1   <none>           <none>

[root@master vagrant]# curl 192.168.104.54:8080
Hello Docker!!
[root@master vagrant]# curl 192.168.104.52:8080
Hello Docker!!
[root@master vagrant]# curl 192.168.166.190:8080
Hello Docker!!
```

데이터를 필터링하여 검색 -> app이 simple-echo인 파드 검색

```
[root@master vagrant]# kubectl get pods --selector app=simple-echo
NAME         READY   STATUS    RESTARTS   AGE
echo-b22m6   2/2     Running   0          10m
echo-cnfnr   2/2     Running   0          10m
echo-slr46   2/2     Running   0          10m
```

<br>

# Deployment

Deployment를 위한 yaml 파일 작성

```yaml
[root@master vagrant]# vi simple-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: echo
 labels:
  app: echo
spec:
 replicas: 4
 selector:
  matchLabels:
   app: my-echo
 template:
  metadata:
   lables:
    app: my-echo
  spec:
   containers:
   - name: nginx
     image: gihyodocker/nginx:latest
     env: 
     - name: BACKEND_HOST
       value: localhost:8080
     ports:
     - containerPort: 80
   - name: echo
     image: gihyodocker/echo:latest
     env:
     - name: HOGE
       value: fuga
     ports:
     - containerPort: 8080
```

Deployment 생성 및 확인

```
[root@master vagrant]# kubectl apply -f simple-deployment.yaml 
deployment.apps/echo created

[root@master vagrant]# kubectl get pods
NAME                          READY   STATUS        RESTARTS   AGE
echo-786c577d9-fvgw8          2/2     Running       0          34s
echo-786c577d9-rppfc          2/2     Running       0          51s
echo-786c577d9-v8d8f          2/2     Running       0          51s
echo-786c577d9-vs5ln          2/2     Running       0          35s
```

라벨이 app=echo인 파드, 레플리카셋, 배포 목록 확인 -> 데이터 필터링

```
[root@master vagrant]# kubectl get pod,rs,deployment --selector app=echo
NAME                       READY   STATUS    RESTARTS   AGE
pod/echo-786c577d9-fvgw8   2/2     Running   0          93s
pod/echo-786c577d9-rppfc   2/2     Running   0          110s
pod/echo-786c577d9-v8d8f   2/2     Running   0          110s
pod/echo-786c577d9-vs5ln   2/2     Running   0          94s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-786c577d9   4         4         4       111s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   4/4     4            4           112s
```
deployment echo의 롤아웃 기록 확인

- 현재 추가로 업데이트 하지 않았기 때문에 REVISION 1과 함께 아무 내용 출력하지 않음

```
[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         <none>
```

변경 작업을 수행한 후 롤아웃 기록 확인

```
[root@master vagrant]# kubectl apply -f simple-deployment.yaml --record
deployment.apps/echo configured

[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=simple-deployment.yaml --record=true
```

<br>

<br>

현재까지의 상황

```
[root@master vagrant]# kubectl get pod,rs,deployment
NAME                        READY   STATUS    RESTARTS   AGE
pod/echo-568f446568-fmpff   2/2     Running   0          3m6s
pod/echo-568f446568-g8wsz   2/2     Running   0          3m6s
pod/echo-568f446568-hftcl   2/2     Running   0          3m6s
pod/echo-568f446568-k222p   2/2     Running   0          3m6s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   4         4         4       3m6s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   4/4     4            4           3m6s
```

<br>

replicaset개수 3개로 바꾸고 적용한 후 롤아웃 확인

```
[root@master vagrant]# vi simple-deployment.yaml 

[root@master vagrant]# kubectl apply -f simple-deployment.yaml --record
deployment.apps/echo configured

[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=simple-deployment.yaml --record=true
```

```
[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=simple-deployment.yaml --record=true

[root@master vagrant]# kubectl get pod,rs,deployment
NAME                        READY   STATUS        RESTARTS   AGE
pod/echo-568f446568-fmpff   2/2     Running       0          5m47s
pod/echo-568f446568-g8wsz   2/2     Running       0          5m47s
pod/echo-568f446568-hftcl   0/2     Terminating   0          5m47s
pod/echo-568f446568-k222p   2/2     Running       0          5m47s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   3         3         3       5m47s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           5m47s
```

<br>

echo 컨테이너의 <u>환경 변수 삭제 후</u> 적용 롤아웃

```
[root@master vagrant]# vi simple-deployment.yaml 
[root@master vagrant]# kubectl apply -f simple-deployment.yaml --record
deployment.apps/echo configured

[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=simple-deployment.yaml --record=true
2         kubectl apply --filename=simple-deployment.yaml --record=true
```

<br>

revision=1과 revision=2의 내용 각각 확인

```
[root@master vagrant]# kubectl rollout history deployment echo --revision=1
deployment.extensions/echo with revision #1
Pod Template:
  Labels:	app=my-echo
	pod-template-hash=568f446568
  Annotations:	kubernetes.io/change-cause: kubectl apply --filename=simple-deployment.yaml --record=true
  Containers:
   nginx:
    Image:	gihyodocker/nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:
      BACKEND_HOST:	localhost:8080
    Mounts:	<none>
   echo:
    Image:	gihyodocker/echo:latest
    Port:	8080/TCP
    Host Port:	0/TCP
    Environment:
      HOGE:	fuga
    Mounts:	<none>
  Volumes:	<none>
```

```
[root@master vagrant]# kubectl rollout history deployment echo --revision=2
deployment.extensions/echo with revision #2
Pod Template:
  Labels:	app=my-echo
	pod-template-hash=6b4dbbb464
  Annotations:	kubernetes.io/change-cause: kubectl apply --filename=simple-deployment.yaml --record=true
  Containers:
   nginx:
    Image:	gihyodocker/nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:
      BACKEND_HOST:	localhost:8080
    Mounts:	<none>
   echo:
    Image:	gihyodocker/echo:latest
    Port:	8080/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

revision=1로 롤백

- `kubectl rollout history deployment echo` : REVISION 2,3으로 변경됨
- `kubectl rollout history deployment echo --revision=3` : REVISION 3 내용을 보면 환경변수 내역이 다시 돌아왔음을 확인

```
[root@master vagrant]# kubectl rollout undo deployment echo --to-revision=1
deployment.extensions/echo rolled back

[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
2         kubectl apply --filename=simple-deployment.yaml --record=true
3         kubectl apply --filename=simple-deployment.yaml --record=true

[root@master vagrant]# kubectl rollout history deployment echo --revision=3
deployment.extensions/echo with revision #3
Pod Template:
  Labels:	app=my-echo
	pod-template-hash=568f446568
  Annotations:	kubernetes.io/change-cause: kubectl apply --filename=simple-deployment.yaml --record=true
  Containers:
   nginx:
    Image:	gihyodocker/nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:
      BACKEND_HOST:	localhost:8080
    Mounts:	<none>
   echo:
    Image:	gihyodocker/echo:latest
    Port:	8080/TCP
    Host Port:	0/TCP
    Environment:
      HOGE:	fuga
    Mounts:	<none>
  Volumes:	<none>
```

<br>

이후 같은 작업 반복 후 파드의 변화 내역을 보면 파드 이름이 568f446568(환경변수 o) 혹은 6b4dbbb464(환경변수 x) 로 계속 변화함을 확인

```
#revision1 (echo 컨테이너 환경변수 존재)
NAME                        READY   STATUS    RESTARTS   AGE
pod/echo-568f446568-fmpff   2/2     Running   0          6m47s
pod/echo-568f446568-g8wsz   2/2     Running   0          6m47s
pod/echo-568f446568-k222p   2/2     Running   0          6m47s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   3         3         3       6m47s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           6m47s




#revision2 (echo 컨테이너 환경변수 삭제)
NAME                        READY   STATUS    RESTARTS   AGE
pod/echo-6b4dbbb464-7qljp   2/2     Running   0          2m19s
pod/echo-6b4dbbb464-b6cgc   2/2     Running   0          2m2s
pod/echo-6b4dbbb464-msr5s   2/2     Running   0          2m11s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   0         0         0       11m
replicaset.extensions/echo-6b4dbbb464   3         3         3       2m19s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           11m



#revision3 (환경 변수 있음)
[root@master vagrant]# kubectl get pod,rs,deployment
NAME                        READY   STATUS    RESTARTS   AGE
pod/echo-568f446568-97x6h   2/2     Running   0          91s
pod/echo-568f446568-b72jz   2/2     Running   0          81s
pod/echo-568f446568-xdvpn   2/2     Running   0          73s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   3         3         3       19m
replicaset.extensions/echo-6b4dbbb464   0         0         0       10m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           19m



#revision4 ((환경 변수 없음))
[root@master vagrant]# kubectl get pod,rs,deployment
NAME                        READY   STATUS        RESTARTS   AGE
pod/echo-6b4dbbb464-ffd4j   2/2     Running       0          31s
pod/echo-6b4dbbb464-m9qs5   2/2     Running       0          23s
pod/echo-6b4dbbb464-vxvzj   2/2     Running       0          15s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-568f446568   0         0         0       22m
replicaset.extensions/echo-6b4dbbb464   3         3         3       13m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           22m

```

<br>

rollout history 내역에 코멘트를 추가하고 싶을 때

```
[root@master vagrant]# kubectl annotate deployment/echo kubernetes.io/change-cause="not exists environment variables"
deployment.extensions/echo annotated

[root@master vagrant]# kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
3         kubectl apply --filename=simple-deployment.yaml --record=true
4         not exists environment variables
```

