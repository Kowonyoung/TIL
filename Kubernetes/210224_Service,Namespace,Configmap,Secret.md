# Service
### NodePort타입 서비스  - 서비스를 이용해 pod를 외부에 노출시키기

1. yaml 파일 생성

```
[vagrant@master ~]$ vi hostname-svc-nodeport.yaml
```

```yaml
# hostname-svc-nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-nodeport
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: NodePort
```



2. 서비스 생성 및 확인

```
[vagrant@master ~]$ kubectl apply -f hostname-svc-nodeport.yaml
[vagrant@master ~]$ kubectl get svc
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-service           NodePort    10.100.84.147   <none>        8080:30221/TCP   4d16h
hostname-svc-nodeport   NodePort    10.99.101.47    <none>        8080:32516/TCP   17m
kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP          5d
nginx-test              NodePort    10.104.64.74    <none>        80:30403/TCP     4d21h
```

```
[vagrant@master ~]$ kubectl get nodes -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master   Ready    master   5d      v1.15.5   192.168.56.10   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.3
node1    Ready    <none>   4d23h   v1.15.5   192.168.56.11   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.3
node2    Ready    <none>   4d23h   v1.15.5   192.168.56.12   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.3
```



3. 외부 ip를 통해 32516포트로 접속 -> 3개의 pod로 접속한 내용을 확인 가능

```
[vagrant@master ~]$ curl 127.0.0.1:32516
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-6cd58767b4-4jkdj</p>     </blockquote>
</div>

[vagrant@master ~]$ curl 127.0.0.1:32516
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-6cd58767b4-5dxxc</p>     </blockquote>
</div>

[vagrant@master ~]$ curl 127.0.0.1:32516
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-6cd58767b4-k5dg2</p>     </blockquote>
</div>
```



4. 내부 ip와 내부 dns이름을 통해 접근 (클러스터 내부로 접근하여)

```
[vagrant@master ~]$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash
If you don't see a command prompt, try pressing enter.
root@debug:/# curl 10.99.101.47:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-6cd58767b4-5dxxc</p>     </blockquote>
</div>

root@debug:/# curl hostname-svc-nodeport:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-6cd58767b4-5dxxc</p>     </blockquote>
</div>
```



<br>

<br>

# Namespace

쿠버네티스 클러스터 안의 가상 클러스터
파드, 레플리카셋, 디플로이먼트, 서비스 등과 같은 쿠버네티스 리소스들이 묶여 있는 하나의 가상 공간 또는 그룹
리소스를 **논리적으로 구분**하기 위한 오브젝트

<br>

### 네임스페이스 조회

1. 네임스페이스 목록 조회

```
[vagrant@master ~]$ kubectl get ns
NAME              STATUS   AGE
default           Active   5d1h			//자동 제공
kube-node-lease   Active   5d1h
kube-public       Active   5d1h			//자동 제공
kube-system       Active   5d1h			//자동 제공
```

2. default 네임스페이스에 생성된 파드 확인

```
[vagrant@master ~]$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
hostname-deployment-6cd58767b4-4jkdj   1/1     Running   0          38m
hostname-deployment-6cd58767b4-5dxxc   1/1     Running   0          38m
hostname-deployment-6cd58767b4-k5dg2   1/1     Running   0          38m

[vagrant@master ~]$ kubectl get pods --namespace default
NAME                                   READY   STATUS    RESTARTS   AGE
hostname-deployment-6cd58767b4-4jkdj   1/1     Running   0          38m
hostname-deployment-6cd58767b4-5dxxc   1/1     Running   0          38m
hostname-deployment-6cd58767b4-k5dg2   1/1     Running   0          38m
```

3. kube-system 네임스페이스에 생성된 파드 확인
- kube-system 네임스페이스 : 쿠버네티스 클러스터 구성에 필수적인 컴포넌트와 설정값 포함

```
[vagrant@master ~]$ kubectl get pods --namespace kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7fc57b95d4-2bgdq   1/1     Running   10         5d1h
calico-node-5sw8d                          1/1     Running   9          4d22h
calico-node-zlm89                          1/1     Running   5          4d23h
calico-node-zr496                          1/1     Running   4          4d21h
coredns-5c98db65d4-7r54w                   1/1     Running   10         5d1h
coredns-5c98db65d4-v26lm                   1/1     Running   10         5d1h
etcd-master                                1/1     Running   11         5d1h
kube-apiserver-master                      1/1     Running   11         5d1h
kube-controller-manager-master             1/1     Running   11         5d1h
kube-proxy-2jtl2                           1/1     Running   4          5d1h
kube-proxy-g485d                           1/1     Running   10         5d1h
kube-proxy-jrjj5                           1/1     Running   6          5d1h
kube-scheduler-master                      1/1     Running   11         5d1h
kubernetes-dashboard-6b8c96cf8c-ck2l7      1/1     Running   4          4d23h

[vagrant@master ~]$ kubectl get pods -n kube-system
```

