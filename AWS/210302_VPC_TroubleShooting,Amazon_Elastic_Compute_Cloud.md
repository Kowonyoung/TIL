# Troubleshooting AWS Network Connectivity: Security Groups and NACLs

### Lab 구성도

![img](210302.assets/lab_diagram_TransitExpress_Labs - NetworkTroubleshooting_Scenario1 (1).png)

<br>

### 현재 상황

Instance3가 동작하지 않음<br>
ping 명령어로 패킷 응답을 확인해보아도 '요청 시간 만료' 출력

![image-20210302100001973](210302.assets/image-20210302100001973.png)

![image-20210302111636807](210302.assets/image-20210302111636807.png)

<br>

### 1. EC2 > 인스턴스 살피기

Instance3는 private IP 주소만 부여되어 있고, public IP주소는 부여되지 않았다.<br>
public IP주소가 필요하므로 작업 > 네트워킹 > ip 주소관리 탭으로 이동하여 새로운 IP를 할당하려고 하면 private IP주소만 부여된다.<br>

![image-20210302100310045](210302.assets/image-20210302100310045.png)

<br>

그러므로 **EIP**를 할당하여 Instance3 와 연결해준다. (실제로 EIP 할당 시 과금되기 시작하므로 주의하기!)

![image-20210302100652074](210302.assets/image-20210302100652074.png)

하지만 여전히 ping 명령어로 패킷이 응답하지 않는 것을 확인할 수 있다.

### 2. 보안 그룹 살피기

Instance3가 속한 보안그룹 속성을 확인하면 인바운드 규칙은 SSH와 모든 ICMP 트래픽이 허용되어 있으며, 아웃바운드 규칙으로 모든 트래픽이 허용되어 있다. 즉, Instance3가 인터넷과 통신하는데 있어 보안 그룹은 문제될 게 없음을 나타낸다.

![image-20210302100839542](210302.assets/image-20210302100839542.png)

![image-20210302100909565](210302.assets/image-20210302100909565.png)

### 3. Subnet 구성 살피기

Instance3 인스턴스의 private IP주소는 10.3.1.147이며 이는 서브넷 CIDR 블록으로 10.3.1.0/24이다. 이를 통해 서브넷을 찾을 수도 있으며 혹은 서브넷 ID (PublicSubnet4) 를 통해 Instance3 인스턴스에 해당하는 서브넷을 찾을 수도 있다.  

![image-20210302101227509](210302.assets/image-20210302101227509.png)

### 4. NACL 살피기

NACL을 살피면 인바운드, 아웃바운드 규칙에 모든 트래픽 허용이 **Deny**되어 있음을 확인할 수 있다. <br>
트래픽을 허용하게끔 (1) NACL을 편집하거나, (2) 새로운 NACL을 만들거나 (3) 다른 NACL을 사용하게끔 변경할 수 있는데 여기서는 다른 NACL로 변경할 것이다. <br>
네트워크 ACL 연결 편집 버튼을 클릭하여 SSH와 모든 ICMP 트래픽을 허용하는 NACL을 찾아 변경한다.

![image-20210302103129601](210302.assets/image-20210302103129601.png)

![image-20210302103140821](210302.assets/image-20210302103140821.png)

하지만 여전히 ping 명령어로 패킷이 응답하지 않는 것을 확인할 수 있다.

### 5. Route Table 살피기

PublicSubnet4의 라우팅 테이블을 확인하면 local network만 설정되어 있고 igw와의 연결이 되어 있지 않음을 발견 가능하다. 이 테이블에 라우팅을 추가하거나 혹은 올바른 경로를 포함하는 다른 라우팅 테이블을 추가해서 연결해야 한다.<br>

![image-20210302103653779](210302.assets/image-20210302103653779.png)

<br>

따라서 라우팅 테이블 연결 편집 버튼을 클릭하여 외부와 연결되어 있는 라우팅 테이블을 선택한다. 

![image-20210302103850459](210302.assets/image-20210302103850459.png)

<br>

ping 명령어를 실행하면 패킷의 응답을 확인 가능하며 ssh 접속도 가능하다.

![image-20210302104021428](210302.assets/image-20210302104021428.png)

