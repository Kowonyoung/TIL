# Managing DNS Records with AWS Route 53

Route 53 : 가용성과 확장성이 우수한 DNS 웹 서비스 -> 도메인 등록, DNS 라우팅, 상태 확인 조합하여 실행

### Lab 구성도

![image-20210303094432829](210303.assets/image-20210303094432829.png)

<br>

### Create the First EC2 Instance

EC2 > 인스턴스 생성

- AMI : Amazon Linux2 AMI
- 인스턴스 유형 : t2.micro (Default)
- 지역: us-east-1e
- 퍼블릭 IP 자동 할당 : 활성화 
- 고급 세부 정보 : 사용자 데이터 아래 이미지와 같은 내용 입력

![image-20210303094912698](210303.assets/image-20210303094912698.png)

<br>

보안 그룹 구성 설정

- 보안 그룹 할당 : 새 보안 그룹 생성
- 보안 그룹 이름 : SG
- 규칙 추가 : 유형 - HTTP, 소스 - 사용자 지정

![image-20210303095002182](210303.assets/image-20210303095002182.png)

<br>

키 페어 생성 -> 키 페어 이름 : R53KP

![image-20210303095035968](210303.assets/image-20210303095035968.png)

<br> 여기까지 첫 번째 인스턴스 생성 완료

### Create the Second EC2 Instance

바로 두 번째 인스턴스를 생성한다.

- AMI : Amazon Linux2 AMI
- 인스턴스 유형 : t2.micro (Default)
- 지역: **us-east-1d**
- 퍼블릭 IP 자동 할당 : 활성화 
- 고급 세부 정보 : 사용자 데이터 아래 이미지와 같은 내용 입력

![image-20210303095141794](210303.assets/image-20210303095141794.png)

![image-20210303095151932](210303.assets/image-20210303095151932.png)

보안 그룹 구성 설정 -> 이번에는 '**기존 보안 그룹 선택**' 
- 첫 번째 인스턴스 생성 시 만들었던 **SG** 보안 그룹 선택

![image-20210303095216729](210303.assets/image-20210303095216729.png)

<br>

키 페어 설정 -> '**기존 키 페어 (R53KP)**' 선택

![image-20210303095241884](210303.assets/image-20210303095241884.png)

<br> 여기까지 두번째 인스턴스 생성 완료<br>
실행 중인 인스턴스의 퍼블릭 IP주소를 주소창에 입력하면 Amazon Linux2 AMI 테스트 페이지를 확인할 수 있다.


![image-20210303104200454](210303.assets/image-20210303104200454.png)

<br>

### Create the Application Load Balancer

로드 밸런싱 > 로드 밸런서 > Load Balancer 생성 클릭
- 종류 : Application Load Balancers, Network Load Balancers, and Classic Load Balancers -> 이 중에서 **Application Load Balancers** 선택 
- 이름 : ELB
- 체계 : 인터넷 경계
- IP 주소 유형 : ipv4
- 가용 영역 : us-east-1d, us-east-1e (**둘 다 선택**)

![image-20210303095516927](210303.assets/image-20210303095516927.png)

<br>

보안 그룹 구성 설정 -> 새 보안 그룹 생성
- 이름 : ELBSG
- 나머지 설정은 default

![image-20210303095548973](210303.assets/image-20210303095548973.png)

<br>

상태 검사 설정
- 정상 임계 값 : 2 -> 애플리케이션 로드 밸런서 상태 검사 시 속도를 높여준다
  - 정상 임계 값 : EC2 인스턴스를 정상으로 선언하기 전까지 발생하는 연속적인 상태 확인 성공 횟수

![image-20210303095818857](210303.assets/image-20210303095818857.png)

<br>

인스턴스 항목에서 모든 인스턴스를 선택한 후 '등록된 항목에 추가' 클릭 -> 등록된 대상 영역에 선택한 인스턴스가 등록됨을 확인

![image-20210303095848634](210303.assets/image-20210303095848634.png)

<br>

ELB 로드밸런서의 DNS 이름을 주소창에 입력하면 Amazon Linux2 AMI 테스트 페이지에 접속된다. 

![image-20210303100202945](210303.assets/image-20210303100202945.png)

![image-20210303104115470](210303.assets/image-20210303104115470.png)

<br>

## Configuring Route 53 DNS Record Sets

![image-20210303100310467](210303.assets/image-20210303100310467.png)

![image-20210303100341403](210303.assets/image-20210303100341403.png)

#### Make Route 53 always point to the correct IP address - The Command Line Tool `dig`