<br>

### 네임스페이스 생성 및 확인

1. yaml 파일 작성

```yaml
# production-namespace.yml

apiVersion: v1
kind: Namespace
metadata:
  name: production
```

2. 네임스페이스를 생성하고 확인한다.

```
[vagrant@master ~]$ kubectl apply -f production-namespace.yaml
namespace/production created

[vagrant@master ~]$ kubectl create namespace mynamespace
namespace/mynamespace created
```

```
[vagrant@master ~]$ kubectl get ns
NAME              STATUS   AGE
default           Active   5d1h
kube-node-lease   Active   5d1h
kube-public       Active   5d1h
kube-system       Active   5d1h
mynamespace       Active   5s
production        Active   96s
```

3. 특정 네임스페이스에 리소스를 생성한 후, 배포한다.

```yaml
# hostname-deploy-svc-ns.yaml

apiVersion: apps/v1
kind: Deployment
metadata:					# 리소스의 부가 정보 (이름,네임스페이스,라벨,주석)
  name: hostname-deployment-ns
  namespace: production		# production 네임스페이스 지정
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      name: my-webserver
      labels:
        app: webserver
    spec:
      containers:
      - name: my-webserver
        image: alicek106/rr-test:echo-hostname
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-clusterip-ns
  namespace: production		# production 네임스페이스 지정
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: ClusterIP
```

```
[vagrant@master ~]$ kubectl apply -f hostname-deploy-svc-ns.yaml
deployment.apps/hostname-deployment-ns created
service/hostname-svc-clusterip-ns created
```

<br>

생성된 리소스를 확인한다.

```
[vagrant@master ~]$ kubectl get po,svc --namespace production
NAME                                          READY   STATUS    RESTARTS   AGE
pod/hostname-deployment-ns-6cd58767b4-jmwxn   1/1     Running   0          40s
pod/hostname-deployment-ns-6cd58767b4-pc55w   1/1     Running   0          40s
pod/hostname-deployment-ns-6cd58767b4-t4fdp   1/1     Running   0          40s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/hostname-svc-clusterip-ns   ClusterIP   10.97.180.223   <none>        8080/TCP   39s
```

4. 모든 네임스페이스의 리소스를 확인하고 싶은 경우

```
[vagrant@master ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       hostname-deployment-6cd58767b4-4jkdj       1/1     Running   0          69m
default       hostname-deployment-6cd58767b4-5dxxc       1/1     Running   0          69m
default       hostname-deployment-6cd58767b4-k5dg2       1/1     Running   0          69m
kube-system   calico-kube-controllers-7fc57b95d4-2bgdq   1/1     Running   11         5d1h
kube-system   calico-node-5sw8d                          1/1     Running   10         4d22h
kube-system   calico-node-zlm89                          1/1     Running   5          4d23h
kube-system   calico-node-zr496                          1/1     Running   4          4d22h
kube-system   coredns-5c98db65d4-7r54w                   1/1     Running   11         5d1h
kube-system   coredns-5c98db65d4-v26lm                   1/1     Running   11         5d1h
kube-system   etcd-master                                1/1     Running   12         5d1h
kube-system   kube-apiserver-master                      1/1     Running   12         5d1h
kube-system   kube-controller-manager-master             1/1     Running   12         5d1h
kube-system   kube-proxy-2jtl2                           1/1     Running   4          5d1h
kube-system   kube-proxy-g485d                           1/1     Running   11         5d1h
kube-system   kube-proxy-jrjj5                           1/1     Running   6          5d1h
kube-system   kube-scheduler-master                      1/1     Running   12         5d1h
kube-system   kubernetes-dashboard-6b8c96cf8c-ck2l7      1/1     Running   4          4d23h
production    hostname-deployment-ns-6cd58767b4-jmwxn    1/1     Running   0          107s
production    hostname-deployment-ns-6cd58767b4-pc55w    1/1     Running   0          107s
production    hostname-deployment-ns-6cd58767b4-t4fdp    1/1     Running   0          107s
```

