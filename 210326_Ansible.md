**프로비저닝** : 사용자의 요구에 맞게 시스템 자원을 할당, 배치, 배포해 두었다가 필요 시 시스템을 즉시 사용할 수 있는 상태로 미리 준비해 두는 것<br>
: 서버 자원 프로비저닝, OS 프로비저닝(Vagrant), SW 프로비저닝, 스토리지 프로비저닝 등이 있으며 Kops 도 프로비저닝 툴이라고 볼 수 있다.<br>

**IoC** : Infrastructure of Code > 코드를 기반으로 인프라 관리

**Microservice**: 하나의 큰 덩어리로 만들었던 애플리케이션을 작은 단위의 서비스로 나누어 관리

---------

# Ansible

**Configuration Management Tools**

- Linux 설치 시 Python 2.7 버전이 자동으로 설치되는데, python을 기본으로 하는 Ansible은 linux 사용 시 별도로 설치할 필요가 없다.
- 하지만 Windows에는 linux가 없기 때문에 python 별도로 설치 필요
- DSL (Domain Specific Language) : 도메인에 특화되어 있는 언어
- Agent 필요 : 마스터 노드가 다른 노드에 어떤 명령을 내릴 때 Agent를 통해 작업하는 경우 

![image](https://user-images.githubusercontent.com/77096463/112560910-7b5cbf80-8e17-11eb-84e3-2914427c5294.png)

<br>

**실습 환경 구성**

![image](https://user-images.githubusercontent.com/77096463/112567523-515dca00-8e24-11eb-840e-07aff46b2586.png)

<br>

**ansible 활용하여 할 수 있는일 :**

- 설치 : apt-get, yum, homebrew 등
- 환경 설정 파일 및 스크립트 배포 : copy, template 등
- 다운로드 : get_url, git, subversion 등
- 실행 : shell, task

**ansible 결과** : ok / failed/ changed / unreachable

<br>

### 1. vagrant 기본 환경 설정

새로운 디렉터리 생성 후 vagrant 프로비저닝 예제 스크립트 실행

```
PS C:\cloud> mkdir ansible
PS C:\cloud> cd .\ansible\

PS C:\cloud\ansible> vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

 <br>

Vagrantfile 파일 내용 변경 & Vagrantfile 반영하여 프로비저닝 진행

- 만일 기존 vagrant 삭제하려면 `vagrant destroy`

```
Vagrant.configure("2") do |config|
  config.vm.define:"ansible-server" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Server"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="ansible-server"
    cfg.vm.synced_folder ".", "/vagrant", disabled: true
    cfg.vm.network "public_network", ip: "172.20.10.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 19210, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 58080
    # cfg.vm.network "forwarded_port", guest: 9000, host: 59000
    # cfg.vm.provision "shell", path: "bootstrap.sh"  
    # cfg.vm.provision "file", source: "Ansible_env_ready.yml", destination: "Ansible_env_ready.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_env_ready.yml"
    # cfg.vm.provision "shell", path: "add_ssh_auth.sh", privileged: false

    # cfg.vm.provision "file", source: "Ansible_ssh_conf_4_CentOS.yml", destination: "Ansible_ssh_conf_4_CentOS.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_ssh_conf_4_CentOS.yml"
  end
end
```

```
PS C:\cloud\ansible> vagrant up
...
1) Intel(R) Wireless-AC 9560 160MHz
2) Hyper-V Virtual Ethernet Adapter
==> ansible-server: When choosing an interface, it is usually the one that is
==> ansible-server: being used to connect to the internet.
==> ansible-server:
    ansible-server: Which interface should the network bridge to? 1
    ...
```

<br>

vagrant 상태 확인

```
PS C:\cloud\ansible> vagrant status
Current machine states:

