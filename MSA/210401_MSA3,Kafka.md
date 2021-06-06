# AWS & Ansible

> ansible-server 54.174.86.113 172.31.26.185
> ansible-node01 54.163.43.181 172.31.27.220
> ansible-node02 54.159.42.36 172.31.24.143

### 1. Ping test

ansible-ec2-node01 ping 테스트

```
[ec2-user@ip-172-31-26-185 ~]$ ping 172.31.27.220
PING 172.31.27.220 (172.31.27.220) 56(84) bytes of data.
64 bytes from 172.31.27.220: icmp_seq=1 ttl=255 time=0.723 ms
64 bytes from 172.31.27.220: icmp_seq=2 ttl=255 time=0.608 ms
64 bytes from 172.31.27.220: icmp_seq=3 ttl=255 time=0.578 ms
^C
--- 172.31.27.220 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2029ms
rtt min/avg/max/mdev = 0.578/0.636/0.723/0.065 ms
```
ansible-ec2-node02 ping 테스트
```
[ec2-user@ip-172-31-26-185 ~]$ ping 172.31.24.143
PING 172.31.24.143 (172.31.24.143) 56(84) bytes of data.
64 bytes from 172.31.24.143: icmp_seq=1 ttl=255 time=0.816 ms
64 bytes from 172.31.24.143: icmp_seq=2 ttl=255 time=0.638 ms
64 bytes from 172.31.24.143: icmp_seq=3 ttl=255 time=0.638 ms
^C
--- 172.31.24.143 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2035ms
rtt min/avg/max/mdev = 0.638/0.697/0.816/0.086 ms
```

<br>

### 2. SSH 접속

devlopment 보안 그룹 확인하면 ansible 서버 내에서 통신하지 못하도록 돼있음 (ssh 접속이 아예 안됨) > **인바운드 규칙으로 SSH 유형/devlopment보안그룹 소스 등록**

![image](https://user-images.githubusercontent.com/77096463/113227176-0254e080-92cd-11eb-9bdd-9b0fc526775a.png)

<br>

ssh 접속은 되나 Permission denied 에러 발생

```
[ec2-user@ip-172-31-26-185 ~]$ ssh 172.31.27.220
The authenticity of host '172.31.27.220 (172.31.27.220)' can't be established.
ECDSA key fingerprint is SHA256:7Tywb3MMzGQ8bn+0mVWLnXOpHKrkm0JQLKEwByhZKRk.
ECDSA key fingerprint is MD5:23:98:01:e9:4c:f0:e3:70:a6:72:4b:9d:14:2a:ca:45.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.31.27.220' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

[ec2-user@ip-172-31-26-185 ~]$ ssh 172.31.24.143
The authenticity of host '172.31.24.143 (172.31.24.143)' can't be established.
ECDSA key fingerprint is SHA256://Mj/B/6+J8nCuOeGyH1+Hc+/vI7mmbT1v3T14Ud2M4.
ECDSA key fingerprint is MD5:98:30:ff:93:72:a9:14:ec:84:7e:ab:6e:88:d6:c6:c1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.31.24.143' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

<br>

키 생성 > 이후 ssh-copy-id를 통해 키를 전달해야 하나 이 또한 ssh를 이용하므로 권한 문제 발생

```
[ec2-user@ip-172-31-26-185 ~]$ ssh-keygen
```

<br>

따라서 ansible-server의 id_rsa.pub 키를 각각의 노드가 가진 authorized_keys 파일에 등록 필요<br>
**즉, id_rsa.pub의 내용을 authorized_keys 내용 모두 지운 후 붙여넣기**

![image](https://user-images.githubusercontent.com/77096463/113228127-27e2e980-92cf-11eb-8f74-d3c7f19130c1.png)

<br>

이후 각 노드로 ssh 접속 가능 확인

ansible-ec2-node01

```
[ec2-user@ip-172-31-26-185 .ssh]$ ssh 172.31.27.220
Last login: Thu Apr  1 00:24:14 2021 from 175.125.198.37

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
[ec2-user@ip-172-31-27-220 ~]$ 
```
ansible-ec2-node02
```
[ec2-user@ip-172-31-26-185 .ssh]$ ssh 172.31.24.143
Last login: Thu Apr  1 00:24:47 2021 from 175.125.198.37

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
[ec2-user@ip-172-31-24-143 ~]$ 
```

<br>

### 3. ansible hosting 수정 후 테스트

/etc/ansible/hosts에 노드 정보 입력

```
[ec2-user@ip-172-31-26-185 .ssh]$ sudo vi /etc/ansible/hosts
```

```
[ec2]
172.31.27.220
172.31.24.143

[node1]
172.31.27.220

[node2]
172.31.24.143
```

<br>

ec2그룹에 ping 테스트

```
[ec2-user@ip-172-31-26-185 .ssh]$ ansible ec2 -m ping

172.31.24.143 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}