5. 동일 네임스페이스 내의 서비스에 접근

임시로 production 네임스페이스에 파드를 생성한 후, production 네임스페이스 ip로 접근한다.

```
[vagrant@master ~]$ kubectl get svc -n production
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hostname-svc-clusterip-ns   ClusterIP   10.97.180.223   <none>        8080/TCP   3m51s
```

```
[vagrant@master ~]$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never --namespace=production -- bash
If you don't see a command prompt, try pressing enter.

root@debug:/# curl 10.97.180.223:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-ns-6cd58767b4-jmwxn</p>  </blockquote>
</div>

root@debug:/# curl hostname-svc-clusterip-ns:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-ns-6cd58767b4-jmwxn</p>  </blockquote>
</div>
```

6. 다른 네임스페이스 내의 서비스에 접근

임시로 default네임스페이스에 파드를 생성한 후, 접근 시도

```
[vagrant@master ~]$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash
If you don't see a command prompt, try pressing enter.

root@debug:/# curl 10.97.180.223:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-ns-6cd58767b4-pc55w</p>  </blockquote>
</div>

root@debug:/# curl hostname-svc-clusterip-ns:8080
curl: (6) Could not resolve host: hostname-svc-clusterip-ns
```

다른 네임스페이스에 존재하는 서비스로 접근하기 위해서는 **[서비스이름].[네임스페이스이름].svc** 형태로 사용! 
서비스 이름으로 접속하는 기존 방식은 접속 불가하지만, [서비스이름].[네임스페이스이름].svc 형태로 접속하면 접속 가능하다.


```
root@debug:/# curl hostname-svc-clusterip-ns.production.svc:8080
<!DOCTYPE html>
<meta charset="utf-8" />
<link rel="stylesheet" type="text/css" href="./css/layout.css" />
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">

<div class="form-layout">
        <blockquote>
        <p>Hello,  hostname-deployment-ns-6cd58767b4-pc55w</p>  </blockquote>
</div>
```

<br>

### 네임스페이스 삭제

```
[vagrant@master ~]$ kubectl delete namespace mynamespace
namespace "mynamespace" deleted
```

```
[vagrant@master ~]$ kubectl delete -f production-namespace.yaml
namespace "production" deleted
```

<br>

### 네임스페이스에 속하는 오브젝트 종류

```
[vagrant@master ~]$ kubectl api-resources --namespaced=true
NAME                        SHORTNAMES   APIGROUP                    NAMESPACED   KIND
bindings                                                             true         Binding
configmaps                  cm                                       true         ConfigMap
endpoints                   ep                                       true         Endpoints
events                      ev                                       true         Event
limitranges                 limits                                   true         LimitRange
persistentvolumeclaims      pvc                                      true         PersistentVolumeClaim
pods                        po                                       true         Pod
podtemplates                                                         true         PodTemplate
...
```

<br>

### 네임스페이스에 속하지 않는 오브젝트 종류

```
[vagrant@master ~]$ kubectl api-resources --namespaced=false
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
componentstatuses                 cs                                          false        ComponentStatus
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumes                 pv                                          false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
tokenreviews                                   authentication.k8s.io          false        TokenReview
...
```

<br>

# Configmap, Secret - 파드에 설정값 전달

configmap은 설정 데이터를, secret은 노출되면 안되는 기밀 데이터를 저장

<br>

### 컨피그맵 생성

yaml 파일로 생성 혹은 `kubectl create configmap` 명령어로 생성
- **--from-literal KEY=VALUE**에서 key,value 형식의 정보를 저장 (여러 개 지정 가능)


```
[vagrant@master ~]$ kubectl create configmap log-level-configmap --from-literal LOG_LEVEL=DEBUG
configmap/log-level-configmap created

[vagrant@master ~]$ kubectl create configmap start-k8s --from-literal k8s=kubernetes --from-literal container=docker
```



### 컨피그맵 확인

```
[vagrant@master ~]$ kubectl get configmap
NAME                  DATA   AGE
log-level-configmap   1      48s
start-k8s             2      40s
```
```
[vagrant@master ~]$ kubectl describe configmap log-level-configmap
Name:         log-level-configmap
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
LOG_LEVEL:
----
DEBUG
Events:  <none>
```

```
[vagrant@master ~]$ kubectl get configmap log-level-configmap -o yaml
apiVersion: v1
data:
  LOG_LEVEL: DEBUG
kind: ConfigMap
metadata:
  creationTimestamp: "2021-02-24T04:34:06Z"
  name: log-level-configmap
  namespace: default
  resourceVersion: "157980"
  selfLink: /api/v1/namespaces/default/configmaps/log-level-configmap
  uid: e8b1a721-bece-4aba-b626-6438a7226388
```