ansible-server            running (virtualbox)
```

<br>

ansible-server 리눅스로 접속

- [vagrant@**ansible-server** ~] : Vagrantfile에서 도메인 이름은 ansible-server로 설정함

```
PS C:\cloud\ansible> vagrant ssh ansible-server
[vagrant@ansible-server ~]$
```

<br>

### 2. 필요 모듈 설치 & 자동화 작업

net-tools 설치

```
[vagrant@ansible-server ~]$ sudo yum install -y net-tools
```

<br>

리눅스 확장 패키지 설치

```
[vagrant@ansible-server ~]$ sudo yum install -y epel-release
```

<br>

Ansible Core 설치 & 설치 후 버전 확인

```
[vagrant@ansible-server ~]$ sudo yum install -y ansible

[vagrant@ansible-server ~]$ ansible --version
ansible 2.9.18
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/vagrant/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
```

<br>

만일 새로운 리눅스를 또 설치하려면 ansible, 리눅스 확장 패키지를 재설치해야하므로 이 절차를 간소화하기 위해 **자동화 작업**

- 만일 vagrant 실행 중이라면 `vagrant halt`

- Vagrantfile의 `cfg.vm.provision "shell", path: "bootstrap.sh"` 주석 해제 > **vagrant가 linux를 실행할 때 bootstrap.sh 파일을 자동 실행하여 필요한 패키지 설치**

```
#Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.define:"ansible-server" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Server"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="ansible-server"
    cfg.vm.synced_folder ".", "/vagrant", disabled: true
    cfg.vm.network "public_network", ip: "172.20.10.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 19210, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 58080
    # cfg.vm.network "forwarded_port", guest: 9000, host: 59000
    cfg.vm.provision "shell", path: "bootstrap.sh"  
    # cfg.vm.provision "file", source: "Ansible_env_ready.yml", destination: "Ansible_env_ready.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_env_ready.yml"
    # cfg.vm.provision "shell", path: "add_ssh_auth.sh", privileged: false

    # cfg.vm.provision "file", source: "Ansible_ssh_conf_4_CentOS.yml", destination: "Ansible_ssh_conf_4_CentOS.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_ssh_conf_4_CentOS.yml"
  end
end
```

```sh
#bootstrap.sh

# ! /usr/bin/env bash

yum install -y epel-release
yum install -y ansible
```

<br>

이후 다시 ansible-server 실행

```
PS C:\cloud\ansible> vagrant up
PS C:\cloud\ansible> vagrant ssh ansible-server
```

<br>

### 3. ansible-node 구축

#### ansible-node01

Vagrantfile에 ansible-node01 코드 추가

- bash_ssh_conf_4_CentOs.sh 파일 생성

```
Vagrant.configure("2") do |config|
  config.vm.define:"ansible-node01" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Node01"
        vb.customize ["modifyvm", :id, "--cpus", 1]
        vb.customize ["modifyvm", :id, "--memory", 1024]
    end
    cfg.vm.host_name="ansible-node01"
    cfg.vm.synced_folder ".", "/vagrant", disabled: false
    cfg.vm.network "public_network", ip: "172.20.10.11"
    cfg.vm.network "forwarded_port", guest: 22, host: 19211, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 80, host: 10080
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOs.sh"
  end

  config.vm.define:"ansible-server" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Server"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="ansible-server"
    cfg.vm.synced_folder ".", "/vagrant", disabled: true
    cfg.vm.network "public_network", ip: "172.20.10.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 19210, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 8080, host: 58080
    # cfg.vm.network "forwarded_port", guest: 9000, host: 59000
    cfg.vm.provision "shell", path: "bootstrap.sh"  
    # cfg.vm.provision "file", source: "Ansible_env_ready.yml", destination: "Ansible_env_ready.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_env_ready.yml"
    # cfg.vm.provision "shell", path: "add_ssh_auth.sh", privileged: false

    # cfg.vm.provision "file", source: "Ansible_ssh_conf_4_CentOS.yml", destination: "Ansible_ssh_conf_4_CentOS.yml"
    # cfg.vm.provision "shell", inline: "ansible-playbook Ansible_ssh_conf_4_CentOS.yml"
  end
