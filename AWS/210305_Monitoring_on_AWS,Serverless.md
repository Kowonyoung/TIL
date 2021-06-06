# Monitoring and Notifications with CloudWatch Events and SNS

**CloudTrail**<br>
AWS 리소스의 모든 읽기, 쓰기 작업의 상세 로그 (작업 내역, 관련 리소스와 리전, 작업 수행자와 작업 시간 등)를 보관<br>
API 작업과 비 API 작업을 모두 기록<br>
 - API 작업 예 : 인스턴스 시작, S3 버킷 생성, VPC 생성 등
 - 비 API 작업 예 : AWS Management Console에 로그인

<br>

**Event** = AWS 계정의 활동 기록
- <u>관리 이벤트</u> = 제어 플레인 작업 (Control Plane Operations)
	- 보안 주체가 AWS 리소스에서 실행하는 작업을 포함
		- 예) 보안 구성 -> IAM AttachRolePolicy API 호출
		- 예) 디바이스 등록 -> EC2 CreateDefaultVPC API 호출
		- 예) 데이터 라우팅 규칙 구성 -> EC2 CreateSubnet API 호출
		- 예) 로깅 설정 -> CloudTrail CreateTrail API 호출
	- 계정에서 발생하는 비 API 이벤트
		- 예) 사용자가 로그인하는 경우 ConsoleLogin 이벤트가 로깅
	- 쓰기 전용, 읽기 전용으로 분류
		- 쓰기 전용 이벤트 : 리소스를 변경하거나 변경할 수 있는 API 작업
		- 읽기 전용 이벤트 : 리소스를 읽기만하고 변경하지 않는 API 작업
- <u>데이터 이벤트</u> = 데이터 플레인 작업
	- 리소스 또는 리소스 내에서 수행되는 리소스 작업에 대한 정보 제공
	- 예) 대량의 작업이 수행되는 S3 객체 수준 활동, Lambda 함수 실행
		- S3 객체 수준 활동 -> GetObject, DeleteObject, PutObject API 호출
		- Lambda 함수 실행 -> Invoke API 호출
	- 추적을 생성할 때 데이터 이벤트는 기본적으로 기록되지 않음 -> 데이터 이벤트를 기록하려면 활동을 수집할 리소스 또는 리소스 유형을 추적에 명시적으로 추가해야 함
- <u>인사이트 이벤트</u> 
  - AWS 계정의 비정상적인 활동을 캡쳐
  - 계정 API 사용량 변화가 계정의 일반적인 사용 패턴과 크게 다를 때 로깅
    - S3 deleteBucket API 호출이 평균적으로 분당 20회 호출 -> 분당 100회 호출 감지한 경우 (비정상적인 활동) -> 비정상적인 활동이 시작될 때와 정상으로 돌아갔을 때를 기록

<br>

**Event History**
- CloudTrail 이벤트에 대한 지난 90일간의 기록
- 조회, 검색, 다운로드 등이 가능
- 리전별로 '이벤트 기록'을 작성, 해당 리전에서의 활동만 기록
- IAM, Route53 등의 글로벌 서비스 이벤트는 모든 리전의 이벤트 기록에 포함

<br>

**Trail**
- 90일이 경과한 이벤트 기록을 저장하거나 CloudTrail이 기록하는 이벤트 유형을 사용자 정의할 때 생성
- 특정 이벤트를 기록하고 지정한 S3버킷에 CloudTrail 로그 파일을 전달, 로그 파일에는 JSON 형식의 로그 항목이 하나 이상 들어 있음
	- eventTime
	- userIdentity
	- eventSource
	- eventName
	- awsRegion
	- sourceIPAddress

<br>

**CloudWatch**
- AWS 리소스와 AWS에서 실시간으로 실행되는 애플리케이션 모니터링
- 리소스와 애플리케이션에 대한 지표(=측정할 수 있는 변수)를 수집하고 추적
- CloudWatch 웹 사이트에는 사용 중인 모든 AWS 서비스에 대한 지표가 자동으로 표시되고, 사용자 지정 대시보드 추가 가능
- 지표를 감시해 알림을 보내거나 임계값을 위반한 경우 모니터링 중인 리소스를 자동으로 변경하는 경보 생성
- 시스템 전체의 리소스 사용량, 애플리케이션 성능 및 운영 상태 파악

<br>

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/110055253-00683200-7da0-11eb-8806-8f948f023462.png)