![image-20210302105328697](210302.assets/image-20210302105328697.png)

<br>

# Create a Windows EC2 Instance and Connect using Remote Desktop Protocol (RDP)

![image-20210302113326581](210302.assets/image-20210302113326581.png)

Linux 기반의 인스턴스의 경우에는 SSH를 이용해서 연결
Windows EC2 인스턴스의 경우에는 일반적으로 RDP(원격 데스크탑 프로토콜)를 사용해서 연결
- RDP : Windows 컴퓨터를 관리하기 위한 GUI

**공통적으로** 
**1) 퍼블릭 IP 주소** 
**2) 외부에서 접속할 수 있도록 서비스 포트를 개방해줘야 함 (보안그룹, NACL)**
**3) 인터넷 게이트웨이로 라우팅 설정**
**4) 개인키 파일을 보관해야 함**

<br>

### VPC 서비스 확인

VPC와 서브넷 확인

![image-20210302113733375](210302.assets/image-20210302113733375.png)

![image-20210302113752724](210302.assets/image-20210302113752724.png)

<br>

IGW 확인

![image-20210302113817850](210302.assets/image-20210302113817850.png)

<br>

NACL 확인 -> RDP 접속을 허용

![image-20210302113848760](210302.assets/image-20210302113848760.png)

![image-20210302113910570](210302.assets/image-20210302113910570.png)

<br>

라우팅 테이블 -> IGW와 연결을 확인

![image-20210302113939567](210302.assets/image-20210302113939567.png)

<br>

보안 그룹 -> 모든 접속을 허용

![image-20210302113959618](210302.assets/image-20210302113959618.png)

![image-20210302114010427](210302.assets/image-20210302114010427.png)

<br>

### RDP 트래픽만 허용하는 보안 그룹 생성

현재 하나 있는 보안 그룹은 모든 트래픽을 허용하고 있으므로 보안 상 RDP 트래픽만 허용하기 위해 별도의 보안 그룹을 생성한다.

![image-20210302114357444](210302.assets/image-20210302114357444.png)

<br>

### EC2 인스턴스 생성 후 RDP 연결

1. 인스턴스 > 인스턴스 시작 클릭

- AMI : Microsoft Windows Server 2019 Base 
- 인스턴스 유형 : Default
- 인스턴스 구성 : 퍼블릭 IP 자동 할당만 활성화
- 태그 추가 : 키 (Name),  값 (WinRDP)
- 보안 그룹 구성 : 기존 보안 그룹 선택 -> EssentialSG 선택

2. 새로운 키 페어 생성 > windowsrdp 이름 부여 후 키 페어 다운로드 받기 

3. 실행 중인 인스턴스 연결 클릭 > RDP 클라이언트 탭에서 '원격 데스크톱 파일 다운로드', '암호 가져오기'

![image-20210302115544684](210302.assets/image-20210302115544684.png)

4. windowsrdp 키 페어 파일 가져오기 > 암호 해독 클릭

![image-20210302115648357](210302.assets/image-20210302115648357.png)

5. 해독된 암호를 복사 
6. 다운받은 원격 데스크톱 파일 (WinRDP.rdp)에 해독된 암호를 붙여넣고 원격 접속하기

![image-20210302115822278](210302.assets/image-20210302115822278.png)

![image-20210302115903233](210302.assets/image-20210302115903233.png)

<br>

# **Creating Your Own EC2 Workstation in the AWS Console**

Workstation: 서버와 PC의 중간 레벨로 **작업을 위한 EC2 인스턴스** <br>
AWS CLI로 AWS 인스턴스를 생성하고 관리할 수 있는 권한을 가진 인스턴스

<br>

### Lab 구성도

![image-20210302144111421](210302.assets/image-20210302144111421.png)

<br>

### EC2 인스턴스 생성 후 해당 인스턴스를 통해 S3 버킷 생성, SNS 문자 메시지 전송

이 때 EC2 인스턴스를 워크스테이션이라 한다. 

1. 인스턴스 생성 

- AMI : Amazon Linux 2 AMI 
- 인스턴스 유형 : Default
- IAM 역할 : EC2InstanceCDALabRole -> EC2 인스턴스에서 AWS 내의 다양한 작업을 수행할 수 있는 권한 부여
- 키 페어 생성 : cdaec2lab -> 키 페어 다운로드 후 인스턴스 시작