end
```

```sh
#bash_ssh_conf_4_CentOs.sh

#! /usr/bin/env bash

now=$(date +"%m_%d_%Y")
cp /etc/ssh/sshd_config /etc/ssh/sshd_config_$now.backup
sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
```

<br>

ansible-node01 프로비저닝 진행

```
PS C:\cloud\ansible> vagrant up ansible-node01
Current machine states:

ansible-node01            running (virtualbox)
ansible-server            running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

<br>

서로의 ip에 ping 테스트

![image](https://user-images.githubusercontent.com/77096463/112570302-23c74f80-8e29-11eb-987b-facdb5896f57.png)

<br>

ansible-server 호스트에서 /etc/ansible/hosts 파일 하위에 아래 내용 추가

```
[vagrant@ansible-server ~]$ sudo vi /etc/ansible/hosts
```

```
[nginx]
172.20.10.11
```

<br>

#### ansible-node02

마찬가지로 Vagrantfile에 ansible-node02 코드 추가

```
Vagrant.configure("2") do |config|
  config.vm.define:"ansible-node01" do |cfg|
...
  end

  config.vm.define:"ansible-node02" do |cfg|
    cfg.vm.box = "centos/7"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Node02"
        vb.customize ["modifyvm", :id, "--cpus", 1]
        vb.customize ["modifyvm", :id, "--memory", 1024]
    end
    cfg.vm.host_name="ansible-node02"
    cfg.vm.synced_folder ".", "/vagrant", disabled: false
    cfg.vm.network "public_network", ip: "172.20.10.12"
    cfg.vm.network "forwarded_port", guest: 22, host: 19212, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 80, host: 20080
    cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOs.sh"
  end

  config.vm.define:"ansible-server" do |cfg|
 ...
  end
end
```

<br>

ansible-node02 프로비저닝 진행

```
PS C:\cloud\ansible> vagrant up ansible-node02

PS C:\cloud\ansible> vagrant status
Current machine states:

ansible-node01            running (virtualbox)
ansible-node02            running (virtualbox)
ansible-server            running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

<br>

ansible-server 호스트에서 /etc/ansible/hosts 파일 하위에 아래 내용 추가

```
[vagrant@ansible-server ~]$ sudo vi /etc/ansible/hosts
```

```
[nginx]
172.20.10.11	#node01_address
172.20.10.12	#node02_address
```

<br>

#### 호스트 이름 등록

ansible-server, ansible-node01, ansible-node02의 호스트 이름 등록

```
[vagrant@ansible-server ~]$ sudo vi /etc/hosts
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.1.1 ansible-server ansible-server

172.20.10.10 ansible-server
172.20.10.11 ansible-node01
172.20.10.12 ansible-node02
```

<br>

이후 ip 주소 대신 호스트 이름을 사용해서 ping 테스트 가능

```
[vagrant@ansible-server ~]$ ping ansible-node01
PING ansible-node01 (172.20.10.11) 56(84) bytes of data.
64 bytes from ansible-node01 (172.20.10.11): icmp_seq=1 ttl=64 time=1.78 ms
64 bytes from ansible-node01 (172.20.10.11): icmp_seq=2 ttl=64 time=1.73 ms
64 bytes from ansible-node01 (172.20.10.11): icmp_seq=3 ttl=64 time=0.928 ms
^C
--- ansible-node01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.928/1.481/1.780/0.392 ms
```

<br>

#### ansible-server에서 ansible-node01 혹은 ansible-node02 접속

비밀번호는 vagrant로 입력

```
[vagrant@ansible-server ~]$ ssh ansible-node01
The authenticity of host 'ansible-node01 (172.20.10.11)' can't be established.
ECDSA key fingerprint is SHA256:aOTryAhjdWXeL34fIAzSJgrjkyUOj5VwZnN/TgNFb8M.
ECDSA key fingerprint is MD5:4e:c1:20:55:07:e3:64:ca:d7:d3:ac:6e:73:40:13:12.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ansible-node01,172.20.10.11' (ECDSA) to the list of known hosts.
vagrant@ansible-node01's password: 
Last login: Fri Mar 26 02:47:09 2021 from 10.0.2.2
[vagrant@ansible-node01 ~]$ 
```

<br>

**:rotating_light: xshell로 ansible-server, ansible-node 접속 방법**

1. [연결 탭] 이름 : ansible-node01 / 호스트 : 127.0.0.1 / 포트 번호 : vagrant에 지정된 포트 입력 (19211)

![image](https://user-images.githubusercontent.com/77096463/112576232-3fd0ee00-8e35-11eb-8756-2020a373f74d.png)

<br>

2. [사용자 인증 탭] 사용자 이름 : vagrant / 방법 : Public Key 

-> C:\cloud\ansible\.vagrant\machines\ansible-node01\virtualbox 경로의 private_key 선택

![image](https://user-images.githubusercontent.com/77096463/112576315-642cca80-8e35-11eb-8533-eca414831d02.png)

<br>

### 4. key 설정

비대칭키 생성

- 접속을 시도하는 쪽 : public key

```
[vagrant@ansible-server ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:3RJzHrcPGlcajGswv2V0xwfErwcqF/WE7Z/UEo8dWR4 vagrant@ansible-server
The key's randomart image is:
+---[RSA 2048]----+
|             ooE+|
|             o=*+|
|          = +.=XX|
|         . X.=+BX|
|        S o BoO+o|
|          .ooB.+o|
|           oo  ..|
|                 |
|                 |
+----[SHA256]-----+

```

<br>

생성한 키를 ansible-node01과 ansible-node02로 복사

- ansible-server에서 ansible-node01 혹은 ansible-node02로 접속할 때 key값을 인증하지 않기 위해 키 값을 미리 복사해둠

```
[vagrant@ansible-server ~]$ ls -al ~/.ssh
total 16
drwx------. 2 vagrant vagrant   80 Mar 26 04:30 .
drwx------. 4 vagrant vagrant  111 Mar 26 01:41 ..
-rw-------. 1 vagrant vagrant  389 Mar 26 01:22 authorized_keys
-rw-------. 1 vagrant vagrant 1679 Mar 26 04:30 id_rsa
-rw-r--r--. 1 vagrant vagrant  404 Mar 26 04:30 id_rsa.pub
-rw-r--r--. 1 vagrant vagrant  189 Mar 26 04:26 known_hosts

[vagrant@ansible-server ~]$ ssh-copy-id root@ansible-node01
[vagrant@ansible-server ~]$ ssh-copy-id root@ansible-node02

[vagrant@ansible-server ~]$ ssh-copy-id vagrant@ansible-node01
[vagrant@ansible-server ~]$ ssh-copy-id vagrant@ansible-node02
```

<br>

명령어 수행 후 ansible-server에서 root@ansible-node01과 ansible-node01로 접속하면 패스워드 물어보지 않는다.

```
[vagrant@ansible-server ~]$ ssh root@ansible-node01
[root@ansible-node01 ~]# 

[vagrant@ansible-server ~]$ ssh ansible-node01
Last login: Fri Mar 26 04:27:58 2021 from 172.20.10.10
[vagrant@ansible-node01 ~]$ 
```

<br>

### 5. ansible all 명령어

> 자주 사용되는 모듈 : ping, shell, user, copy, yum, apt

ansible 호스트에 등록된 모든 호스트(all) 에 ping 명령어 전달

```
[vagrant@ansible-server ~]$ ansible all -m ping
172.20.10.12 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
172.20.10.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

<br>

ansible-node01과 ansible-node02 노드에 명령어를 일일이 입력하지 않아도 ansible-server 노드에서 아래 명령어 통해 한꺼번에 결과 확인 가능

- `-m` : 모듈 선택

```
[vagrant@ansible-server ~]$ ansible all -m shell -a "uptime"
172.20.10.12 | CHANGED | rc=0 >>
 05:56:07 up  2:59,  2 users,  load average: 0.08, 0.03, 0.05
172.20.10.11 | CHANGED | rc=0 >>
 05:56:08 up  3:21,  2 users,  load average: 0.24, 0.06, 0.06
```

<br>

/etc/ansible/hosts 파일 아래와 같이 코드 추가 

```
[nginx]
172.20.10.11
172.20.10.12

[webserver]
172.20.10.11

[backupserver]
172.20.10.12
```

아래와 같이 필요한 그룹에 명령어를 수행 가능
```
[vagrant@ansible-server ~]$ ansible webserver -m ping
172.20.10.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}

[vagrant@ansible-server ~]$ ansible backupserver -m shell -a "uptime"
172.20.10.12 | CHANGED | rc=0 >>
 06:35:21 up  3:38,  2 users,  load average: 0.24, 0.06, 0.06
```

<br>

`--list-hosts` : 어떤 호스트가 있는지 확인 가능

```
[vagrant@ansible-server ~]$ ansible backupserver -m shell -a "uptime" --list-hosts
  hosts (1):
    172.20.10.12
```

<br>

### 6. ansible-node03 추가

Vagrantfile 코드 추가 후 ansible-node03 프로비저닝

```
Vagrant.configure("2") do |config|
  config.vm.define:"ansible-node01" do |cfg|
...
  end

  config.vm.define:"ansible-node02" do |cfg|
...
  end

  config.vm.define:"ansible-node03" do |cfg|
    cfg.vm.box = "ubuntu/trusty64"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Node03"
        vb.customize ["modifyvm", :id, "--cpus", 1]
        vb.customize ["modifyvm", :id, "--memory", 1024]
    end
    cfg.vm.host_name="ansible-node03"
    cfg.vm.synced_folder ".", "/vagrant", disabled: false
    cfg.vm.network "public_network", ip: "172.20.10.13"
    cfg.vm.network "forwarded_port", guest: 22, host: 19213, auto_correct: false, id: "ssh"
    cfg.vm.network "forwarded_port", guest: 80, host: 30080
    #cfg.vm.provision "shell", path: "bash_ssh_conf_4_CentOs.sh"
  end

  config.vm.define:"ansible-server" do |cfg|
...
  end
end
```

```
PS C:\cloud\ansible> vagrant up ansible-node03

PS C:\cloud\ansible> vagrant status
Current machine states:

ansible-node01            running (virtualbox)
ansible-node02            running (virtualbox)
ansible-node03            running (virtualbox)
ansible-server            running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

<br>

/etc/hosts와 /etc/ansible/hosts 경로 하위에 ansible-node03 호스트와 ip 추가

```
[vagrant@ansible-server ~]$ sudo vi /etc/hosts
...
172.20.10.10 ansible-server
172.20.10.11 ansible-node01
172.20.10.12 ansible-node02
172.20.10.13 ansible-node03


[vagrant@ansible-server ~]$ sudo vi /etc/ansible/hosts
...
[nginx]
172.20.10.11
172.20.10.12
172.20.10.13

[webserver]
172.20.10.11

[backupserver]
172.20.10.12

[ubuntu]
172.20.10.13
```

<br>

ansible-node03에 인증 정보 전달 > **실패**

- `ssh-copy-id root@ansible-node03` 는 이전과 동일하게 진행하면 Permission Denied 에러 발생

```
[vagrant@ansible-server ~]$ ssh-copy-id root@ansible-node03
[vagrant@ansible-server ~]$ ssh-copy-id vagrant@ansible-node03
```

<br>

ansible-node03에서 28번째 줄 값을 yes로 변경

```
vagrant@ansible-node03:~$ sudo vi /etc/ssh/sshd_config
```

```
...
 26 # Authentication:
 27 LoginGraceTime 120
 28 PermitRootLogin yes
 29 StrictModes yes
 ...
```

데몬 재시작하여 sshd_config 변경 사항 반영
```
vagrant@ansible-node03:~$ sudo /etc/init.d/ssh restart
```

<br>

위의 과정 완료 후 root@anisible-node03에 인증 정보 재전달 > **성공**

```
[vagrant@ansible-server ~]$ ssh-copy-id root@ansible-node03
...
root@ansible-node03's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@ansible-node03'"
and check to make sure that only the key(s) you wanted were added.
```

<br>

---------------

**:rotating_light: ansible-node03 프로비저닝 중 Connection disconnect (Timeout) 에러 발생** <br>
: `vagrant destroy ansible-node03` 수행<br>
(기존에 `vagrant reload ansible-node03` 혹은 `vagrant halt` -> `vagrant up` 명령어도 계속 타임아웃 에러가 나서 아예 해당 노드를 지운 후 재작업)

![image](https://user-images.githubusercontent.com/77096463/112598111-7452a300-8e51-11eb-9f27-cc75d839bd27.png)

<br>

----------

Disk Space 확인

```
[vagrant@ansible-server ~]$ ansible all -m shell -a "df -h" 
172.20.10.13 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
udev            493M   12K  493M   1% /dev
tmpfs           100M  372K  100M   1% /run
/dev/sda1        40G  1.5G   37G   4% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
none            5.0M     0  5.0M   0% /run/lock
none            497M     0  497M   0% /run/shm
none            100M     0  100M   0% /run/user
172.20.10.12 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.7M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G  3.0G   38G   8% /
tmpfs           100M     0  100M   0% /run/user/1000
172.20.10.11 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.7M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G  3.0G   38G   8% /
tmpfs           100M     0  100M   0% /run/user/1000
```

<br>

Memory 확인

```
172.20.10.13 | CHANGED | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:          993M       260M       733M       384K        13M       122M
-/+ buffers/cache:       125M       868M
Swap:           0B         0B         0B
172.20.10.12 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           990M        102M        741M        6.7M        147M        746M
Swap:          2.0G          0B        2.0G
172.20.10.11 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           990M        104M        738M        6.7M        147M        743M
Swap:          2.0G          0B        2.0G
```

<br>

### 7. 사용자 관리

ansible-node1, 2, 3 노드 각각 사용자 확인

```
[vagrant@ansible-node01 ~]$ cat /etc/passwd	#centOS
[vagrant@ansible-node02 ~]$ cat /etc/passwd	#centOS
[vagrant@ansible-node03 ~]$ cat /etc/passwd	#Ubuntu
```

<br>

루트 권한으로 변경 후 ,각 노드에 한꺼번에 사용자(test1/1234) 추가<br>:heavy_exclamation_mark: 원인은 모르겠으나 처음에는 3개의 노드 모두 'Failed to connect to the host via ssh' 에러가 나더니 명령어 계속 실행하니 결국엔 순차적으로 실행된다.


```
[root@ansible-server ~]# ansible all -m user -a "user=test1 password=1234" -k
SSH password: 
[WARNING]: The input password appears not to have been hashed. The 'password'
argument must be encrypted for this module to work properly.
172.20.10.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "append": false, 
    "changed": false, 
    "comment": "", 
    "group": 1001, 
    "home": "/home/test1", 
    "move_home": false, 
    "name": "test1", 
    "password": "NOT_LOGGING_PASSWORD", 
    "shell": "/bin/bash", 
    "state": "present", 
    "uid": 1001
}
172.20.10.13 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 1002, 
    "home": "/home/test1", 
    "name": "test1", 
    "password": "NOT_LOGGING_PASSWORD", 
    "shell": "", 
    "state": "present", 
    "system": false, 
    "uid": 1002
}
172.20.10.12 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 1001, 
    "home": "/home/test1", 
    "name": "test1", 
    "password": "NOT_LOGGING_PASSWORD", 
    "shell": "/bin/bash", 
    "state": "present", 
    "system": false, 
    "uid": 1001
}
```

<br>

각 노드에서 추가된 사용자 정보(test1) 확인

![image](https://user-images.githubusercontent.com/77096463/112603739-c0551600-8e58-11eb-8b17-1bf903805187.png)

<br>

다시 vagrant 계정으로 돌아와서 test_server.txt 임의의 파일 생성

```
[vagrant@ansible-server ~]$ cat test_server.txt
Hi, there
Hello, Ansible.
```

<br>

생성한 파일을 ansible-node01, 02, 03 노드로 배포

```
[vagrant@ansible-server ~]$ ansible all -m copy -a "src=./test_server.txt dest=/home/vagrant"
172.20.10.13 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "3e2c2e116653cfff4054dbe1ade8d02decfc13de", 
    "dest": "/home/vagrant/test_server.txt", 
    "gid": 1000, 
    "group": "vagrant", 
    "md5sum": "b42f3e03293b06380cf1e88e41b0a8fa", 
    "mode": "0664", 
    "owner": "vagrant", 
    "size": 26, 
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1616747808.53-22932-19993333380715/source", 
    "state": "file", 
    "uid": 1000
}
172.20.10.12 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "3e2c2e116653cfff4054dbe1ade8d02decfc13de", 
    "dest": "/home/vagrant/test_server.txt", 
    "gid": 1000, 
    "group": "vagrant", 
    "md5sum": "b42f3e03293b06380cf1e88e41b0a8fa", 
    "mode": "0664", 
    "owner": "vagrant", 
    "secontext": "unconfined_u:object_r:user_home_t:s0", 
    "size": 26, 
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1616747809.51-22930-269847588699705/source", 
    "state": "file", 
    "uid": 1000
}
172.20.10.11 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "3e2c2e116653cfff4054dbe1ade8d02decfc13de", 
    "dest": "/home/vagrant/test_server.txt", 
    "gid": 1000, 
    "group": "vagrant", 
    "md5sum": "b42f3e03293b06380cf1e88e41b0a8fa", 
    "mode": "0664", 
    "owner": "vagrant", 
    "secontext": "unconfined_u:object_r:user_home_t:s0", 
    "size": 26, 
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1616747809.3-22928-96318820200933/source", 
    "state": "file", 
    "uid": 1000
}
```

<br>

각 노드에서 실제로 파일이 이동되었는지 확인

![image](https://user-images.githubusercontent.com/77096463/112604850-f9da5100-8e59-11eb-9b56-d17c4e4975f1.png)

<br>

:memo:TODO : ansible-node05 노드 설치 (windows 기반) > 설치하는데 30분 정도 소요됨

```
# Ansible-Node05 (Windows2012R2)
  config.vm.define:"ansible-node05" do |cfg|
    cfg.vm.box = "opentable/win-2012r2-standard-amd64-nocm"
    cfg.vm.provider:virtualbox do |vb|
        vb.name="Ansible-Node05"
        vb.customize ["modifyvm", :id, "--cpus", 2]
        vb.customize ["modifyvm", :id, "--memory", 2048]
    end
    cfg.vm.host_name="ansible-node05"
    cfg.vm.synced_folder ".", "/vagrant", disabled: true
    cfg.vm.network "public_network", ip: "172.20.10.15"
    cfg.vm.network "forwarded_port", guest: 22, host: 19215, auto_correct: false, id: "ssh"
    cfg.vm.provision "shell", inline: "netsh firewall set opmode disable"
  end
```