<br>

### 파드에서 컨피그맵 사용하는 방법1 - 컨피그맵의 값을 컨테이너 환경변수로 사용

1. yaml 파일을 생성한다.
- **envFrom** : 컨피그맵에 정의되어 있는 **모든** key=value 쌍을 가져와서 환경변수로 설정

```
[vagrant@master ~]$ vi all-env-from-configmap.yaml
```

```
# all-env-from-configmap.yaml

apiVersion: v1
kind: Pod
metadata:
  name: container-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail','-f','/dev/null']
      envFrom:			# configmap에 지정된 key=value를 환경변수로 설정
      - configMapRef:
          name: log-level-configmap
      - configMapRef:
          name: start-k8s
```

2. 파드 생성

```
[vagrant@master ~]$ kubectl apply -f all-env-from-configmap.yaml
pod/container-env-example created
```

3. 컨테이너 환경 변수 확인

```
[vagrant@master ~]$ kubectl exec container-env-example -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=container-env-example
LOG_LEVEL=DEBUG
container=docker
k8s=kubernetes
KUBERNETES_SERVICE_HOST=10.96.0.1
HOSTNAME_SVC_NODEPORT_SERVICE_PORT_WEB_PORT=8080
..
```

4. yaml 파일을 생성한다.
- **valueFrom,** **configMapKeyRef**: 컨피그맵에 존재하는 key=value 중에서 **원하는 데이터 선택**하여 환경변수로 지정

```
[vagrant@master ~]$ vi selective-env-from-configmap.yaml
```

```yaml
# selective-env-from-configmap.yaml

apiVersion: v1
kind: Pod
metadata:
  name: container-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ['tail','-f','/dev/null']
      env:
      - name: ENV_KEYNAME_1				#새롭게 정의될 환경변수 이름
        valueFrom:
          configMapKeyRef:
            key: LOG_LEVEL				#해당 컨피그맵에 정의된 변수의 키
            name: log-level-configmap	#참조할 컨피그맵 이름
      - name: ENV_KEYNAME_2
        valueFrom:
          configMapKeyRef:
            key: start-k8s
            name: k8s
```

5. 파드 생성 및 환경변수 확인

```
[vagrant@master ~]$ kubectl delete -f selective-env-from-configmap.yaml
pod "container-env-example" deleted

[vagrant@master ~]$ kubectl apply -f selective-env-from-configmap.yaml
pod/container-env-example created

[vagrant@master ~]$ kubectl exec container-env-example -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=container-env-example
ENV_KEYNAME_1=DEBUG			# 차이점 주목
ENV_KEYNAME_2=kubernetes	# 차이점 주목
HELLO_SERVICE_SERVICE_PORT=8080
HOSTNAME_SVC_NODEPORT_PORT=tcp://10.99.101.47:8080
HOSTNAME_SVC_NODEPORT_PORT_8080_TCP_ADDR=10.99.101.47
KUBERNETES_PORT=tcp://10.96.0.1:443
HELLO_SERVICE_PORT_8080_TCP_PORT=8080
```

<br>

### 파드에서 컨피그맵 사용하는 방법2 - 컨피그맵의 값을 파드 내부의 파일로 마운트해 사용

1. yaml 파일을 생성한다.
- **volumeMounts**: 모든 key-value 데이터를 파드에 마운트
- configmap-volume이름으로 만들어진 볼륨마운트를 /etc/config로 매핑

```
[vagrant@master ~]$ vi volume-mount-configmap.yaml
```

```
# volume-mount-configmap.yaml

apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - name: configmap-volume
          mountPath: /etc/config
  volumes:
    - name: configmap-volume
      configMap:
        name: start-k8s
```

2. 파드 생성

```
[vagrant@master ~]$ kubectl apply -f volume-mount-configmap.yaml
pod/configmap-volume-pod created
```

3. 파드 내부의 /etc/config 디렉터리 조회

```
[vagrant@master ~]$ kubectl exec configmap-volume-pod -- ls -al /etc/config
total 0
drwxrwxrwx    3 root     root            87 Feb 24 05:46 .
drwxr-xr-x    1 root     root            20 Feb 24 05:47 ..
drwxr-xr-x    2 root     root            34 Feb 24 05:46 ..2021_02_24_05_46_58.625256452
lrwxrwxrwx    1 root     root            31 Feb 24 05:46 ..data -> ..2021_02_24_05_46_58.625256452
lrwxrwxrwx    1 root     root            16 Feb 24 05:46 container -> ..data/container
lrwxrwxrwx    1 root     root            10 Feb 24 05:46 k8s -> ..data/k8s
```

