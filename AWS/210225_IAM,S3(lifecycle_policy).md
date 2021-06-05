확장성 : auto scaling 서비스 <br>
탄력성 : 수요가 떨어졌을 때 용량을 자동으로 줄이는 개념 -> 비용 효율적<br>
비용 관리 : 자본 지출(CAPEX)에서 운영 비용(OPEX)으로 IT 지출 내용을 변경<br>
ex) 사용한 리소스 양만큼만 비용 지불 & 장기적 수요 예측 가능 -> 위험 감수



AWS 클라우드 서비스 범위
- **컴퓨팅** : *물리 서버가 하는 역할*을 복제한 클라우드 서비스 
  - 물리 서버가 하는 역할 : Auto Scaling, Load Balancing, Serverless Architecture 제공
  - **EC2** : 클라우드를 통해 VM 운영 
  - **Lambda**(Serverless Architecture 대표) : 서버 없이 프로그램 실행
  - **Auto Scaling**
  - **Elastic Load Balancing**
  - **Elastic Beanstalk** : 소스코드만 올리면 실행환경은 자동으로 구축해주는 서비스
  
- **네트워킹** : 애플리케이션 연결, 액세스 제어, 접근 제어, 원격 연결 제공하는 서비스
  
  - **VPC (Virtual Private Cloud)** : 사설 네트워크 환경
  - **Direct Connect** : 전용 구축선
  - **Route 53**: DNS 서비스 (도메인 네임 -> IP 주소로 변환)
  - **CloudFront** : CDN (Contents Delivery Network) 서비스 담당 -> 같은 네트워크를 분산시켜서 네트워크 지연을 최소화
  
- **스토리지** : 빠른 액세스, 장기 백업과 같은 요구를 충족하는 스토리지 플랫폼 서비스

  - **S3 (Simple Storage Service)** : 객체 저장소
  - **Glacier** : 저렴한 비용으로 데이터 저장 -> but  데이터 액세스 시간이 많이 소요됨 
  - **EBS (Elastic Block Storage)** 
  - **Storage Gateway**  

- **데이터베이스** : Relational, NoSQL, 캐싱 등

  - **RDS (Relational Database Service)**
  - **DynamoDB** 

- 애플리케이션 관리

  - **CloudWatch** : 리소스 모니터링 서비스
  - **CloudFormation** : 배포 툴
  - **CloudTrail**
  - Config

- 보안과 자격 증명

  - **IAM** (Identity and Access Management)
  - **KMS** (Key Management Service)
  - Directory Service

- 애플리케이션 통합

  - **SNS (Simple Notification Service)**

  - **Simple Workflow** (SWF)
  - **SQS** (Simple Queue Service)
  - **API Gateway**

<br>

<br>

# 1. Introduction to AWS Identity and Access Management (IAM)

**IAM** : 사용자 권한 제어 서비스로, 자격을 인증하고 권한을 부여하는 서비스이다.<br>
사용자별로 AWS 서비스, 서비스에 생성된 자원 등에 대해 세분화된 권한을 지정하게 해준다.<br>
서비스에서 생성된 자원에 대해서도 계정과 권한을 만들어서 관리하게 해준다.<br>
사용자, 보안 자격 증명 (ex. API Access Key) 을 관리하고 사용자가 AWS 리소스에 액세스 할 수 있도록 허용한다.

<br>

**IAM 정책** 

- 기본 format은 아래와 같다.

```
{
	"Version" : "2012-10-17"
	"Statement" : [				# 정책 문서
		{
			"Resource" : "*"	# 자원
			"Action" : "*"		# 작업
			"Effect" : "Allow"	# 효과 (리소스에 대한 작업의 허용 여부 명시)
		}
	]
}
```

- 관리형 정책 : 일괄적으로 정책을 적용하고 관리한다.
![image-20210225161647148](https://user-images.githubusercontent.com/77096463/109319372-22d6e880-7892-11eb-82ca-2ba853ae90c4.png)

- 인라인 정책 : 개별적으로 정책을 적용하고 개별적으로 관리한다. -> 단일 사용자, 그룹, 역할에 직접 추가하는 방식

![image-20210225161732891](https://user-images.githubusercontent.com/77096463/109319428-34b88b80-7892-11eb-821f-7f3791acbe93.png)

### 추가 Lab

1. S3-Support Group에 EC2 인스터스 실행하고, 중지하는 권한 추가

- IAM > 그룹 -> S3-Support 그룹으로 이동
- 권한 탭 > 인라인 정책 > 정책 생성

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": "elasticloadbalancing:Describe*",
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:Describe*"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": "autoscaling:Describe*",
      "Resource": "*",
      "Effect": "Allow"
    }
  ]
}
```

![image-20210225153523746](https://user-images.githubusercontent.com/77096463/109319481-40a44d80-7892-11eb-8039-ad853e127907.png)

<br>

2. user-1 사용자가 EC2 인스턴스 조회할 수 있도록 권한 추가

- IAM > 사용자 > user-1 > 권한 추가 이동
- 권한 추가하는 3가지 방법이 있지만 '기존 정책 직접 연결' -> AmazonEC2ReadOnlyAccess 정책 선택

![image-20210225153750877](https://user-images.githubusercontent.com/77096463/109319513-4b5ee280-7892-11eb-8c11-0ea5fe7a0103.png)

<br>

3. EC2 인스턴스를 실행하고 중지할 수 있는 관리형 정책 ec2-manager 생성 후 기존 ec2-admin 인라인 정책 대신 적용

- IAM > 정책 > 정책 생성 > 아래의 json 코드 입력 -> ec2-manager 이름 -> 정책 생성 완료

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": "elasticloadbalancing:Describe*",
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:Describe*"
      ],
      "Resource": "*",
      "Effect": "Allow"
    },
    {
      "Action": "autoscaling:Describe*",
      "Resource": "*",
      "Effect": "Allow"
    }
  ]
}
```