```
[ec2-user@ip-10-0-0-103 ~]$ dig www.linuxacademy.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.2 <<>> www.linuxacademy.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52092
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.linuxacademy.com.          IN      A

;; ANSWER SECTION:
www.linuxacademy.com.   60      IN      A       52.87.130.251
www.linuxacademy.com.   60      IN      A       54.157.29.13
www.linuxacademy.com.   60      IN      A       35.153.84.11
www.linuxacademy.com.   60      IN      A       52.86.201.244

;; Query time: 26 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Wed Mar 03 01:06:56 UTC 2021
;; MSG SIZE  rcvd: 113
```

```
[ec2-user@ip-10-0-0-103 ~]$ dig www.linuxacademy.com +trace

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.2 <<>> www.linuxacademy.com +trace
;; global options: +cmd
.                       518400  IN      NS      J.ROOT-SERVERS.NET.
.                       518400  IN      NS      K.ROOT-SERVERS.NET.
.                       518400  IN      NS      L.ROOT-SERVERS.NET.
.                       518400  IN      NS      M.ROOT-SERVERS.NET.
.                       518400  IN      NS      A.ROOT-SERVERS.NET.
.                       518400  IN      NS      B.ROOT-SERVERS.NET.
.                       518400  IN      NS      C.ROOT-SERVERS.NET.
.                       518400  IN      NS      D.ROOT-SERVERS.NET.
.                       518400  IN      NS      E.ROOT-SERVERS.NET.
.                       518400  IN      NS      F.ROOT-SERVERS.NET.
.                       518400  IN      NS      G.ROOT-SERVERS.NET.
.                       518400  IN      NS      H.ROOT-SERVERS.NET.
.                       518400  IN      NS      I.ROOT-SERVERS.NET.
;; Received 239 bytes from 10.0.0.2#53(10.0.0.2) in 25 ms
```

### Create the Record Sets

![image-20210303101146109](210303.assets/image-20210303101146109.png)

![image-20210303101234257](210303.assets/image-20210303101234257.png)

![image-20210303101501615](210303.assets/image-20210303101501615.png)

![image-20210303103232112](210303.assets/image-20210303103232112.png)

![image-20210303103255960](210303.assets/image-20210303103255960.png)

![image-20210303103318087](210303.assets/image-20210303103318087.png)

# AWS ELB Connectivity Troubleshooting Scenario

## Lab 구성도

![img](210303.assets/lab_diagram_Orion Papers 2018 Copy - Page 18.png)

<br>

### First Attempt to Connect

인스턴스 각각 서비스 여부 확인 -> 모두 정상 작동
- Web1

![image-20210303111924422](210303.assets/image-20210303111924422.png)

<br>

- Web 2

![image-20210303111935282](210303.assets/image-20210303111935282.png)

<br>

로드 밸런서의 DNS 주소로 접속 -> **작동하지 않음**
- 로드 밸런서 정보 확인

![image-20210303112218696](210303.assets/image-20210303112218696.png)

<br>

- DNS 주소로 접속 
  - **'사이트에 연결할 수 없음' : 서비스 포트가 유효하지 않기 때문에 연결할 수 없음**

![image-20210303112204442](210303.assets/image-20210303112204442.png)

<br>

### Solutions

로드밸런서의 보안 그룹 설정 확인
- 인바운드 규칙에 80번 포트가 누락되어 있음을 확인

![image-20210303112352007](210303.assets/image-20210303112352007.png)

<br>

인바운드 규칙 편집 > 80번 포트 추가하기

![image-20210303112439093](210303.assets/image-20210303112439093.png)

<br>

- DNS 주소로 접속
  - 하지만 아까와 메시지가 다름 -> **연결은 되었으나 페이지가 작동하지 않는다**

![image-20210303112535910](210303.assets/image-20210303112535910.png)

<br>

로드 밸런서의 인스턴스의 상태 점검 -> OutOfService

![image-20210303112750466](210303.assets/image-20210303112750466.png)

<br>

로드밸런서의 상태검사 (health check) 설정 확인
- Ping 대상이 8000번으로 설정되어 있으나, 인스턴스가 작동하는 포트는 80번임

![image-20210303113009219](210303.assets/image-20210303113009219.png)

<br>

이를 80번 포트로 변경한 후 health check가 제대로 동작하는지 재확인

![image-20210303113219910](210303.assets/image-20210303113219910.png)

<br>

![image-20210303113233702](210303.assets/image-20210303113233702.png)

<br>

다시 로드 밸런서의 DNS 주소로 접속할 때 이번에는 정상 접속됨을 확인

![image-20210303113309028](210303.assets/image-20210303113309028.png)

<br>

# **Creating an Amazon Aurora RDS Database (MySQL Compatible)**