아래 명령어를 수행하면 파일 내용은 컨피그맵의 값과 동일하다.

```
[vagrant@master ~]$ kubectl exec configmap-volume-pod -- cat /etc/config/k8s
kubernetes

[vagrant@master ~]$ kubectl exec configmap-volume-pod -- cat /etc/config/container
docker
```

<br>

### 원하는 key-value 데이터만 선택해서 파드에 마운트

1. yaml 파일 생성한다.

```yaml
# selective-volume-configmap.yaml

apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - name: configmap-volume
          mountPath: /etc/config
  volumes:
    - name: configmap-volume
      configMap:
        name: start-k8s
        items:
        - key: k8s                      
          path: k8s_fullname 
```

2. 파드 생성 후 마운트 디렉터리 및 파일 확인

```
[vagrant@master ~]$ kubectl apply -f selective-volume-configmap.yaml
pod/configmap-volume-pod created

[vagrant@master ~]$ kubectl exec configmap-volume-pod -- ls -al /etc/config
total 0
drwxrwxrwx    3 root     root            79 Feb 24 05:59 .
drwxr-xr-x    1 root     root            20 Feb 24 06:00 ..
drwxr-xr-x    2 root     root            26 Feb 24 05:59 ..2021_02_24_05_59_54.777798358
lrwxrwxrwx    1 root     root            31 Feb 24 05:59 ..data -> ..2021_02_24_05_59_54.777798358
lrwxrwxrwx    1 root     root            19 Feb 24 05:59 k8s_fullname -> ..data/k8s_fullname
```

```
[vagrant@master ~]$ kubectl exec configmap-volume-pod -- cat /etc/config/k8s_fullname
kubernetes
```



### 파일로부터 컨피그맵 생성

nginx.conf (nginx 서버의 설정 파일) 혹은 mysql.conf (MYSQL 서버의 설정 파일)의 내용 전체를 컨피그맵에 저장할 때 사용

1. 설정 파일 생성 

```
[vagrant@master ~]$ echo Hello World >> index.html
```

2. index.html 파일로 컨피그맵 생성

```
[vagrant@master ~]$ kubectl create configmap index-file --from-file index.html
```

3. 컨피그맵 내용 확인
- 파일 명 : 컨피그맵의 key로 사용
- 파일 내용 : 컨피그맵의 value로 사용

```
[vagrant@master ~]$ kubectl describe configmap index-file
Name:         index-file
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
index.html:			# key (파일 명)
----
Hello World			# value (파일 내용)

Events:  <none>
```

4. 키 이름을 직접 지정해서 컨피그맵 생성

```
[vagrant@master ~]$ kubectl create configmap index-file-customkey --from-file myindex=index.html
```

```
[vagrant@master ~]$ kubectl describe configmap index-file-customkey
Name:         index-file-customkey
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
myindex:
----
Hello World

Events:  <none>
```

<br>

### 여러 개의 key-value 형식의 내용으로 구성된 설정 파일을 한꺼번에 컨피그맵으로 가져오기

### --dry-run -o yaml 옵션으로 컨피그맵 생성하지 않고 yaml 파일로 생성



# Secret

SSH 키, 비밀번호 등과 같이 민감한 정보를 저장하는 용도

**타입**

- opaque 타입
	- 기본 타입
	- 사용자가 정의하는 데이터를 정의하는 일반적인 목적의 시크릿
- 비공개 레지스트리 (private registry)에 접근할 때 사용하는 인증설정 시크릿

<br>

###  시크릿 생성 

1. --from-literal 이용

- generic : opaque 타입의 시크릿 생성

```
[vagrant@master ~]$ kubectl create secret generic my-password --from-literal password=p@ssw0rd
secret/my-password created
```

```
[vagrant@master ~]$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-wqzv4   kubernetes.io/service-account-token   3      5d6h
my-password           Opaque                                1      9s
```

2. --from-file 이용

pw1, pw2 파일 생성

```
[vagrant@master ~]$ echo mypassword > pw1 && echo yourpassword > pw2
[vagrant@master ~]$ ls pw*
pw1  pw2
```