생성된 ec2-manager 확인하기

![image-20210225154607967](https://user-images.githubusercontent.com/77096463/109319566-57e33b00-7892-11eb-95f2-9681cfd11c23.png)

- IAM > 그룹 >  EC2-Admin > 권한 탭 > 관리형 정책 탭 > ec2-manager 정책 연결
- user-1 으로 로그인하여 실행 중인 인스턴스를 중지할 수 있는지 확인 

![image-20210225154904434](https://user-images.githubusercontent.com/77096463/109319608-60d40c80-7892-11eb-9d90-9970c4ea2b44.png)

<br>

4. ec2-manager 정책에서 중지 권한 삭제 후 user-1 (S3-Support 그룹), user-3 (EC2-Admin 그룹) 계정으로 EC2 인스턴스 중지 가능 여부 확인

- IAM > 정책 > ec2-manager 정책 편집 > JSON 탭에서 'ec2:StopInstances' 항목 삭제
- user-1 으로 로그인하여 ec2 인스턴스 중지 불가 확인

![image-20210225155231924](https://user-images.githubusercontent.com/77096463/109319635-6af60b00-7892-11eb-8141-cad98b1a2de4.png)

<br>

<br>

# 2. Creating a basic Amazon S3 Lifecycle Policy

개인, 애플리케이션, AWS 서비스의 데이터 (파일) 를 보관
- 백업, 로그 파일 재해 복구 이미지 유지 관리
- 분석을 위한 빅데이터 저장
- 정적 웹 사이트 호스팅

99.999999999%의 내구성 제공 -> 데이터 유실 거의 발생하지 않음 / AWS 리전 내에서 최소 3개의 물리적 가용영역에 분산해서 저장

99.5% ~ 99.99%의 가용성 제공  



<br>

1. S3 > 버킷 > 버킷 만들기 
- 이름 : myawsbucket130 
- 모든 퍼블릭 액세스 차단 클릭 해제하기 -> 외부에서 객체 주소를 이용한 접근을 모두 차단

![image-20210225164657295](https://user-images.githubusercontent.com/77096463/109319678-76493680-7892-11eb-8251-3c71c391645c.png)

<br>

2. 생성된 myawsbucket130 클릭 > 객체 탭 > 업로드 클릭 

- https://github.com/tia-la/ccp 에서 jpg 파일 다운받아 업로드하기

![image-20210225164955307](https://user-images.githubusercontent.com/77096463/109319701-7f3a0800-7892-11eb-8c1e-545dcdf1243c.png)

<br>

3. myawsbucket130 클릭 > 관리 탭 > 수명 주기 구성 > 수명 주기 규칙 생성 클릭

- 이름 : sample-s3-to-glacier-rule
- 필터 유형 : 접두사 - pinehead 로 지정
- 수명 주기 규칙 작업 : 스토리지 클래스 간에 객체의 *현재* 버전 전환, 스토리지 클래스 간에 객체의 *이전* 버전 전환 클릭

![image-20210225165548517](https://user-images.githubusercontent.com/77096463/109319720-86611600-7892-11eb-82c4-59d530180a44.png)

<br>

4. 스토리지 클래스 간에 객체의 현재 버전 전환
- 스토리지 클래스 전환 : Glacier 
- 객체 생성 후 경과 기간(일) : 30

![image-20210225165732011](https://user-images.githubusercontent.com/77096463/109319749-8fea7e00-7892-11eb-8411-1fea3e103b93.png)

<br>

5. 스토리지 클래스 간에 객체의 이전 버전 전환
- 스토리지 클래스 전환 : Glacier Deep Archive (더욱 더 많이 안쓰는 데이터 저장 - 비용 저렴, 빠른 액세스 보장 안됨)
- 객체가 최신이 아닌 상태로 전환된 후 경과 기간(일) : 15

![image-20210225165742030](https://user-images.githubusercontent.com/77096463/109319770-9842b900-7892-11eb-9694-501a4f79b478.png)

모든 과정이 완료되면 규칙 생성 버튼 클릭하기

<br>

6. 수명 주기 규칙이 적용된 S3

![image-20210225165827764](https://user-images.githubusercontent.com/77096463/109319800-9f69c700-7892-11eb-824d-7fc9fc238baf.png)