172.31.27.220 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```
ec2그룹에 uptime 테스트
```
[ec2-user@ip-172-31-26-185 .ssh]$ ansible ec2 -m shell -a "uptime"

172.31.27.220 | CHANGED | rc=0 >>
 01:17:48 up  1:04,  2 users,  load average: 0.00, 0.00, 0.00

172.31.24.143 | CHANGED | rc=0 >>
 01:17:48 up  1:04,  2 users,  load average: 0.00, 0.00, 0.00
```
node1 메모리 테스트
```
[ec2-user@ip-172-31-26-185 .ssh]$ ansible node1 -m shell -a "free -h"

172.31.27.220 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           983M         91M        694M        488K        197M        759M
Swap:            0B          0B          0B
```
node2 Disk Space 테스트
```
[ec2-user@ip-172-31-26-185 .ssh]$ ansible node2 -m shell -a "df -h"

172.31.24.143 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        482M     0  482M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  488K  492M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1       20G  1.6G   19G   8% /
tmpfs            99M     0   99M   0% /run/user/1000
tmpfs            99M     0   99M   0% /run/user/0
```

<br>

---

# MSA 표준 구성요소

**전체적인 구조 (매우 중요)**

![image](https://user-images.githubusercontent.com/77096463/113247668-73f55480-92f6-11eb-9446-182d9b539704.png)

<br>

순서도

![image](https://user-images.githubusercontent.com/77096463/113247726-88d1e800-92f6-11eb-9dad-60534cc6ecec.png)

<br>

---



# Kafka - Message Queuing Server 

**Message Queuing Server 개요 (예시)**

![image](https://user-images.githubusercontent.com/77096463/113247974-05fd5d00-92f7-11eb-9565-bace5aa520da.png)

<br>

**Kafka**

- Apache Software Foundation의 Scalar언어로 된 오픈 소스 메시지 브로커 (서버) 프로젝트

- Producer/Consumer 분리
- 메시지를 여러 Consumer에게 허용
- 높은 처리량을 위한 메시지 최적화
- Scale-out 가능
- Eco-system

<br>

**Kafka Broker**

- 실행 된 Kafka 애플리케이션 서버 중 1대
- 3대 이상의 Broker Cluster 구성
- Zookeeper 연동
  - 메타데이터 (Broker ID, Controller ID 등) 저장
- n대 Broker 중 1대는 Controller 기능 수행
  - 각 Broker에게 담당 파티션 (메시지 저장 공간) 할당 수행
  - Broker 정상 동작 모니터링 관리
  - Zookeeper에게 저장됨

<br>

**Record**

- 객체를 Producer에서 Consumer로 전달하기 위해 Kafka 내부에 byte 형태로 저장할 수 있도록 직렬화(byte로 전환)/역직렬화하여 사용
- 기본 제공 직렬화 Class: StringSerializer, ShortSerializer 등
- 커스텀 직렬화 Class 사용 가능: Customer Object 직렬화/역직렬화 가능

<br>

**Topic**

- 관련성 있는 파티션을 모아둔 그룹
- n개의 파티션 할당 가능
- 메시지 분류 단위

<br>

### APACHE KAFKA QUICKSTART

> 참고 : http://kafka.apache.org/quickstart

### 1. 필요한 파일 다운로드

Kafka 홈페이지에서 다운로드

```
C:\Users\Lenovo>cd C:\cloud\kafka_2.13-2.7.0

C:\cloud\kafka_2.13-2.7.0>dir
 C 드라이브의 볼륨: Windows-SSD
 볼륨 일련 번호: 567B-4B2A

 C:\cloud\kafka_2.13-2.7.0 디렉터리

2021-04-01  오후 02:55    <DIR>          .
2021-04-01  오후 02:55    <DIR>          ..
2021-04-01  오후 02:55    <DIR>          bin
2021-04-01  오후 02:55    <DIR>          config
2021-04-01  오후 02:55    <DIR>          libs
2020-12-16  오후 10:58            29,975 LICENSE
2020-12-16  오후 10:58               337 NOTICE
2021-04-01  오후 02:55    <DIR>          site-docs
               2개 파일              30,312 바이트
               6개 디렉터리  302,437,928,960 바이트 남음