![image-20210302142606190](210302.assets/image-20210302142606190.png)

<br>

2. SSH 클라이언트를 이용해서 ec2 인스턴스 접속

- 부여받은 키 페어 등록
- Host : Public IP 등록
- Username : ec2-user

![image-20210302142952999](210302.assets/image-20210302142952999.png)

3. DynamoDB에 테이블 목록을 조회

```
[ec2-user@ip-10-0-0-191 ~]$ aws dynamodb list-tables
{
    "TableNames": []
}
```

4. aws config 명령어로 기본 지역 설정
```
[ec2-user@ip-10-0-0-191 ~]$ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [us-east-1]: us-east-1
Default output format [None]:
```
5. S3 버킷 목록 생성 및 조회
```
[ec2-user@ip-10-0-0-191 ~]$ aws s3 mb s3://haerim
make_bucket: haerim

[ec2-user@ip-10-0-0-191 ~]$ aws s3 ls
2021-03-02 05:10:35 haerim
```

6. boto3 (Python API) SDK를 이용한 SNS 서비스

```
[ec2-user@ip-10-0-0-191 ~]$ curl -O https://bootstrap.pypa.io/2.7/get-pip.py
[ec2-user@ip-10-0-0-191 ~]$ python get-pip.py --user	
```

```
[ec2-user@ip-10-0-0-191 ~]$ sudo pip install boto3
// 중간에 안돼서 sudo easy-install boto3 명령어로 설치함
```

```
[ec2-user@ip-10-0-0-191 ~]$ python
Python 2.7.18 (default, Feb 18 2021, 06:07:59)
[GCC 7.3.1 20180712 (Red Hat 7.3.1-12)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import boto3
>>> sns = boto3.client('sns')
```

```
>>> phone_number= '+8201012345678'
>>> sns.publish(Message='메롱', PhoneNumber=phone_number)
{'ResponseMetadata': {'RetryAttempts': 0, 'HTTPStatusCode': 200, 'RequestId': '0b0c2a35-871b-5ac6-85f7-728257e1dc0c', 'HTTPHeaders': {'x-amzn-requestid': '0b0c2a35-871b-5ac6-85f7-728257e1dc0c', 'date': 'Tue, 02 Mar 2021 05:35:53 GMT', 'content-length': '294', 'content-type': 'text/xml'}}, u'MessageId': '5a38659f-d6ea-51d1-94d8-3d280610d71b'}
```

<br>

# Create a Custom AMI in AWS

Auto Scaling 환경과 같이 동일한 상태의 애플리케이션을 즉시 사용해야 하는 경우 -> 사용자 지정 AMI를 생성해서 사전 구성된 인스턴스를 시작하고 대기를 건너 뛸 수 있음

### Lab 구성도

1) 기존 AWS Linux AMI 로부터 인스턴스 생성
2) SSH 접속을 통해 인스턴스 연결하고 Apache와 PHP 수동으로 구성
3) Custom AMI 생성
4) Custom AMI로부터 새로운 인스턴스 생성하고 Apache와 PHP가 설치되어 있는지 확인

![image-20210302160124100](210302.assets/image-20210302160124100.png)

<br>

### Install Apache and PHP

SSH 접속

![image-20210302153340326](210302.assets/image-20210302153340326.png)

<br>

패키지 모두 업데이트하기

```
[ec2-user@ip-10-0-0-46 ~]$ sudo yum update -y
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
No packages marked for update
```
Apache and PHP 설치
```
[ec2-user@ip-10-0-0-46 ~]$ sudo yum install -y httpd php
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.46-1.amzn2 will be installed
```