<br>

### Create an SNS Topic and Subscribe an Email Address

SNS 주제 생성 
- 이름 입력
![image](https://user-images.githubusercontent.com/77096463/110052407-e8da7a80-7d9a-11eb-9ad6-4245cc17b379.png)

<br>

구독생성 -> 이후 confirmation 진행
![image](https://user-images.githubusercontent.com/77096463/110052448-f6900000-7d9a-11eb-8fc9-e4d19ee4ab9a.png)

<br>

구독 확인 됨 -> 구독 상태가 '확인됨'
![image](https://user-images.githubusercontent.com/77096463/110052546-13c4ce80-7d9b-11eb-9b17-d406ee21728d.png)

<br>


### Create a CloudWatch Events Rule to Trigger the SNS Topic When There is a State Change to an EC2 Instance

CloudWatch 이벤트 규칙 생성
- 서비스 : EC2
- 이벤트 유형 : EC2 instance State-change Notification
- 대상 : 생성한 SNS 주제 선택
![image](https://user-images.githubusercontent.com/77096463/110052725-643c2c00-7d9b-11eb-900b-0421f866a16d.png)

<br>

이벤트 규칙 이름 부여
![image](https://user-images.githubusercontent.com/77096463/110052794-803fcd80-7d9b-11eb-98c1-bd7137563634.png)

<br>


### Change the State of the EC2 Instance, and Verify the Receipt of the SNS Notification

인스턴스 상태를 중지로 변경한 후 메일이 오는지 확인
![image](https://user-images.githubusercontent.com/77096463/110053059-fe03d900-7d9b-11eb-8c22-b4fd346a33db.png)

<br>

인스턴스 중지 메일
![image](https://user-images.githubusercontent.com/77096463/110053084-0e1bb880-7d9c-11eb-913b-11c0e9502e3e.png)

<br>

인스턴스 재시작 -> 시작 메일
![image](https://user-images.githubusercontent.com/77096463/110053156-32779500-7d9c-11eb-9568-87232917f6fc.png)

<br>

# **AWS EC2 Custom Logging with CloudWatch**

EC2 인스턴스에서 생성되는 로그 정보를 **CloudWatch로 전송해서 로그를 통합** -> EC2 인스턴스에 CloudWatch Logs 에이전트를 설치하고, 로그 서비스를 켜고, 메시지를 수신하도록 CloudWatch를 구성

1. EC2 인스턴스 생성

![image](https://user-images.githubusercontent.com/77096463/110057144-6d30fb80-7da3-11eb-924c-c1ccabb52e94.png)

![image](https://user-images.githubusercontent.com/77096463/110057194-85087f80-7da3-11eb-9c29-4607c97eae09.png)

<br>

2. SSH 접속

![image](https://user-images.githubusercontent.com/77096463/110057363-d31d8300-7da3-11eb-90a7-2354b597f61e.png)

<br>

3. EC2 인스턴스에 awslogs 추가

```
[ec2-user@ip-10-0-0-28 ~]$ sudo yum update -y
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
No packages marked for update

[ec2-user@ip-10-0-0-28 ~]$ sudo yum install -y awslogs
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Resolving Dependencies
--> Running transaction check
---> Package awslogs.noarch 0:1.1.4-3.amzn2 will be installed
--> Processing Dependency: aws-cli-plugin-cloudwatch-logs for package: awslogs-1.1.4-3.amzn2.noarch
--> Running transaction check
---> Package aws-cli-plugin-cloudwatch-logs.noarch 0:1.4.6-1.amzn2.0.1 will be installed
--> Finished Dependency Resolution
```

```
[ec2-user@ip-10-0-0-28 ~]$ cd /etc/awslogs

[ec2-user@ip-10-0-0-28 awslogs]$ ls -l
total 20
-rw------- 1 root root   55 Mar  5 02:14 awscli.conf	//자격증명과 지역정보 포함
-rw-r--r-- 1 root root 8355 Jul 25  2018 awslogs.conf	//cloudwatch 로깅에 대한 설정 정보 포함
drwxr-xr-x 2 root root    6 Jul 25  2018 config
-rw-r--r-- 1 root root  147 Jul 25  2018 proxy.conf
```

4. awslogs 서비스 시작

```
[ec2-user@ip-10-0-0-28 awslogs]$ sudo systemctl start awslogsd
```

awslogs 에이전트가 생성하는 로그 확인
```
[ec2-user@ip-10-0-0-28 awslogs]$ tail -f /var/log/awslogs.log
2021-03-05 02:18:37,216 - cwlogs.push - INFO - 3454 - MainThread - Missing or invalid value for queue_size config. Defaulting to use 10
2021-03-05 02:18:37,216 - cwlogs.push - INFO - 3454 - MainThread - Using default logging configuration.
2021-03-05 02:18:37,233 - cwlogs.push.stream - INFO - 3454 - Thread-1 - Starting publisher for [67811a06abe66c18d08cf0c15c225182, /var/log/messages]
2021-03-05 02:18:37,238 - cwlogs.push.stream - INFO - 3454 - Thread-1 - Starting reader for [67811a06abe66c18d08cf0c15c225182, /var/log/messages]
2021-03-05 02:18:37,239 - cwlogs.push.reader - INFO - 3454 - Thread-4 - Start reading file from 0.
2021-03-05 02:18:43,312 - cwlogs.push.publisher - WARNING - 3454 - Thread-3 - Caught exception: An error occurred (ResourceNotFoundException) when calling the PutLogEvents operation: The specified log group does not exist.
2021-03-05 02:18:43,313 - cwlogs.push.batch - INFO - 3454 - Thread-3 - Creating log group /var/log/messages.
2021-03-05 02:18:43,372 - cwlogs.push.batch - INFO - 3454 - Thread-3 - Creating log stream i-04f1d792e488764ef.
2021-03-05 02:18:43,475 - cwlogs.push.publisher - INFO - 3454 - Thread-3 - Log group: /var/log/messages, log stream: i-04f1d792e488764ef, queue size: 0, Publish batch: {'skipped_events_count': 0, 'first_event': {'timestamp': 1614910326000, 'start_position': 0L, 'end_position': 161L}, 'fallback_events_count': 0, 'last_event': {'timestamp': 1614910716000, 'start_position': 67185L, 'end_position': 67250L}, 'source_id': '67811a06abe66c18d08cf0c15c225182', 'num_of_events': 783, 'batch_size_in_bytes': 86825}
2021-03-05 02:18:48,509 - cwlogs.push.publisher - INFO - 3454 - Thread-3 - Log group: /var/log/messages, log stream: i-04f1d792e488764ef, queue size: 0, Publish batch: {'skipped_events_count': 0, 'first_event': {'timestamp': 1614910722000, 'start_position': 67250L, 'end_position': 67336L}, 'fallback_events_count': 0, 'last_event': {'timestamp': 1614910722000, 'start_position': 67250L, 'end_position': 67336L}, 'source_id': '67811a06abe66c18d08cf0c15c225182', 'num_of_events': 1, 'batch_size_in_bytes': 111}
```

5. 부팅 시 awslogs 서비스 자동 실행

```
[ec2-user@ip-10-0-0-28 awslogs]$ sudo systemctl enable awslogsd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/awslogsd.service to /usr/lib/systemd/system/awslogsd.service.
```

6. EC2에서 보낸 CloudWatch Logs 확인 (수집된 로그 확인) <br>
EC2 인스턴스의 /var/log/messages 파일과 CloudWatch의 /var/log/messages 로그 그룹에 수집된 내용이 일치

![image](https://user-images.githubusercontent.com/77096463/110058510-c863ed80-7da5-11eb-9a3b-a7b8082dc7e7.png)

참고 : https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/QuickStartEC2Instance.html

<br>

# AWS Access Control Alerts with CloudWatch and CloudTrail

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/110065935-b2f5c000-7db3-11eb-85ce-2539ca0f2c47.png)

<br>

### Create an S3 Bucket

S3 버킷은 중요 정보를 담고 있으므로 오브젝트의 변화를 **모니터링** 해야 함<br>
우선 top-secret-bucket2 이름의 버킷을 생성한 후, 객체 업로드를 통해 중요 파일을 업로드한다.

![image](https://user-images.githubusercontent.com/77096463/110067345-e554ec80-7db6-11eb-9393-b3b10c91e329.png)

<br>

### Create a CloudTrail Trail 

CloudTrail 서비스 이동 > 추적 생성
- 이름 : top-secret-bucket-trail
- 스토리지 위치 : 새 S3 버킷 생성
- 추적 로그 버킷 및 폴더 : top-secret-bucket-trail-2


![image](https://user-images.githubusercontent.com/77096463/110068143-6660b380-7db8-11eb-8973-99bcc14184db.png)

<br>

로그 이벤트로 '데이터 이벤트' 선택<br>
앞서 생성한 top-secret-bucket2 S3 버킷 선택 > 읽기, 쓰기 권한 모두 체크

![image](https://user-images.githubusercontent.com/77096463/110068332-cbb4a480-7db8-11eb-9d29-8c8c9e692b03.png)

![image](https://user-images.githubusercontent.com/77096463/110068293-b63f7a80-7db8-11eb-9f24-7fda40595e98.png)

<br>

### Create a CloudWatch Log Group

top-secret-bucket-trail에서 cloudwatch와 연결하기 위해 CloudWatch 로그 그룹을 생성

- CloudWatch Logs 활성화됨 체크
- IAM 역할 : 기존
- 역할 이름 : CloudTrailCloudWatchRole

![image](https://user-images.githubusercontent.com/77096463/110068674-847ae380-7db9-11eb-824c-840a41ecb703.png)

<br>

### Set Up a CloudWatch Alarm

CloudWatch 지표 설정

```
{ ($.eventSource = s3.amazonaws.com) && (($.eventName = PutObject) || ($.eventName = GetObject)) }
```

![image](https://user-images.githubusercontent.com/77096463/110069098-6f528480-7dba-11eb-9e53-71a071e6e20f.png)

<br>

지표 이름 : PutObject나 GetObject 필터링에 해당하는 로그를 AccessMetrics (라벨을 부여) <br>
지표 네임스페이스 : 그러한 개별 필터에 해당하는 로그들을 그룹핑해서 LogMetrics

![image](https://user-images.githubusercontent.com/77096463/110069380-04557d80-7dbb-11eb-9ddf-1c8422eb3cbc.png)

<br>

경보를 알릴 데이터 포인트 :  조건이 한 번 발생 / 1번의 기간 (5분) -> 알람 발생 처리

![image](https://user-images.githubusercontent.com/77096463/110070052-767a9200-7dbc-11eb-8192-df9d70c06c0f.png)

<br>

경보 상태일 경우 이메일로 상태를 전송하기 위해 새 SNS 주제와 이메인 엔드포인트 설정 

![image](https://user-images.githubusercontent.com/77096463/110070225-d4a77500-7dbc-11eb-8a82-4bada2f8c78d.png)

<br>

s3 버킷에 객체 업로드-> CloudTrail로 인해 로그가 쌓인다.
![image](https://user-images.githubusercontent.com/77096463/110070882-4f24c480-7dbe-11eb-97b3-8845fafa1306.png)

<br>

S3 버킷에 파일 업로드 후 경보 발생 확인 

![image](https://user-images.githubusercontent.com/77096463/110071091-c3f7fe80-7dbe-11eb-9bd5-ad58af9c532e.png)

<br>

이메일로 알람 발송된 내역 확인

![image](https://user-images.githubusercontent.com/77096463/110071187-f3a70680-7dbe-11eb-80e8-585bf963d2fb.png)

<br>

# Creating a Simple AWS Lambda Function

### Lab 구성도
![image](https://user-images.githubusercontent.com/77096463/110073660-15a28800-7dc3-11eb-9f5f-88c88ef27bee.png)

### Create a Lambda Function within the AWS Lambda Console

람다 함수 생성

![image](https://user-images.githubusercontent.com/77096463/110072903-dde71080-7dc1-11eb-9b54-ac039dabc5c4.png)

<br>

생성된 람다 함수의 권한을 확인하는 데 이때 CloudWatch Logs, Lambda 서비스가 설정되어 있는지 확인한다.

![image](https://user-images.githubusercontent.com/77096463/110075227-d0cc2080-7dc5-11eb-9cd5-ad83938b1948.png)

<br>

람다 함수의 코드를 수정하여 메시지를 출력하도록 한다.

![image](https://user-images.githubusercontent.com/77096463/110072943-eb9c9600-7dc1-11eb-8c9c-a41a0af2a566.png)

<br>

### Create a Test Event and Manually Invoke the Function Using the Test Event

앞서 작성한 람다코드는 `event['message']`를 호출하므로 테스트에서 message에 해당하는 키가 있어야 한다. 따라서 새로이 message를 포함한 Test Event를 작성한다. 

![image](https://user-images.githubusercontent.com/77096463/110073093-2ef70480-7dc2-11eb-89ba-e29e83c6796b.png)

<br>

테스트 이벤트 작성 후, 테스트를 해보면 결과를 곧바로 확인할 수 있다.

![image](https://user-images.githubusercontent.com/77096463/110075854-d5450900-7dc6-11eb-93d5-ed47efbd0b56.png)

<br>

### Verify That CloudWatch Has Captured Function Logs

CloudWatch Logs로 이동하여 배포한 람다 함수 로그 확인

![image](https://user-images.githubusercontent.com/77096463/110073745-44206300-7dc3-11eb-8462-0c5ece95fa84.png)

<br>

# Using the AWS CLI to Create an AWS Lambda Function

람다 함수를 클라이언트에서 작성하고 AWS CLI를 통해 배포 및 실행

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/110077304-3a99f980-7dc9-11eb-8fce-d9ab1a0bd4b5.png)

<br>

### Verify Lab Resources and Configure AWS CLI

S3, EC2, IAM 역할 확인<br>
ssh 접속 후 AWS CLI 설치 여부 확인

```
[cloud_user@ip-10-99-1-4 ~]$ aws --version
aws-cli/1.16.113 Python/2.7.16 Linux/4.14.173-106.229.amzn1.x86_64 botocore/1.12.103
```
us-east-1 리젼의 람다 함수 목록 확인
```
[cloud_user@ip-10-99-1-4 ~]$ aws lambda list-functions --region us-east-1
{
    "Functions": []
}
```

<br>

### Create a Lambda Function Using the AWS CLI

S3 버킷 목록을 반환하는 람다 함수 작성

```
[cloud_user@ip-10-99-1-4 ~]$ vim lambda_function.js
```

```js
// lambda_function.js

// aws-sdk 모듈 가져오기
const AWS = require('aws-sdk');

// We need to set the region.
AWS.config.update({region: 'us-east-1'});

// Creating S3 service object
const s3 = new AWS.S3({apiVersion: '2006-03-01'});

// 함수 한 번만 쓸 때는 함수 이름 생략 (익명함수)
exports.handler = (event, context, callback) => {
    // Here we list all S3 Buckets
    s3.listBuckets(function(err, data) {
       if (err) {
          console.log("Error:", err);
       } else {
          console.log("List of S3 Buckets", data.Buckets);
       }
    });
};
```
<br>

배포하기 위해 zip 파일로 압축

```
[cloud_user@ip-10-99-1-4 ~]$ zip lambda_function.zip lambda_function.js
  adding: lambda_function.js (deflated 33%)
```

`create-function` 명령어를 사용해 람다 함수 생성

- IAM 역할에서 확인한 것으로 입력

```
[cloud_user@ip-10-99-1-4 ~]$ aws lambda create-function \
--region us-east-1 \
--function-name "ListS3Buckets" \
--runtime "nodejs12.x" \
--role "arn:aws:iam::876698693386:role/lambda_exec_role_LA" \
--handler "lambda_function.handler" \
--zip-file fileb:///home/cloud_user/lambda_function.zip
{
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "CodeSha256": "AmX6m+XakilIKvo+upFluSJjS+65J0Ygl1/xg8wATx8=",
    "FunctionName": "ListS3Buckets",
    "CodeSize": 420,
    "RevisionId": "525275dc-6902-479f-b9f4-691821ec3e3f",
    "MemorySize": 128,
    "FunctionArn": "arn:aws:lambda:us-east-1:876698693386:function:ListS3Buckets",
    "Version": "$LATEST",
    "Role": "arn:aws:iam::876698693386:role/lambda_exec_role_LA",
    "Timeout": 3,
    "LastModified": "2021-03-05T07:03:34.765+0000",
    "Handler": "lambda_function.handler",
    "Runtime": "nodejs12.x",
    "Description": ""
}
```

```
[cloud_user@ip-10-99-1-4 ~]$ aws lambda list-functions --region us-east-1
{
    "Functions": [
        {
            "TracingConfig": {
                "Mode": "PassThrough"
            },
            "Version": "$LATEST",
            "CodeSha256": "AmX6m+XakilIKvo+upFluSJjS+65J0Ygl1/xg8wATx8=",
            "FunctionName": "ListS3Buckets",
            "MemorySize": 128,
            "RevisionId": "525275dc-6902-479f-b9f4-691821ec3e3f",
            "CodeSize": 420,
            "FunctionArn": "arn:aws:lambda:us-east-1:876698693386:function:ListS3Buckets",
            "Handler": "lambda_function.handler",
            "Role": "arn:aws:iam::876698693386:role/lambda_exec_role_LA",
            "Timeout": 3,
            "LastModified": "2021-03-05T07:03:34.765+0000",
            "Runtime": "nodejs12.x",
            "Description": ""
        }
    ]
}
```

AWS Console 가서 생성한 람다 함수 확인

![image](https://user-images.githubusercontent.com/77096463/110079553-9914a700-7dcc-11eb-98e0-738af47d0acd.png)

<br>

### Invoke Your Function Using AWS CLI

람다 함수 호출 & 함수의 실행 결과를 파일(OUTPUT.log) 로 저장하는 명령 입력
- `cat OUPUT.log` 를 통해 파일의 내용을 확인하면 반환값이 없기 때문에 아무 내용이 없다. (null)

```
[cloud_user@ip-10-99-1-4 ~]$ aws lambda invoke --region us-east-1 --function-name "ListS3Buckets" OUTPUT.log
{
    "ExecutedVersion": "$LATEST",
    "StatusCode": 200
}
```

CloudWatch 에서 로그 그룹 확인

- lambda_function.js의 `console.log("List of S3 Buckets", data.Buckets);` 부분 출력

![image](https://user-images.githubusercontent.com/77096463/110079886-04f70f80-7dcd-11eb-9e7d-ae171a0e9d48.png)

<br>

`cat OUTPUT.log` 명령어를 통해 반환 값을 확인하기 위해 코드 수정 -> callback 함수를 통해 호출 결과가 호출한 쪽으로 넘어간다.

![image](https://user-images.githubusercontent.com/77096463/110260880-5384f380-7ff1-11eb-8b00-7d807f2f481e.png)

```js
// lambda_function.js
const AWS = require('aws-sdk');

AWS.config.update({ region: 'us-east-1' });

const s3 = new AWS.S3({ apiVersion: '2006-03-01' });

exports.handler = (event, context, callback) => {
	s3.listBuckets(
		(err, data) => {
			if (err) {
				console.log("Error: ", err);
			} else {
				console.log("List of S3 Buckets", data.Buckets);
                callback(null, data.Buckets);
			}
		}
	);
};
```

OUTPUT.log 파일 내용 확인

```
[cloud_user@ip-10-99-1-4 ~]$ cat OUTPUT.log
[{"Name":"cfst-660-2792a7f96ac22227cf737205bc3-labs3bucketa-1hlzna6on4r73","CreationDate":"2021-03-08T00:13:56.000Z"},{"Name":"cfst-660-2792a7f96ac22227cf737205bc3-labs3bucketb-6bc8xv1vxtft","CreationDate":"2021-03-08T00:13:56.000Z"}]
```



### 참고 : Serverless Framework

https://myanjini.tistory.com/entry/Serverless-Framework-1

https://myanjini.tistory.com/entry/Serverless-Framework-2

https://myanjini.tistory.com/entry/Serverless-Framework-3-%EC%8B%A4%ED%96%89-%ED%99%98%EA%B2%BD-%EC%A0%9C%ED%95%9C-%EC%84%A4%EC%A0%95

https://myanjini.tistory.com/entry/Serverless-Framework-4-%EB%9E%8C%EB%8B%A4-%ED%95%A8%EC%88%98-%EC%8B%A4%ED%96%89%EC%97%90-%ED%95%84%EC%9A%94%ED%95%9C-%EA%B6%8C%ED%95%9C-%EC%84%A4%EC%A0%95

https://myanjini.tistory.com/entry/Serverless-Framework-5-%EB%9E%8C%EB%8B%A4-%ED%95%A8%EC%88%98-%EC%8B%A4%ED%96%89%EC%97%90-%ED%95%84%EC%9A%94%ED%95%9C-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EC%84%A4%EC%A0%95

https://myanjini.tistory.com/entry/Serverless-Framework-6-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80%EC%9D%98-%EC%8D%B8%EB%84%A4%EC%9D%BC-%EC%9E%90%EB%8F%99-%EC%83%9D%EC%84%B1



-------------------------------



# **Triggering Lambda from Amazon SQS**

### Lab 구성도

SQS를 사용해서 람다 함수를 트리거 하는 방법 -> SQS 대기열의 메시지를 처리하고 메시지 데이터를 DynamoDB 테이블에 삽입

![image](https://user-images.githubusercontent.com/77096463/110242768-dc247500-7f9a-11eb-8b1c-5c8bfa3b1fe3.png)

<br>

### Create IAM Policy

lambda_execution_policy 이름의 정책을 아래와 같이 생성한다.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*",
      "Effect": "Allow"
    },
    {
      "Action": [
        "dynamodb:*",
        "sqs:*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

### Create IAM Role

람다 함수를 트리거할 예정이므로 사용 사례는 'Lambda'로 선택

![image](https://user-images.githubusercontent.com/77096463/110242978-c1063500-7f9b-11eb-9f55-10ea120d4ed7.png)

<br>

생성한 lambda_execution_policy 정책과 연결

![image](https://user-images.githubusercontent.com/77096463/110242987-cc596080-7f9b-11eb-9e29-f8bf5e25ae9a.png)

<br>

해당 역할에 lambda_exeuction_role 이름을 붙이고 반드시 lambda_execution_policy 와 연결되어 있는지 확인한 후 생성 버튼을 누른다.

![image](https://user-images.githubusercontent.com/77096463/110242995-d8ddb900-7f9b-11eb-9ccf-ce9161f6403f.png)

<br>

### Create the Lambda Function

람다 함수를 생성한다.

- 함수 이름 : SQSDynamoDB
- 런타임 : Python3.7
- 실행 역할 : 기존 역할 사용 -> lambda_execution_role 선택

![image](https://user-images.githubusercontent.com/77096463/110243103-4a1d6c00-7f9c-11eb-91f2-a466f9f35c57.png)

<br>

아래의 python 코드를 lambda_function.py 영역에 작성한 후 DEPLOY 하기

![image](https://user-images.githubusercontent.com/77096463/110243138-789b4700-7f9c-11eb-936f-786828deceff.png)

```python
# Lambda_Function_code
import json
import os
from datetime import datetime

import boto3

QUEUE_NAME = os.environ['QUEUE_NAME']	#환경변수 가져오기
MAX_QUEUE_MESSAGES = os.environ['MAX_QUEUE_MESSAGES']
DYNAMODB_TABLE = os.environ['DYNAMODB_TABLE']

sqs = boto3.resource('sqs')
dynamodb = boto3.resource('dynamodb')


def lambda_handler(event, context):

    # Receive messages from SQS queue
    queue = sqs.get_queue_by_name(QueueName=QUEUE_NAME)

    print("ApproximateNumberOfMessages:",
          queue.attributes.get('ApproximateNumberOfMessages'))

    for message in queue.receive_messages(
            MaxNumberOfMessages=int(MAX_QUEUE_MESSAGES)):

        print(message)

        # Write message to DynamoDB
        table = dynamodb.Table(DYNAMODB_TABLE)

        response = table.put_item(
            Item={
                'MessageId': message.message_id,
                'Body': message.body,
                'Timestamp': datetime.now().isoformat()
            }
        )
        print("Wrote message to DynamoDB:", json.dumps(response))

        # Delete SQS message
        message.delete()
        print("Deleted message:", message.message_id)
```

<br>

코드에 작성된 3개의 환경 변수 (QUEUE_NAME, MAX_QUEUE_MESSAGES, DYNAMODB_TABLE) 를 가져오기 위해 환경 변수의 값을 설정한다.

![image](https://user-images.githubusercontent.com/77096463/110243244-e6e00980-7f9c-11eb-8df8-a7fb8fe331a4.png)

<br>

아래와 같이 QUEUE_NAME에는 Messages가, DynamoDB에는 Message 테이블을 환경 변수의 값으로 설정할 수 있다.

[QUEUE_NAME]

![image](https://user-images.githubusercontent.com/77096463/110243382-800f2000-7f9d-11eb-8f0e-e0264a26bc6e.png)

<br>

[DYNAMODB_TABLE]

![image](https://user-images.githubusercontent.com/77096463/110243398-8c937880-7f9d-11eb-97f3-9cb8d24f93d3.png)

<br>

### Add a Trigger

생성한 람다함수 클릭한 후 '+트리거 추가' 버튼을 클릭하여 트리거 구성 설정

- SQS 대기열 : Messages

![image](https://user-images.githubusercontent.com/77096463/110243522-18a5a000-7f9e-11eb-8b51-bcd51cfd8861.png)

<br>

트리거 구성 후 연결된 모습

![image](https://user-images.githubusercontent.com/77096463/110261494-d60eb280-7ff3-11eb-8a5c-e8b61fe0f253.png)

<br>

### SSH

send_message.py 실행을 위한 python3.7 및 필요 모듈 설치

```
[cloud_user@ip-10-1-10-208 ~]$ sudo yum install gcc openssl-devel bzip2-devel libffi-devel
[cloud_user@ip-10-1-10-208 ~]$ cd /opt
[cloud_user@ip-10-1-10-208 ~]$ sudo wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
[cloud_user@ip-10-1-10-208 ~]$ sudo tar xzf Python-3.7.9.tgz
[cloud_user@ip-10-1-10-208 ~]$ cd Python-3.7.9/
[cloud_user@ip-10-1-10-208 Python-3.7.9]$ sudo ./configure --enable-optimizations
[cloud_user@ip-10-1-10-208 Python-3.7.9]$ sudo make altinstall
[cloud_user@ip-10-1-10-208 Python-3.7.9]$ sudo rm /usr/src/Python-3.7.9.tgz
[cloud_user@ip-10-1-10-208 Python-3.7.9]$ cd
[cloud_user@ip-10-1-10-208 ~]$ python3.7 -V
Python 3.7.9
[cloud_user@ip-10-1-10-208 ~]$ pip3.7 install boto3
[cloud_user@ip-10-1-10-208 ~]$ pip3.7 install Faker
```

메시지 생성

```
[cloud_user@ip-10-1-10-208 ~]$ ls
send_message.py

[cloud_user@ip-10-1-10-208 ~]$ cat send_message.py
#!/usr/bin/env python3.7
# -*- coding: utf-8 -*-
import argparse
import logging
import sys
from time import sleep
import boto3
from faker import Faker


parser = argparse.ArgumentParser()
parser.add_argument("--queue-name", "-q", required=True,
                    help="SQS queue name")
parser.add_argument("--interval", "-i", required=True,
                    help="timer interval", type=float)
parser.add_argument("--message", "-m", help="message to send")
parser.add_argument("--log", "-l", default="INFO",
                    help="logging level")
args = parser.parse_args()

if args.log:
    logging.basicConfig(
        format='[%(levelname)s] %(message)s', level=args.log)

else:
    parser.print_help(sys.stderr)

sqs = boto3.client('sqs')

response = sqs.get_queue_url(QueueName=args.queue_name)

queue_url = response['QueueUrl']

logging.info(queue_url)

while True:
    message = args.message
    if not args.message:
        fake = Faker()
        message = fake.text()

    logging.info('Sending message: ' + message)

    response = sqs.send_message(
        QueueUrl=queue_url, MessageBody=message)

    logging.info('MessageId: ' + response['MessageId'])
    sleep(args.interval)
```

```
[cloud_user@ip-10-1-10-208 ~]$ ./send_message.py -q Messages -i 0.1
[INFO] https://queue.amazonaws.com/090823763833/Messages
[INFO] Sending message: Bad buy quickly general but new serious. Reveal onto reduce development goal Mr approach. Environmental billion sport perhaps what with she. Poor tax door give painting reason.
[INFO] MessageId: 1315ba0b-3ea3-4d35-901b-5bfb2c79cbd0
[INFO] Sending message: Full past today grow. Theory customer effort structure even happy. Should where local focus rule bit anything.
Quality list necessary capital family best away. Without camera individual site.
[INFO] MessageId: 7f9ef02d-f26f-446d-9f98-9a3a1292609c
[INFO] Sending message: Common head government reason his marriage.
Late develop total stage fear few. Level institution guy message take point we.
[INFO] MessageId: 36d85b9f-d516-4228-8b97-e00dfb8350b2
[INFO] Sending message: Boy begin carry sit art war able card. Official fact color special catch condition month morning.
[INFO] MessageId: 4556412f-0146-4adb-b4b4-9105f435133b
[INFO] Sending message: Democrat too trouble television. Mrs show charge. Section discover wish quite.
Buy him manager she special. Little later evening possible son. Can whole body section side best prove possible.
...
```

<br>

CloudWatch 로그 확인

![image](https://user-images.githubusercontent.com/77096463/110262548-6d293980-7ff7-11eb-8731-a590c3620e7c.png)

<br>

DynamoDB 테이블에 데이터가 생성됨을 확인

![image](https://user-images.githubusercontent.com/77096463/110262619-95189d00-7ff7-11eb-9419-5604d7292cb0.png)