```

<br>

Java SE 11 (LTS) > jdk-11.0.10_windows-x64_bin.exe 다운로드 후 버전 확인

```
[C:\cloud\kafka_2.13-2.7.0]$ java --version
java 11.0.10 2021-01-19 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.10+8-LTS-162)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.10+8-LTS-162, mixed mode)

[C:\cloud\kafka_2.13-2.7.0]$ javac --version
javac 11.0.10
```

<br>

### 2. Kafka 환경 설정

Zookeeper 서버 시작 > 아래 binding port~ 문구가 나오면 정상 연결됨!

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\zookeeper-server-start.bat config\zookeeper.properties
...
[2021-04-01 15:30:33,138] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
...
```

kafka 서버 기동 > 아래 Awaiting socket connections~ 문구가 나오면 정상 연결됨!

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-server-start.bat config\server.properties 
...
[2021-04-01 15:49:57,678] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.Acceptor)
...
```

<br>

### 3. Topic 생성

quickstart-events 이름의 topic 생성

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --create --topic quickstart-events --bootstrap-server localhost:9092
Created topic quickstart-events.
```
topic 리스트 확인
```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
quickstart-events
```
quickstart-events topic 상세 정보 확인 
```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic: quickstart-events	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: quickstart-events	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

<br>

### 4. Topic의 Event 읽기 작업

발송될 메시지를 받는 Consumer 기동

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-console-consumer.bat --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```

<br>

### 5. Topic에 Event 생성

Producer 기동 후 메시지를 입력하면 Consumer에서 이를 받아옴을 확인

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-console-producer.bat --topic quickstart-events --bootstrap-server localhost:9092
>Hello, World!
>Hi, there!
>This is my first event
>This is my second event
```

![image](https://user-images.githubusercontent.com/77096463/113257812-17019a80-9306-11eb-9cc5-e195899dd6e7.png)

<br>

2개의 Consumer 기동 후 Producer에서 메시지 전송

- `--from-beginning` 옵션을 빼면 consumer 실행 시점부터의 메시지를 받아옴 (옵션을 붙이면 그동안 받아온 메시지 모두 출력)

```
C:\cloud\kafka_2.13-2.7.0>bin\windows\kafka-console-producer.bat --topic quickstart-events --bootstrap-server localhost:9092
>!!!!!This is my THIRD event!!!!!
```

![image](https://user-images.githubusercontent.com/77096463/113258326-c6d70800-9306-11eb-8b27-cc39dfa61b60.png)

<br>

### 6. Topic 삭제

첫 번째 방법 : kafka-broker에서 직접 삭제

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --delete --topic quickstart-events --bootstrap-server localhost:9092
```

<br>

두 번째 방법 : zookeeper에서 직접 삭제 (**권장하지 않음**)

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --delete --topic quickstart-events --zookeeper localhost:2181
Topic quickstart-events is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
```

그러면 강제로 kafka-server가 삭제됨

![image](https://user-images.githubusercontent.com/77096463/113259756-73fe5000-9308-11eb-89dd-d5a88acf7c6c.png)

zookeeper-server 내역에서도 kafka-server가 끊긴 메시지 확인

![image](https://user-images.githubusercontent.com/77096463/113259847-8d070100-9308-11eb-89aa-ef98f56b890b.png)

<u>C:\tmp 에 kafka-logs와 zookeeper 로그가 남아있기 때문에 이를 모두 삭제</u>하고 zookeeper-server와 kafka-server 재실행 > 이후 topic 내역 확인하면 모두 삭제되었기 때문에 아무 내용도 출력하지 않음

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092

[C:\cloud\kafka_2.13-2.7.0]$ 
```

<br>

<br>

### Python에서 message 전송

### 1. Consumer 테스트

가상환경 실행 후 kafka-python 모듈 다운로드

```
(base) C:\Users\Lenovo>cd C:\cloud\kafka_client

(base) C:\cloud\kafka_client>conda activate base

(base) C:\cloud\kafka_client>pip install kafka-python
```

<br>

C:\cloud\kafka_client\kafka_consumer.py 파일 생성

