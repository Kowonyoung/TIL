# Deployment (배포)

**빅뱅 배포**<br>
애플리케이션의 전체 또는 대부분을 한 번에 업데이트

**롤링 배포 (단계적 배포)**<br>
애플리케이션의 이전 버전 (파란색)을 점차적으로 새 버전(초록색) 으로 교체

![image](https://user-images.githubusercontent.com/77096463/110263563-77990280-7ffa-11eb-9025-1e6f0323f576.png)

<br>

**Blue-Green, Red-Black, A/B 배포** <br>
두 개의 동일한 프로덕션 환경이 병렬로 작동<br>
하나는 모든 트래픽을 수신하는 실행 환경, 다른 하나는 유휴 상태<br>

![image](https://user-images.githubusercontent.com/77096463/110263703-d494b880-7ffa-11eb-8b2f-2b26c26a5c71.png)

새로운 버전의 애플리케이션은 Green 환경에 배포되고 기능 및 성능 테스트를 수행<br>
테스트 결과가 성공이면 애플리케이션의 트래픽을 파란색에서 초록색으로 라우팅

![image](https://user-images.githubusercontent.com/77096463/110263826-2ccbba80-7ffb-11eb-908a-b378fde6feb8.png)

초록색이 활성화된 후 문제가 발생하면 트래픽을 다시 파란색으로 라우팅 <br>
Blue-Green 배포에서는 두 시스템 모두가 동일한 지속 계층 또는 데이터베이스 백엔드를 사용해야 함

**Canary (카나리아) 배포**<br>
프로덕션 인프라의 작은 부분에 새 애플리케이션 코드를 배포해서 소수의 사용자만 해당 애플리케이션으로 라우팅<br>
보고된 오류가 없는 새 버전은 나머지 인프라에 점차적으로 롤 아웃

![image](https://user-images.githubusercontent.com/77096463/110264178-02c6c800-7ffc-11eb-935b-9eba1ae22896.png)

<br>

**Amazon API Gateway**<br>
개발자가 규모와 관계 없이 API를 쉽게 생성, 게시, 유지관리, 모니터링, 보안 유지를 할 수 있도록 하는 완전 관리형 서비스<br>
API : 애플리케이션이 백엔드 서비스의 데이터, 비즈니스 로직 또는 기능에 액세스할 수 있도록 해주는 역할<br>
API 유형 : RESTful API, WebSocket API
![image](https://user-images.githubusercontent.com/77096463/110264661-2b02f680-7ffd-11eb-9831-fc01545ad7a9.png)

<br>

-----------

# **API Gateway Canary Release Deployment**

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/110264762-66052a00-7ffd-11eb-9d4f-a931420f1821.png)

<br>

IAM 역할 확인 -> CloudWatch Logs와 DynamoDB 권한 가짐

![image](https://user-images.githubusercontent.com/77096463/110265022-f5aad880-7ffd-11eb-8f61-f4cb47dd7204.png)

<br>

### Create First Lambda Function

첫 번째 람다 함수 생성
- 함수 이름 : mathCeil
- 실행 역할 : 기존 역할 사용 -> cfst~ 역할 선택

![image](https://user-images.githubusercontent.com/77096463/110265120-30ad0c00-7ffe-11eb-93fa-862ee2f36937.png)

<br>

아래의 코드를 index.js에 작성하기

```js
console.log('Loading Lambda function');

exports.handler = async (event, context, callback) => {
    let resultNum = Math.ceil(999.99); //소수점 이하 올림

    callback(null, 'this is the original function (Math.ceil) = ' + resultNum);
};
```

![image](https://user-images.githubusercontent.com/77096463/110265329-a44f1900-7ffe-11eb-8ee2-a6ad9eb938e3.png)

### Create Second Lambda Function

두 번째 람다 함수 생성

- 함수 이름 : mathFloor
- 실행 역할 : 기존 역할 사용 -> cfst~ 역할 선택

![image](https://user-images.githubusercontent.com/77096463/110265434-e5dfc400-7ffe-11eb-8835-858ab746794e.png)

<br>

아래의 코드를 index.js에 작성하기

```js
console.log('Loading Lambda function');

exports.handler = async (event, context, callback) => {
    let resultNum = Math.floor(999.99);	//소수점 이하 내림

    callback(null, 'this is the canary function (Math.floor) = ' + resultNum);
};
```

![image](https://user-images.githubusercontent.com/77096463/110265483-03149280-7fff-11eb-9c67-d39c5f454aa4.png)

<br>

### Create and Deploy Our API Within API Gateway

### Create the API

> 외부에서 람다 함수 호출할 수 있도록 API 생성<br>

API Gateway > REST API > 구축<br>
- 새 API
- API 이름 : devopsCanary
- 설명 : API Gateway Canary Testing
- 엔드포인트 유형 : 지역

![image](https://user-images.githubusercontent.com/77096463/110265624-571f7700-7fff-11eb-82b1-698ec82eb889.png)

<br>

리소스 탭 작업 > 메서드 생성 > / > GET 선택 <br>
/-GET- 설정 > Lambda 함수 -> mathCeli 선택

![image](https://user-images.githubusercontent.com/77096463/110265786-b7aeb400-7fff-11eb-882e-46273e167e4a.png)

<br>

API 동작 방식 확인

![image](https://user-images.githubusercontent.com/77096463/110265889-f6dd0500-7fff-11eb-8bde-b19d7d1d98ec.png)

<br>

API 테스트

![image](https://user-images.githubusercontent.com/77096463/110266154-94d0cf80-8000-11eb-8210-a500d2a0ff5b.png)

<br>

### Deploy the First Function

리소스 탭 작업 > API 배포 

![image](https://user-images.githubusercontent.com/77096463/110266391-09a40980-8001-11eb-8d47-253220fbc11c.png)

<br>

test 스테이지 편집기에서 URL 주소를 클릭했을 때 새로고침을 여러번 해도 아까 API 테스트했던 결과와 동일한 결과가 나오는 것을 확인한다. (API 게이트웨이에 연결되어 있는 람다 함수가 MathCeil 하나이므로)

![image](https://user-images.githubusercontent.com/77096463/110266479-3821e480-8001-11eb-8286-a1cfdfe8b78b.png)

<br>

Canary 탭 > Canary로 지정된 요청 백분율을 10%로 변경 
- 브라우저로 API 게이트웨이 엔드포인트 호출해도 변화가 없음을 확인 -> 새로운 버전이 등록되지 않았기 때문


![image](https://user-images.githubusercontent.com/77096463/110267586-7d471600-8003-11eb-949d-8f25565fb621.png)

<br>

람다 함수를 mathFloor로 변경

![image](https://user-images.githubusercontent.com/77096463/110267749-c4cda200-8003-11eb-8e17-1e77d59d5092.png)

<br>

테스트를 여러번 수행해도 동일한 결과 (999) 를 반환하는 것을 확인

![image](https://user-images.githubusercontent.com/77096463/110267792-dd3dbc80-8003-11eb-96fb-b659b10a76e7.png)

<br>

### Deploy the Second Function

API 배포 -> 배포 스테이지 : test (Canary 사용)

![image](https://user-images.githubusercontent.com/77096463/110267869-01999900-8004-11eb-99ea-7da6822fbad4.png)

<br>

주소는 동일한데 1:9의 비율로 999, 1000 값이 번갈아가며 나옴

![image](https://user-images.githubusercontent.com/77096463/110267939-24c44880-8004-11eb-856e-89aed741bd9c.png)

<br>![image](https://user-images.githubusercontent.com/77096463/110267954-2a219300-8004-11eb-8183-5bfe3e73d3ff.png)

<br>

Canary로 지정된 요청 비율을 50%로 조정

![image](https://user-images.githubusercontent.com/77096463/110268094-766cd300-8004-11eb-9045-0a0755cd4c75.png)

<br>

동일한 URL에서 새 버전과 이전 버전이 5:5 비율로 제공됨을 확인할 수 있다.

### Promote the Canary

Canary 탭 > Canary 승격 

- test로 지정된 요청 백분율 : 100%로 변경됨

![image](https://user-images.githubusercontent.com/77096463/110268309-cba8e480-8004-11eb-94fc-8c0456e1f890.png)

<br>

이제 URL 접속 시 새로운 버전의 실행 결과인 999만 제공됨을 확인

![image](https://user-images.githubusercontent.com/77096463/110268418-f2ffb180-8004-11eb-89bd-50641a3dae4d.png)

----------

# AWS EC2 인스턴스 생성 (P.13~25)

### 1. 리전 확인 및 설정

리전 : us-east-1 (비용 문제 때문에)<br>
EC2 인스턴스 생성<br>

- AMI : Amazon Linux 2 AMI (**CentOS** 기반으로 AWS에 최적화되어 있는 리눅스 가상머신 이미지)
- 인스턴스 유형 : t2.micro
- 인스턴스 세부 정보 구성 : 나머지는 default, 퍼블릭 IP 자동 할당 영역만 '활성화'
- 스토리지 : default
- 태그 : 키 (Name) / 값 (exercise-instance) -> 추후 인스턴스를 용이하게 사용하기 위해 태그 부여
- 보안 그룹 이름 : ssh / 설명 : security rule for ssh access
- 새 키 페어 생성 -> **서버에 접속하는 키이기 때문에 분실해서도 안되며 유출해서도 안된다**

아래 화면처럼 생성한 인스턴스 및 세부 정보를 확인한다.<br>
**퍼블릭 IP와 도메인은 인스턴스가 꺼질 때 사라지고 켜질 때 새로 할당받기 때문에** EIP를 할당하는 게 좋다.

![image](https://user-images.githubusercontent.com/77096463/110277397-87bfda80-8018-11eb-9c82-d711c65d2619.png)

<br>

### 2. EC2 인스턴스 접속

ssh 접속을 위해 Server host에 생성한 인스턴스의 퍼블릭 IP 주소를 입력, Authentication User로 ec2-user를 입력한다. 인스턴스 생성 시 발급받았던 exercise key를 이용하여 접속한다.
- ec2-user : Amazon Linux의 시스템 사용자 계정 -> 루트 권한 제외한 대부분 작업은 여기서 진행

접속 성공 시, 아래와 같은 화면 출력

```

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
No packages needed for security; 2 packages available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-XX-XX-XXX ~]$
```

<br>

### 3. HTTP, HTTPS도 접근 가능하도록 보안 그룹 추가하기

현재 ssh 접속만 허용되어 있으므로 HTTP, HTTPS 프로토콜도 허용해주기 위해 보안 그룹을 생성한다.<br>
- 인바운드 규칙으로 HTTP, HTTPS 선택

![image](https://user-images.githubusercontent.com/77096463/110276795-4b3faf00-8017-11eb-8c18-0a25be39c0ed.png)

<br>

인스턴스 보안 그룹 변경 -> ssh, web 보안 그룹 모두 포함하도록 변경

![image](https://user-images.githubusercontent.com/77096463/110277690-24827800-8019-11eb-835b-3dd9ea47a44d.png)

<br>

### 4. 서버 환경 구성 - 실습에 필요한 프로그램 설치

nvm (node version manager) 설치 스크립트 가져와서 설치

```
[ec2-user@ip-172-XX-XX-XXX ~]$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12819  100 12819    0     0   379k      0 --:--:-- --:--:-- --:--:--  379k
=> Downloading nvm as script to '/home/ec2-user/.nvm'
...
```

nvm 설치 스크립트 실행
- `첫 번째 .` : source 의미 (환경 변수를 변경할 때 사용)  = ($ source ~/.nvm/nvm.sh)
- `두 번째 .` : 숨김 파일/디렉터리
- `세 번째 .` : 파일명과 확장자를 구분하는 구분자

```
[ec2-user@ip-172-XX-XX-XXX ~]$ . ~/.nvm/nvm.sh
```

10.13.0 버전의 Node.js 설치

```
[ec2-user@ip-172-XX-XX-XXX ~]$ nvm install 10.13.0
Downloading and installing node v10.13.0...
Downloading https://nodejs.org/dist/v10.13.0/node-v10.13.0-linux-x64.tar.xz...
############################################################################################# 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v10.13.0 (npm v6.4.1)
Creating default alias: default -> 10.13.0 (-> v10.13.0)
```

원하는 버전이 설치되었는지 확인
- `node -e "command"` : 뒤의 문자열을 js 명령어로 실행 


```
[ec2-user@ip-172-XX-XX-XXX ~]$ node -e "console.log('Running Node.js ' + process.version)"
Running Node.js v10.13.0
```

<br>

----------



# 소스코드 배포 (P.25~42)

### 1. git을 이용한 소스코드 배포

git 설치에 필요한 패키지 설치

```
[ec2-user@ip-172-XX-XX-XXX ~]$ sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
```

작업 디렉터리 생성 후 이동

```
[ec2-user@ip-172-XX-XX-XXX ~]$ cd /var
[ec2-user@ip-172-XX-XX-XXX var]$ sudo mkdir www

# www 의 경로를 ec2-user 의 소유로 변경
[ec2-user@ip-172-XX-XX-XXX var]$ sudo chown ec2-user www
[ec2-user@ip-172-XX-XX-XXX var]$ cd /var/www
[ec2-user@ip-172-XX-XX-XXX www]$
```

github로부터 소스코드 가져옴

```
[ec2-user@ip-172-XX-XX-XXX www]$ git clone https://github.com/deopard/aws-exercise-a.git
```

```
[ec2-user@ip-172-XX-XX-XXX www]$ cd aws-exercise-a/
[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ sudo yum install -y tree
```

소스코드 확인

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ ls -l
total 60
-rw-rw-r-- 1 ec2-user ec2-user   294 Mar  8 05:57 app.js
-rw-rw-r-- 1 ec2-user ec2-user 35149 Mar  8 05:57 LICENSE
-rw-rw-r-- 1 ec2-user ec2-user   539 Mar  8 05:57 package.json
-rw-rw-r-- 1 ec2-user ec2-user 13432 Mar  8 05:57 package-lock.json
drwxrwxr-x 2 ec2-user ec2-user    19 Mar  8 05:57 public

[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ cat app.js
const express = require('express');	//Node 기반의 웹서버 모듈 가져오기
const app = express();

app.get('/', (req, res) => {	// '/'로 GET 방식의 요청이 온다면
  res.send('AWS exercise의 A project입니다.');
});

app.listen(3000, () => {		//3000번 포트로 요청을 대기
  console.log('Example app listening on port 3000!');
});

app.get('/health', (req, res) => {	// '/health' GET 방식의 요청이 온다면
  res.status(200).send();			// 응답 상태 코드의 값을 200으로 설정해서 반환
});
```

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ cat package.json
{
  "name": "aws-exercise-a",
  "version": "1.0.0",
  "description": "AWS exercise project A",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/deopard/aws-exercise-a.git"
  },
  "author": "Tom Kim",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/deopard/aws-exercise-a/issues"
  },
  "homepage": "https://github.com/deopard/aws-exercise-a#readme",
  "dependencies": {
    "express": "^4.16.3"		// 의존 모듈 등록 -> express 모듈이 반드시 필요 
  }
}
```

의존 모듈 설치

- `npm` : node package manger -> node.js에서 사용하는 의존성 패키지를 쉽게 관리할 수 있도록 도와주는 프로그램 (지금 애플리케이션 실행에 필요한 외부 라이브러리를 설치해준다)

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ npm install
added 50 packages from 47 contributors and audited 50 packages in 1.633s
found 0 vulnerabilities
```

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-a]$ ls ./node_modules/
accepts              debug        finalhandler  merge-descriptors  parseurl        serve-static
array-flatten        depd         forwarded     methods            path-to-regexp  setprototypeof
body-parser          destroy      fresh         mime               proxy-addr      statuses
bytes                ee-first     http-errors   mime-db            qs              type-is
content-disposition  encodeurl    iconv-lite    mime-types         range-parser    unpipe
content-type         escape-html  inherits      ms                 raw-body        utils-merge
cookie               etag         ipaddr.js     negotiator         safe-buffer     vary
cookie-signature     express      media-typer   on-finished        send
```

<br>

------------------



### 2. 웹 서버와 웹 애플리케이션 서버로 이원화

**웹서버** : nginx -> 정적 자원 요청에 대한 응답<br>
**웹 애플리케이션 서버** : Phusion Passenger -> 응용 프로그램의 실행 결과 반환
- 단독으로 동작하는 '독립형 모드'
- 아파치 웹 서버와 연동되어 실행되는 '아파치 통합 모드'
- nginx 웹 서버와 연동되어 실행되는 **'nginx 통합 모드'**

<br>

Phusion Passenger 설치 파일 다운로드

```
[ec2-user@ip-172-XX-XX-XXX www]$ wget https://s3.amazonaws.com/phusion-passenger/releases/passenger-5.3.6.tar.gz
```

작업 디렉터리 생성 및 압축 해제
```
[ec2-user@ip-172-XX-XX-XXX www]$ sudo mkdir /var/passenger
[ec2-user@ip-172-XX-XX-XXX www]$ sudo chown ec2-user /var/passenger
[ec2-user@ip-172-XX-XX-XXX www]$ tar -xzvf passenger-5.3.6.tar.gz -C /var/passenger
```

경로 설정

```
#현재 설정된 PATH ($PATH)에다가 /var/passenger/passenger-5.3.6/bin를 추가하여 bash_profile 내용에 추가
[ec2-user@ip-172-XX-XX-XXX www]$ echo export PATH=/var/passenger/passenger-5.3.6/bin:$PATH >> ~/.bash_profile

#bash_profile 파일 반영
[ec2-user@ip-172-XX-XX-XXX www]$ source ~/.bash_profile
```

rvm (Ruby version manager) 설치 

```
[ec2-user@ip-172-XX-XX-XXX www]$ gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

# stable 버전의 rvm 설치
[ec2-user@ip-172-XX-XX-XXX www]$ curl -sSL https://get.rvm.io | bash -s stable

# 현재 터미널에서 rvm 사용할 수 있도록 설정
[ec2-user@ip-172-XX-XX-XXX www]$ source ~/.rvm/scripts/rvm
[ec2-user@ip-172-XX-XX-XXX www]$ rvm reload
RVM reloaded!

# 루비를 설치하는 데 필요한 라이브러리 등을 설치
[ec2-user@ip-172-XX-XX-XXX www]$ rvm requirements run

# 루비 설치
[ec2-user@ip-172-XX-XX-XXX www]$ rvm install 2.4.3
```

passenger Nginx module 설치
- passenger와 nginx를 이어주기 위한 모듈 설치 -> 사용할 언어로 'Node.js' 선택

```
[ec2-user@ip-172-XX-XX-XXX www]$ passenger-install-nginx-module
```

<u>가상 메모리 부족하여 설치 시 문제 발생</u> -> ctrl+c 로 종료

```
Your system does not have a lot of virtual memory		

Compiling Phusion Passenger works best when you have at least 1024 MB of virtual
memory. However your system only has 983 MB of total virtual memory (983 MB
RAM, 0 MB swap). It is recommended that you temporarily add more swap space
before proceeding. You can do it as follows:

  sudo dd if=/dev/zero of=/swap bs=1M count=1024
  sudo mkswap /swap
  sudo swapon /swap

See also https://wiki.archlinux.org/index.php/Swap for more information about
the swap file on Linux.

If you cannot activate a swap file (e.g. because you're on OpenVZ, or if you
don't have root privileges) then you should install Phusion Passenger through
DEB/RPM packages. For more information, please refer to our installation
documentation:

  https://www.phusionpassenger.com/library/install/nginx/
```

가상 메모리 증설

```
[ec2-user@ip-172-XX-XX-XXX www]$ sudo dd if=/dev/zero of=/swap bs=1M count=1024

[ec2-user@ip-172-XX-XX-XXX www]$ sudo mkswap /swap
mkswap: /swap: insecure permissions 0644, 0600 suggested.
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=c7a6f44b-b97b-4a55-b9a9-d2aa7d285b8f

[ec2-user@ip-172-XX-XX-XXX www]$ sudo swapon /swap
swapon: /swap: insecure permissions 0644, 0600 suggested.
```

passenger Nginx module 재설치 -> node.js 선택 & 1번 선택 -> <u>권한 오류 발생</u>

```
[ec2-user@ip-172-XX-XX-XXX www]$ passenger-install-nginx-module
```

원래 /opt/nginx 경로에 설치하려 하였으나 현재 ec2-user로 로그인되어 작업을 진행하기 때문에 권한 오류가 발생한다. 지금 ruby를 이용하여 설치를 진행하고 있기에 **rvmsudo**를 이용하여 설치를 진행하는 것이 권장된다. rvm에서 제공하는 기능인 rvmsudo는 sudo 권한을 실행하면서 ruby에 필요한 환경변수 값까지 보존해주는 기능을 제공한다.

권한 부여 후 rvmsudo을 이용하여 passenger Nginx module 재설치  -> 설치에 2~30분 소요

```
[ec2-user@ip-172-XX-XX-XXX www]$ export ORIG_PATH="$PATH"
[ec2-user@ip-172-XX-XX-XXX www]$ rvmsudo -E /bin/bash
[root@ip-172-XX-XX-XXX www]# export PATH="$ORIG_PATH"
[root@ip-172-XX-XX-XXX www]# export rvmsudo_secure_path=1
[root@ip-172-XX-XX-XXX www]# /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby /var/passenger/passenger-5.3.6/bin/passenger-install-nginx-module
```

```
# 설치 성공 메시지
Nginx with Passenger support was successfully installed.

# nginx 설정 파일 경로
The Nginx configuration file (/opt/nginx/conf/nginx.conf)
must contain the correct configuration options in order for Phusion Passenger
to function correctly.

This installer has already modified the configuration file for you! The
following configuration snippet was inserted:

  http {
      ...
      passenger_root /var/passenger/passenger-5.3.6;
      passenger_ruby /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby;
      ...
  }

After you start Nginx, you are ready to deploy any number of Ruby on Rails
applications on Nginx.

Press ENTER to continue.
...

[root@ip-172-XX-XX-XXX www]# exit
exit
[ec2-user@ip-172-XX-XX-XXX www]$
```

nginx.conf 수정

```
[ec2-user@ip-172-XX-XX-XXX www]$ sudo vi /opt/nginx/conf/nginx.conf
```

```
# nginx.conf
worker_processes  1;

events {
    worker_connections 1024;
}


http {
    server_names_hash_bucket_size 256;
    passenger_root /var/passenger/passenger-5.3.6;
    passenger_ruby /home/ec2-user/.rvm/gems/ruby-2.4.3/wrappers/ruby;

    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen                 80;
        server_name            <EC2의 퍼블릭 IP>;
        root                   /var/www/aws-exercise-a/public;
        passenger_enabled      on;
        passenger_app_type     node;
        passenger_startup_file /var/www/aws-exercise-a/app.js;
    }
}
```

nginx 서비스 시작 & EC2 인스턴스의 주소로 접속하면 서버의 응답을 받을 수 있다.

```
[ec2-user@ip-172-XX-XX-XXX www]$ sudo /opt/nginx/sbin/nginx
```

![image](https://user-images.githubusercontent.com/77096463/110288877-f9555400-802b-11eb-9353-ac2f98634b94.png)



만일 nginx 서비스를 중지하고 싶다면 `sudo /opt/nginx/sbin/nginx -s stop` <br>
만일 nginx 서비스를 재구동하고 싶다면 `sudo /opt/nginx/sbin/nginx -s reload`

------------



### 3. nginx, Phusion Passenger 서비스 명령어 추가

/etc/init.d 경로 (서비스 스크립트가 존재하는 경로)에 스크립트 추가

```
[ec2-user@ip-172-XX-XX-XXX www]$ cd /etc/init.d
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo vi nginx

# 파일 권한 변경 -> 실행 가능한 서비스 위해
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo chmod 755 nginx
```
```
# nginx

```

```
# nginx 서비스 중지
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo service nginx stop
Reloading systemd:                                         [  OK  ]
Stopping nginx (via systemctl):                            [  OK  ]

# nginx 서비스 실행
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo service nginx start
Starting nginx (via systemctl):                            [  OK  ]
```

```
# nginx 서비스 실행 상태 확인
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo service nginx status
● nginx.service - SYSV: Nginx is an HTTP(S) server, HTTP(S) reverse proxy and IMAP/POP3 proxy server
   Loaded: loaded (/etc/rc.d/init.d/nginx; bad; vendor preset: disabled)
   Active: active (running) since Mon 2021-03-08 07:39:30 UTC; 1min 4s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 23079 ExecStart=/etc/rc.d/init.d/nginx start (code=exited, status=0/SUCCESS)
 Main PID: 22501 (nginx)
   CGroup: /system.slice/nginx.service
           ‣ 22501 nginx: master process /opt/nginx/sbin/nginx

```

시스템 시작 시 자동 시작 서비스에 등록

```
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo chkconfig --add nginx
[ec2-user@ip-172-XX-XX-XXX init.d]$ sudo ntsysv
```

![image](https://user-images.githubusercontent.com/77096463/110289821-50a7f400-802d-11eb-91d9-f5b461d2ebc3.png)

-------------



### 4. 하나의 서버에서 두 개의 애플리케이션 서비스하기

애플리케이션 추가 설정

```
[ec2-user@ip-172-XX-XX-XXX init.d]$ cd /var/www

[ec2-user@ip-172-XX-XX-XXX www]$ git clone https://github.com/deopard/aws-exercise-b.git
Cloning into 'aws-exercise-b'...
remote: Enumerating objects: 10, done.
remote: Total 10 (delta 0), reused 0 (delta 0), pack-reused 10
Unpacking objects: 100% (10/10), done.
```

```
[ec2-user@ip-172-XX-XX-XXX www]$ cd aws-exercise-b

[ec2-user@ip-172-XX-XX-XXXaws-exercise-b]$ cat app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {		// aws-exercise-a와 달라지는 부분
  res.send('AWS exercise의 B project입니다.');
});

app.listen(3000, () => {
  console.log('Example app listening on port 3000!');
});

app.get('/health', (req, res) => {
  res.status(200).send();
});

[ec2-user@ip-172-XX-XX-XXX aws-exercise-b]$ npm isntall
added 50 packages from 47 contributors and audited 50 packages in 1.31s
found 0 vulnerabilities
```

nginx 설정 변경 -> 기존 내용은 그대로 두고 **server_name을 EC2 인스턴스의 DNS**로, **root와 passenger_startup_file을 aws-exercise-b**로 변경하여 추가

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-b]$ sudo vi /opt/nginx/conf/nginx.conf
```

```
# nginx.conf
...
    server{
        listen          80;
        server_name <EC2의 퍼블릭 DNS>;
        root    /var/www/aws-exercise-b/public;
        passenger_enabled       on;
        passenger_app_type      node;
        passenger_startup_file  /var/www/aws-exercise-b/app.js;
    }
    server {
        listen       80;
        server_name  <EC2의 퍼블릭 IP>;
        root         /var/www/aws-exercise-a/public;
        passenger_enabled       on;
        passenger_app_type      node;
        passenger_startup_file  /var/www/aws-exercise-a/app.js;
...
```

서버 재실행 후 접속 테스트

- EC2 ip를 이용하여 웹 서버에 접속, EC2 DNS를 이용하여 웹 서버에 접속하면 각기 다른 애플리케이션이 실행되는 것을 확인 가능하다.

```
[ec2-user@ip-172-XX-XX-XXX aws-exercise-b]$ sudo service nginx restart
Restarting nginx (via systemctl):                          [  OK  ]
```

![image](https://user-images.githubusercontent.com/77096463/110291748-9665bc00-802f-11eb-8e60-5a13beb3749e.png)








<br>

<br>



출처 : 서비스 운영이 쉬워지는 AWS 인프라 구축 가이드