Apache 서비스 시작
```
[ec2-user@ip-10-0-0-46 ~]$ sudo service httpd start
Redirecting to /bin/systemctl start httpd.service
```
부팅 시 자동 시작되게끔 설정
```
[ec2-user@ip-10-0-0-46 ~]$ sudo chkconfig httpd on
Note: Forwarding request to 'systemctl enable httpd.service'.
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

이렇게 설정을 마친 후 Apache 웹 페이지가 잘 구동되는지 확인 (Public IP 주소를 입력)

![image-20210302153711350](210302.assets/image-20210302153711350.png)

<br>

ec2-user를 Apache Group에 추가

```
[ec2-user@ip-10-0-0-46 ~]$ sudo usermod -a -G apache ec2-user
```
/var/www 디렉터리의 권한 변경 후 var/www/html 디렉터리에 PHP 페이지 생성

- phpinfo.php에 php의 정보를 출력

```
[ec2-user@ip-10-0-0-46 ~]$ sudo chown -R ec2-user:apache /var/www
[ec2-user@ip-10-0-0-46 ~]$ echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
```

위 명령어 입력 후 아까 구동한 Apache 웹페이지 주소에 /phpinfo.php를 추가하여 접속 -> http://18.235.1.33/phpinfo.php

![image-20210302154120258](210302.assets/image-20210302154120258.png)

<br>

이 서버와 동일한 인스턴스를 생성할 때 위와 같은 작업을 반복하면 오래 걸리고 인스턴스 대기 시간이 길어진다 --> **시간을 줄이기 위해 현재 인스턴스의 상태를 사용자 정의 이미지로 만들기 (= Custom AMI)**

### Create a Custom AMI

web-config 인스턴스 선택 > 작업 > 이미지 및 템플릿 > 이미지 생성 

- 이름 : WebPHP7
- 설명 : (임의) Web server with Apache and PHP7

![image-20210302154240593](210302.assets/image-20210302154240593.png)

<br>

이미지 생성 설정 후 AMI 탭으로 이동하여 이미지 생성 과정 지켜보기 (pending -> available)

![image-20210302154348592](210302.assets/image-20210302154348592.png)

<br>

![image-20210302155654616](210302.assets/image-20210302155654616.png)

AMI가 생성되었다면 새로이 인스턴스를 생성 > 이 때 나의 AMI 탭에서 방금 생성한 AMI를 선택

- 모든 과정은 default로 세팅
- 태그 - Key : Name / Value : web-php
- 보안 그룹 : 기존의 Web Security Group 선택
- 키 페어 : 기존의 WebConfig 키 페어 선택

![image-20210302154711390](210302.assets/image-20210302154711390.png)

<br>

생성된 web-php 인스턴스의 'public ip주소/phpinfo.php' 접속 확인

![image-20210302155148058](210302.assets/image-20210302155148058.png)

<br>

![image-20210302154924277](210302.assets/image-20210302154924277.png)

<br>

# Resizing Root AWS EBS Volumes to Increase Performance

루트 볼륨의 크기를 조정 <- IOPS (Input, Output Operations Per Second) 향상을 위해 <br>
1) 독립형 인스턴스 (bastion host)<br>
2) Auto Scaling Group (2개의 웹 서버 인스턴스)

### Lab 구성도

![image-20210302171021956](210302.assets/image-20210302171021956.png)

<br>

### Create an EBS Snapshot - Bastion host

EC2에서 실행 중인 3개의 인스턴스를 확인 후, bastion-host 선택<br>
- 스토리지 > 볼륨 ID 선택

![image-20210302171649686](210302.assets/image-20210302171649686.png)

<br>

Auto Scaling 그룹 확인
- 최소 용량, 최대 용량 : 2 -> 인스턴스 2개 지정 조건
- 시작 구성 : 최소용량, 최대용량과 같은 조건을 만족시키지 못하는 경우 새로운 인스턴스를 생성할 때의 설정

![image-20210303092507783](210302_TroubleShooting,RDP,WorkStation,EBS.assets/image-20210303092507783.png)

<br>

작업 > 스냅샷 생성 > 설명 : BastionSnap 으로 지정한 뒤 스냅샹 생성 버튼 클릭 

![image-20210302171634345](210302.assets/image-20210302171634345.png)

<br>

생성한 스냅샷 확인

![image-20210302171821096](210302.assets/image-20210302171821096.png)

<br>

### Create a new (larger) EBS Volume - Bastion host

생성한 스냅샷 선택 > 작업 > 볼륨 생성
- 크기 : 40 Gib -> IOPS : 120/3000 자동으로 변경됨
- 가용 영역 : us-east-1a

![image-20210302172018507](210302.assets/image-20210302172018507.png)

<br>

생성한 볼륨 확인

![image-20210302172116230](210302.assets/image-20210302172116230.png)

<br>

### Attach the (larger) EBS volume to an EC2 instance - Bastion host

인스턴스에 부여 되었던 기존의 볼륨을 방금 생성한 볼륨으로 변경할 예정<br>
우선 bastion-host 인스턴스의 상태를 '**중지**' 로 변경

![image-20210302172342576](210302.assets/image-20210302172342576.png)

<br>

볼륨 > bastion-host와 연결되어 있는 볼륨 선택 > 작업 > 볼륨 분리

![image-20210302172528193](210302.assets/image-20210302172528193.png)

<br>

위에서 생성한 40 Gib 볼륨을 선택 > 작업 > 볼륨 연결
- 인스턴스 :  bastion-host 선택
- 디바이스 : /dev/xvda

![image-20210302173049255](210302.assets/image-20210302173049255.png)

<br>

인스턴스 > bastion-host 인스턴스 선택 > 인스턴스 시작

![image-20210302173222369](210302.assets/image-20210302173222369.png)

<br>

SSH 접속 및 블록 디바이스 목록 확인

```
[cloud_user@ip-10-99-1-117 ~]$ ssh cloud_user@52.206.34.121
[cloud_user@ip-10-99-1-117 ~]$ lsblk
```

![image-20210302173657100](210302.assets/image-20210302173657100.png)

<br>

### Create a new Auto-scaling Launch Configuration and Update the existing Auto-scaling Group

**처음 auto scaling 그룹 시작 구성과 동일한 조건으로 시작 구성을 생성하되 스토리지 볼륨만 변경할 것**

시작 구성 > 사용자 데이터 내용 복사해두기

```
#!/bin/bash
yum update -y
yum install -y httpd24 php70 mysql56-server php70-mysqlnd git
cd /var/www/html
git clone https://github.com/linuxacademy/content-aws-sysops-administrator.git
cd content-aws-sysops-administrator/wp-site/
mv * /var/www/html
groupadd www
usermod -a -G www ec2-user
chown -R root:www /var/www
chmod -R 2775 /var/www
echo '<?php phpinfo(); ?>' > /var/www/html/phpinfo.php
service httpd start
chkconfig httpd on
service mysqld start
chkconfig mysqld on
```

시작 구성 생성 선택
- 이름 : newLC
- AMI : ami-07e677d445d41baeb
- 인스턴스 유형 : t2.micro


![image-20210303093158113](210302_TroubleShooting,RDP,WorkStation,EBS.assets/image-20210303093158113.png)

고급 세부 정보 > 사용자 데이터 > 복사해둔 사용자 데이터 내용 붙여넣기 (**기존과 동일한 인스턴스를 생성할 수 있도록 설정**)
- IP 주소 유형 : '어느 인스턴스에도 퍼블릭 ip 주소를 할당하지 않습니다' 선택

![image-20210302174338504](210302.assets/image-20210302174338504.png)

스토리지 볼륨 > 새 볼륨 추가 
- 사이즈 : 40 Gib


![image-20210302174354400](210302.assets/image-20210302174354400.png)

보안 그룹 : 기존 보안 그룹 선택 -> WebServerSecurityGroup

![image-20210302174405482](210302.assets/image-20210302174405482.png)

키 페어 옵션 : 키 페어 없이 계속

![image-20210302174433983](210302.assets/image-20210302174433983.png)

Auto Scaling Group에 새롭게 생성한 시작 구성으로 변경

- Auto Scaling Group 그룹 > 우측 편집 버튼 선택 > 시작 구성 변경 (newLC로)

![image-20210303093337899](210302_TroubleShooting,RDP,WorkStation,EBS.assets/image-20210303093337899.png)

기존에 동작하고 있던 인스턴스를 종료 → Auto Scaling 설정에 따라 (시작 구성에 설정된) 새로운 인스턴스가 생성 

![image-20210303093603371](210302_TroubleShooting,RDP,WorkStation,EBS.assets/image-20210303093603371.png)

<u>새로운 인스턴스 생성 확인</u>