```python
from kafka import KafkaConsumer
from json import loads
import time

consumer = KafkaConsumer('quickstart-events',
            bootstrap_servers=['127.0.0.1:9092'],
            auto_offset_reset='earliest',
            enable_auto_commit=True,
            group_id='my-group',
            # value_deserializer=lambda x: loads(x.decode('utf-8')),
            consumer_timeout_ms=1000)

start = time.time()

for message in consumer:
    topic = message.topic
    partition = message.partition
    offset = message.offset
    key = message.key
    value = message.value
    print("Topic:{},Partition:{},Offset:{},Key:{},Value:{}".format(topic, partition, offset, key, value))

print("Elapsed: ", (time.time()-start))
```

<br>

producer에서 메시지 전송 후 kafka_consumer.py 파일 실행

![image](https://user-images.githubusercontent.com/77096463/113264963-8c716900-930e-11eb-8eb9-7f564cd4660e.png)

```
(base) C:\cloud\kafka_client>python kafka_consumer.py
Topic:quickstart-events,Partition:0,Offset:0,Key:None,Value:b'Hello, Kafka!'
Topic:quickstart-events,Partition:0,Offset:1,Key:None,Value:b'Hello, World!'
Topic:quickstart-events,Partition:0,Offset:2,Key:None,Value:b'This is my first message!'
Elapsed:  1.2316091060638428
```

<br>

### 2. Producer 테스트

my_users_topic 토픽 생성

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-topics.bat --create --topic my_topic_users --bootstrap-server localhost:9092
Created topic my_topic_users.
```

kafka_producer.py 파일 생성

- 기존의 메시지는 직렬화되지 않았으므로 기존의 quickstart_events 토픽으로 설정하면 충돌할 수 있음 -> kafka_consumer.py의 토픽 변경 & 역직렬화 주석 해제

```python
from kafka import KafkaProducer
from json import dumps
import time

# dict (key,value) -> object
# str -> json

# acks 수치가 낮을 수록 체크하지 않겠다는 뜻 -> 성능 올라감
producer = KafkaProducer(acks=0, 
                compression_type='gzip',
                bootstrap_servers=['127.0.0.1:9092'],
                value_serializer=lambda x: dumps(x).encode('utf-8')
                )

start = time.time()

# 10개의 data값 전달
for i in range(10):
    data = {'name': 'Haerim-' + str(i)}
    producer.send('my_topic_users', value=data)
    producer.flush()

print("Done. Elapsed time: ", (time.time()-start))
```

kafka-producer 기동

```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-console-producer.bat --topic my_topic_users --bootstrap-server localhost:9092
```

kafka_producer.py 파일 실행
```
(base) C:\cloud\kafka_client>python kafka_producer.py
Done. Elapsed time:  0.029314041137695312
```

kafka-consumer 확인
```
[C:\cloud\kafka_2.13-2.7.0]$ bin\windows\kafka-console-consumer.bat --topic my_topic_users --from-beginning --bootstrap-server localhost:9092


{"name": "Haerim-0"}
{"name": "Haerim-1"}
{"name": "Haerim-2"}
{"name": "Haerim-3"}
{"name": "Haerim-4"}
{"name": "Haerim-5"}
{"name": "Haerim-6"}
{"name": "Haerim-7"}
{"name": "Haerim-8"}
{"name": "Haerim-9"}
```

<br>

kafka_producer.py 파일의 data 영역 수정 후 kafka_producer.py 파일 재실행

```python
...
for i in range(10):
    #data = {'name': 'Haerim-' + str(i)}
    data = {"schema":{"type":"struct","fields":[{"type":"int32","field":"id"},{"type":"string","field":"user_id"},{"type":"string","field":"pwd"},{"type":"string","field":"NAME"},{"type":"int64","name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"created_at"}],"name":"users"},"payload":{"id":10,"user_id":"new_test10","pwd":"new_pwd10","NAME":"NEW TEST USER10","created_at":1615349727000}}
    ...
```

kafka-consumer 확인
```
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
{"schema": {"type": "struct", "fields": [{"type": "int32", "field": "id"}, {"type": "string", "field": "user_id"}, {"type": "string", "field": "pwd"}, {"type": "string", "field": "NAME"}, {"type": "int64", "name": "org.apache.kafka.connect.data.Timestamp", "version": 1, "field": "created_at"}], "name": "users"}, "payload": {"id": 10, "user_id": "new_test10", "pwd": "new_pwd10", "NAME": "NEW TEST USER10", "created_at": 1615349727000}}
```

<br>