secret 생성 (our-password)
```
[vagrant@master ~]$ kubectl create secret generic our-password --from-file pw1 --from-file pw2
secret/our-password created
[vagrant@master ~]$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-wqzv4   kubernetes.io/service-account-token   3      5d6h
my-password           Opaque                                1      9m51s
our-password          Opaque                                2      4s
```



### 시크릿 내용 확인

```
[vagrant@master ~]$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-wqzv4   kubernetes.io/service-account-token   3      5d6h
my-password           Opaque                                1      9m51s
our-password          Opaque                                2      4s
[vagrant@master ~]$ kubectl describe secret my-password
Name:         my-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
```

- cEBzc3cwcmQ= : BASE64+padding 
  - BASE64: ASCII 문자 중에서 가시 영역의 문자로 데이터를 표현하는 방식 -> 주로 첨부파일, 암호화된 데이터 저장에 사용
  - padding: 글자 수를 맞춰주기 위함

```
[vagrant@master ~]$ kubectl get secrets my-password -o yaml
apiVersion: v1
data:
  password: cEBzc3cwcmQ=	# BASE64로 인코딩됨
kind: Secret
metadata:
  creationTimestamp: "2021-02-24T07:05:50Z"
  name: my-password
  namespace: default
  resourceVersion: "171208"
  selfLink: /api/v1/namespaces/default/secrets/my-password
  uid: 9148cc5c-3d18-4dc9-91aa-702c7260176a
type: Opaque
```

인코딩과 디코딩

```
[vagrant@master ~]$ echo p@ssw0rd | base64
cEBzc3cwcmQK

[vagrant@master ~]$ echo cEBzc3cwcmQK | base64 -d
p@ssw0rd
```

<br>

### 시크릿에 저장된 값을 파드의 환경변수로 참조

시크릿에 저장된 모든 값을 가져오기

```
[vagrant@master ~]$ vi env-from-secret.yaml
```

```yaml
# env-from-secret.yaml

apiVersion: v1
kind: Pod
metadata:
  name: secret-env-example
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail","-f","/dev/null"]
      envFrom:
        - secretRef:	#my-password 시크릿에 저장된 모든 key-value 환경변수로 설정
            name: my-password
```

시크릿에 저장된 일부 값을 가져오기

```
[vagrant@master ~]$ vi selective-env-from-secret.yaml
```

```yaml
# selective-env-from-secret.yaml

apiVersion: v1
kind: Pod
metadata:
  name: secret
spec:
  containers:
    - name: my-container
      image: busybox
      args: ["tail","-f","/dev/null"]
      env:
        - name: YOUR_PASSWORD
          valueFrom:
            secretKeyRef:
              name: out-password
              key: pw2
```

시크릿에 저장된 값 전체를 파드에 볼륨 마운트

```
[vagrant@master ~]$ vi volume-mount-secret.yaml
```

```yaml
# volume-mount-secret.yaml

apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-example
spec:
  container:
    - name: my-container
      image: busybox  
      args: ["tail","-f","/dev/null"]
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secret
  volumes:
    - name: secret-volume
      secret: 
        secretName: our-password
```



시크릿에 저장된 값 일부를 파드에 볼륨 마운트

```
[vagrant@master ~]$ vi selective-volume-mount-secret.yaml
```

```yaml
# selective-volume-mount-secret.yaml

apiVersion: v1
kind: Pod
metadata:
  name: selective-secret-volume-example
spec:
  container:
    - name: my-container
      image: busybox  
      args: ["tail","-f","/dev/null"]
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secret
  volumes:
    - name: secret-volume
      secret: 
        secretName: our-password
        items: 
          - key: pw1
            path: password1
```

```
[vagrant@master ~]$ kubectl apply -f selective-volume-mount-secret.yaml
pod/selective-secret-volume-example created

[vagrant@master ~]$ kubectl exec selective-secret-volume-example -- ls -al /etc/secret
total 0
drwxrwxrwt    3 root     root           100 Feb 24 08:11 .
drwxr-xr-x    1 root     root            20 Feb 24 08:11 ..
drwxr-xr-x    2 root     root            60 Feb 24 08:11 ..2021_02_24_08_11_08.654928795
lrwxrwxrwx    1 root     root            31 Feb 24 08:11 ..data -> ..2021_02_24_08_11_08.654928795
lrwxrwxrwx    1 root     root            16 Feb 24 08:11 password1 -> ..data/password1

[vagrant@master ~]$ kubectl exec selective-secret-volume-example -- cat /etc/secret/password1
mypassword
```