### Lab 구성도

![img](210303.assets/lab_diagram_RDS Network Diagram (1).png)

### Configure the Security Groups, Route Tables, and NACL

### 1. Verify Subnet Route Tables and NACL

서브넷 > Public Subnet1 서브넷의 라우팅 테이블 확인 -> **인터넷 게이트웨이 연결됨 확인**

![image-20210303131942645](210303.assets/image-20210303131942645.png)

Public Subnet1의 NACL 확인 -> **모든 트래픽 허용** (인바운드, 아웃바운드 규칙)

![image-20210303132056722](210303.assets/image-20210303132056722.png)

<br>

Private Subnet2 서브넷의 라우팅 테이블 확인 > **인터넷 게이트웨이 연결되지 않음 (local만 돌게끔 설정됨)**

![image-20210303132204922](210303.assets/image-20210303132204922.png)

Private Subnet2의 NACL 확인 -> **모든 트래픽 허용** (인바운드, 아웃바운드 규칙)

![image-20210303132240644](210303.assets/image-20210303132240644.png)

Private Subnet3 서브넷의 라우팅 테이블과 NACL도 확인 (Private Subnet2와 동일함)

<br>

### 2. Configure Security Groups

Default 보안 그룹의 인바운드 규칙 편집 (안해도 관계 없음)
- SSH 허용하도록 추가 

![image-20210303132416846](210303.assets/image-20210303132416846.png)

<br>

보안 그룹 생성
- 이름 : Database
- 설명 : Database security group
- VPC : LinuxAcademy
- 인바운드 규칙 추가 : 유형 - MYSQL/Aurora , 소스 - 보안그룹 (Public subnet의 보안그룹에서 들어올 수 있게끔)


![image-20210303132532856](210303.assets/image-20210303132532856.png)

<br>

현재 2개의 보안그룹 확인 가능

![image-20210303135714584](210303.assets/image-20210303135714584.png)

<br>

### Set Up an EC2 Instance for SSH Tunneling

EC2 인스턴스 생성

- 서브넷 : Public Subnet1
- 퍼블릭 IP 자동 할당 : 활성화

![image-20210303132638995](210303.assets/image-20210303132638995.png)

<br>

보안 그룹 구성 -> 기존 보안 그룹 선택 -> default 선택

![image-20210303132721998](210303.assets/image-20210303132721998.png)

이렇게하면 Public Bastion Subnet의 보안 그룹 설정 완료!

<br>

### Create an RDS Aurora Database

### 1. Create Database Subnet Group

RDS > 서브넷 그룹 > DB 서브넷 그룹 생성
- 이름 : Aurora Subnet Group
- 설명 : Aurora Subnet Group

![image-20210303132958146](210303.assets/image-20210303132958146.png)

서브넷 추가 영역 
- 가용 영역 : 모든 가용 영역 선택
- 서브넷 : 퍼블릭 서브넷 제외 모든 서브넷 선택


![image-20210303133224055](210303.assets/image-20210303133224055.png)

### 2. Create Database

데이터베이스 생성 
- 엔진 옵션 : Amazon Aurora
- 에디션 : MYSQL과 호환되는 Amazon Aurora

![image-20210303133414474](210303.assets/image-20210303133414474.png)

설정 영역 
- DB 클러스터 식별자 : AuroraInstance
- 마스터 사용자 이름 : Admin
- 마스터 암호 : 본인이 알아서 설정

![image-20210303133606693](210303.assets/image-20210303133606693.png)

DB 인스턴스 크기 영역
- 버스터블 클래스 선택 후 db.t2.small 선택 (성능과 비용 문제)

![image-20210303133704694](210303.assets/image-20210303133704694.png)

**퍼블릭 액세스 가능 여부 : 아니요 & VPC 보안 그룹 : Database 만 설정 (보안 그룹 설정 매우 중요!!!)**

- 외부에서의 DB로의 직접적인 접근을 막기 위한 것이므로 '아니요' 선택

![image-20210303133902116](210303.assets/image-20210303133902116.png)

초기 데이터베이스 이름 설정 후 Enhanced 모니터링 활성화 체크 해제 -> DB 생성 버튼 클릭

![image-20210303134047493](210303.assets/image-20210303134047493.png)

생성된 데이터베이스 확인 후 '**쓰기**' 유형의 엔드포인트 이름 알아두기

![image-20210303140726355](210303.assets/image-20210303140726355.png)

<br>

### 3. Verify Connectivity Using MySQL Workbench

> MYSQL Workbench외에도 DBeaver 사용 가능 <br>
> **DBeaver** : 다양한 DB 연결을 지원하는 툴 -> JDBC 드라이버를 이용해 데이터베이스와 통신

