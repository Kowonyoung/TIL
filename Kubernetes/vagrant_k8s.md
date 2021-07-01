# Vagrant를 이용한 kubernetes 설치

> Vagrant : Virtual Box, VM Ware를 이용한 리눅스 설치 작업을 조금 더 용이하고 간편하게 설치해주는 tool

### Vagrant 설치

- https://www.vagrantup.com/

```
vagrant --version	//설치 확인
```

<br/>

<br/>

### 작업 폴더 생성

```
C:\cloud\vagrant	//생성한 작업 폴더 
cd C:\cloud\vagrant
vagrant init	//vagrant VM 초기화
```

다운받은 Vagrantfile, bash_ssh_conf_4_CeontOS.sh 파일을 작업 폴더로 복사

![image](https://user-images.githubusercontent.com/77096463/108020612-1667ba00-7060-11eb-8021-978634c4f968.png)

<br/>

**Vagrant VM 실행**

```
vagrant up
```

<br/>

**Vagrant VM 확인**

```
vagrant status
```

![image](https://user-images.githubusercontent.com/77096463/108023166-5bdab600-7065-11eb-8122-2bf408aa499e.png)

<br/>

**Vagrant 가상 이미지에 ssh 접근**

```
vagrant ssh node-1
vagrant ssh node-2
vagrant ssh master
```

<br/>

**Vagrant ssh config 설정 확인**

- identityfile 경로 알아두기 (추후 xshell로 세션 생성시 키 경로 필요)

```
vagrant ssh-config node-1
vagrant ssh-config node-2
vagrant ssh-config master
```

![image](https://user-images.githubusercontent.com/77096463/108021704-72cbd900-7062-11eb-936f-b7c87be82852.png)

<br/>

**Vagrant 기타 명령어**

```
vagrant halt [VM_name]		//노드 종료 -> poweroff status
vagrant destroy [VM_name]		//노드 삭제
vagrant reload			//reload=halt+up
```

<br/>

<br/>

### xshell로 리눅스 접속

1. xshell -> 새 세션 만들기  

- 이름 : Node1
- 호스트 : 127.0.0.1
- 포트번호 : 19211 (vagrant ssh-config node-1를 통해 확인한 포트번호)
- 사용자 키 등록 (vagrant ssh-config node-1를 통한 identityfile 경로 통해 키 가져오기)
- 사용자 이름 : vagrant

총 3개의 노드 (node1, node2, master) 를 만들었으므로 3개 모두 노드에 맞춰 동일하게 적용한다.

![image](https://user-images.githubusercontent.com/77096463/108023280-8cbaeb00-7065-11eb-86b9-562993c152f5.png)

<br/>

2. 3개의 세션을 만들면 윈도우에서 3개의 cmd 를 이용하여 각각 환경을 설정하던 작업을 xshell에서 한번에 컨트롤 할 수 있다. 

![image](https://user-images.githubusercontent.com/77096463/108022079-38af0700-7063-11eb-81b7-7ead3a435865.png)

<br/>

<br/>

### 사전 준비 (master, node 모두)

1. root 계정 변경 (pwd: vagrant)

```
su -
```

2. SELinux 설정

```
setenforce 0
sestatus
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

3. 방화벽 해제

```
systemctl stop firewalld && systemctl disable firewalld
systemctl stop NetworkManager && systemctl disable NetworkManager
```

4. SWAP 비활성화

- SWAP : 시스템 메모리가 부족할 때 하드디스크 공간을 활용하여 작업 처리 -> 하드디스크의 성능과 용량에 실질적으로 영향을 줄 수 있기 때문에 서버 운영할 땐 사용하지 않는걸 권장

```
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```

5. Iptables 커널 옵션 활성화

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
sysctl --system
```

6. 쿠버네티스를 위한 yum repository 설정

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

7. Centos Update

```
yum update
```

8. Hosts 파일 수정

- ip_address 도 편하지만, 사용자는 호스트 네임으로 접속하는게 더욱 용이하기 때문에 별도로 호스트 네임을 지정해준다.

```
vi /etc/hosts
192.168.56.10 master
192.168.56.11 node1
192.168.56.12 node2
```

![image](https://user-images.githubusercontent.com/77096463/108022358-be32b700-7063-11eb-8cb3-81131eb87d29.png)

<br/>

<br/>

### Docker 설치, 실행 (master, node 모두)

```
yum install -y yum-utils device-mapper-persistent-data lvm2 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum update && yum install docker-ce
```

```
useradd dockeradmin		//docker를 사용하는 별도의 사용자 생성
passwd dockeradmin		//pwd:dockeradmin
usermod -aG docker dockeradmin
systemctl enable --now docker && systemctl start docker
```

<br/>

<br/>

### Docker compose 설치

```
curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose -version 
```

<br/>

<br/>

### Docker compose 설치 확인

```
docker run hello-world		//hub에서 이미지 자동으로 가져옴
```

<br/>

<br/>

### Kubernetes 설치 (master, node 모두)

- Kubernetes 버전은 환경에 따라 임의로 변경 가능
- kubeadm은 클러스터링 작업을 위해 필요

```
yum install -y --disableexcludes=kubernetes kubeadm-1.15.5-0.x86_64 kubectl-1.15.5-0.x86_64 kubelet-1.15.5-0.x86_64
```

<br/>

### Kubernetes 설정 (master만)

1. 실행

```
systemctl enable --now kubelet
```

2. 초기화

```
kubeadm init --pod-network-cidr=10.96.0.0/16 --apiserver-advertise-address=192.168.56.10
```

3. 설치가 다 되었다면 아래와 같은 command가 마지막에 나오는데, 이를 복사해둔다.

```
kubeadm join 192.168.56.10:6443 --token 4ank3f.klf0z58yjyf2qo2l \
    --discovery-token-ca-cert-hash sha256:8aaea5996a430f9f8f5f54e763477caffc7dca33fa58421545d8648d4fed2470
```

4. 환경 변수 설정

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get pods --all-namespaces # 모든 pods가 Running 상태인지 확인 
```

5. Calico 기본 설치 (Kubernetes Cluster Networking plugin)

- 직접 command 입력하여 설치하는 방법과, yaml 파일로 정의하여 한 번에 실행하는 방법이 있음

```
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
kubectl get pods --all-namespaces
```

<br/>

<br/>

### Kubernetes 노드 연결 (node만)

1. 연결 

- 앞선 과정에서 master 노드에서 k8s 설정이 완료되었을 때 마지막으로 나온 command를 입력
- 마스터노드와 node1, node2 가 연결됨

```
kubeadm join 192.168.56.10:6443 --token 4ank3f.klf0z58yjyf2qo2l \
    --discovery-token-ca-cert-hash sha256:8aaea5996a430f9f8f5f54e763477caffc7dca33fa58421545d8648d4fed2470
```

2. 확인 (master에서)

- 정상적으로 모든 작업이 진행된 상황이라면, 3개의 노드가 모두 Ready 상태여야 한다.

```
kubectl get pods --all-namespaces
kubectl get nodes
```

![image](https://user-images.githubusercontent.com/77096463/108022990-00102d00-7065-11eb-9ff4-cf478fee1a7a.png)

<br/>

3. 만일 연결 시 오류 발생한다면 초기화 후 다시 실행 (node 모두 초기화)

```
kubeadm reset
```



출처 : https://github.com/joneconsulting/k8s/blob/master/kubernetes_install.md