Workbench로 데이터베이스 연결 확인 

- Connection Name: Aurora
- Connection Method : Standard TCP/IP over SSH
- SSH Hostname: EC2 인스턴스의 Public IP
- SSH Username : ec2-user
- SSH Key File : EC2 인스턴스 생성 시 생성한 키 페어
- MYSQL Hostname : 데이터베이스 '쓰기' 유형의 엔드포인트 이름
- Username: Admin (데이터베이스 생성 시 입력했던 Username)
- Password : (데이터베이스 생성 시 입력했던 Password)

![image-20210303141417496](210303.assets/image-20210303141417496.png)

<br>

`show databases;` 쿼리 통해 데이터베이스 목록 확인

![image-20210303141340518](210303.assets/image-20210303141340518.png)

<br>

# AWS DynamoDB in the Console - Creating Tables, Items, and Indexes

DynamoDB : 완벽하게 관리되는 NoSQL 데이터베이스 서비스 (성능 및 가용성 보장을 위해 사용자가 해야 할 일이 없음, 운영에 오버헤드가 발생하지 않음)
- 배포가 단순하고 신속 > DB 설계하고 서비스하는데까지 몇 번의 클릭만 있으면 됨
- 확장이 단순하고 신속
- 데이터가 자동으로 백업 > 데이터 손실률 낮추기 위해
- 빠르고 일관된 응답 시간
- 보조 인덱스를 통한 빠른 조회
- 사용한 만큼 지불

핵심 구성요소 (Core components)

- table : 아이템의 집합
- items : 항목 - 속성들이 모여 하나의 항목이 됨
- attributes : 속성

Ex. Person 테이블 
- 속성 : PersonID, LastName, FirstName, Address
- 항목 : 속성들의 모음 (아래 그림에서 박스 하나)
- PersonID가 102인 항목은 Address 속성으로 끝나고, PersonID가 103인 항목은 FavoriteColor 속성으로 끝남 > **기본키를 제외하고 스키마가 없다**
  - 기본키 : 테이블의 각 항목을 나타내는 고유한 식별자 = 동일한 키를 가질 수 없음
    - 단순 기본키 : 파티션 키 하나로 구성
      - 일치 방식의 검색만 가능
    - 복합 기본키 : 파티션 키와 정렬 키로 구성
      - 일치, 부등호, 포함 등의 범위를 지정한 검색 지원
- 대부분의 속성은 스칼라로 문자열 또는 숫자 같이 하나의 값만 가질 수 있으며, Address 처럼 내포 속성 가질 수 있음 (DynamoDB는 32depth까지 지원)

![image-20210303151140043](210303.assets/image-20210303151140043.png)

- 첫번째 항목과 두번째 항목의 파티션 키(Artist)가 동일하므로, 항목을 구분할 수 있도록 SongTitle이라는 정렬키 추가

![image-20210303153551148](210303.assets/image-20210303153551148.png)

보조 인덱스
- 테이블에서는 하나 이상의 보조 인덱스 생성 가능
- 보조 인덱스 사용 시 기본 키에 대한 쿼리 외에 대체 키를 사용해서 테이블의 데이터를 쿼리할 수 있음
- DynamoDB에서는 각 테이블에 20개의 Global Secondary Index와 5개의 Local Secondary Index 제공

Global Secondary Index
- 파티션 키 및 정렬 키가 테이블의 파티션 키 및 정렬 키와 다를 수 있는 인덱스

Local Secondary Index
- 테이블과 파티션 키는 동일하지만 정렬 키는 다른 인덱스

<br>

### Lab 구성도

![lab_diagram_DynamoDB Data Loading](https://user-images.githubusercontent.com/77096463/110243773-27408700-7f9f-11eb-9446-bcddaa99ad4b.png)

<br>

### Create a DynamoDB Table

![image-20210303160845104](210303.assets/image-20210303160845104.png)

<br>

![image-20210303160912255](210303.assets/image-20210303160912255.png)

### Add Items to Your Table

![image-20210303161153043](210303.assets/image-20210303161153043.png)

<br>

![image-20210303161303546](210303.assets/image-20210303161303546.png)

<br>

### Add a Global Secondary Index to your Table

![image-20210303161427512](210303.assets/image-20210303161427512.png)

<br>

![image-20210303161601463](210303.assets/image-20210303161601463.png)

<br>

### Query a DynamoDB Table

![image-20210303161726448](210303.assets/image-20210303161726448.png)

<br>

![image-20210303161851770](210303.assets/image-20210303161851770.png)

<br>
