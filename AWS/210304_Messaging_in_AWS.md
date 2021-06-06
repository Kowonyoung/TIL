# Creating and Subscribing to AWS SNS Topics

**Amazon SNS (Simple Notification Service)** <br>
게시자 (Publishers) -> 구독자 (Subscribers)로 메시지 전달 (전송) 조정, 관리하는 관리형 서비스 <br>
게시자 : 주제에 대한 메시지를 생산, 발송하여 구독자와 비동시적으로 통신하는 논리적 액세스 및 커뮤니케이션 채널 <br>
구독자 : 주제를 구독하는 경우 지원되는 프로토콜 중 하나를 통해 메시지 또는 알림을 소비하거나 수신하는 주체 (웹 서버, 이메일 주소, Amazon SQS 대기열, Lambda 함수)

![image-20210304093857332](https://user-images.githubusercontent.com/77096463/109893217-9895e680-7cce-11eb-8786-23856b2e1971.png)

<br>

![image](https://user-images.githubusercontent.com/77096463/109893159-7dc37200-7cce-11eb-8f4c-0cb15bd55c0d.png)

주제 (Topic)

- 통신 채널 역할을 하는 논리적 액세스 포인트
- 주제를 사용해 여러 엔드포인트 (Lambda, SQS, HTTP/S, 이메일 등)을 그룹화할 수 있음

구독 (Subscribe)
- 주제에 게시된 메시지를 수신할 수 있도록 주제에 대한 엔드포인트 등록

게시 (Publish)
- 주제에 메시지 등록

사용 예 ) 
- Fadeout - 하나의 데이터에서 나누어지는 것 (확산)

![image-20210304094524044](https://user-images.githubusercontent.com/77096463/109893233-a481a880-7cce-11eb-853c-8a018761621a.png)

- 경고 - 사전에 정의된 임계값에 의해 트리거되는 알림
- 사용자에게 문자 메시지(SMS) 또는 이메일 등으로 알림
- 모바일 푸시 알림

<br>

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/109895606-d6950980-7cd2-11eb-9d81-c3d95f542615.png)

<br>

### Create an SNS Topic

SNS > mytopic 토픽 생성 (FIFO - 순서가 중요할 때 / 표준 - 순서가 중요하지 않을 때) <br>
구독 > 구독 생성 클릭 <br>

- 주제 ARN : 드롭다운 메뉴에서 선택
- 프로토콜 : 이메일
- 엔드포인트 : 이메일 주소 

![image](https://user-images.githubusercontent.com/77096463/109893648-61740500-7ccf-11eb-94e0-7fca54e2f5f5.png)

<br>

입력한 이메일로 Confirmation 메일이 오고, 이를 확인하면 된다.

![image](https://user-images.githubusercontent.com/77096463/109894307-8fa61480-7cd0-11eb-9ba5-74f57a7c7a6c.png)

<br>

이번에는 SMS 구독을 생성한다.

- 주제 ARN : 드롭다운 메뉴에서 선택
- 프로토콜 : SMS
- 엔드포인트 : 핸드폰 번호 

![image](https://user-images.githubusercontent.com/77096463/109894849-8b2e2b80-7cd1-11eb-99d8-d6d04805d490.png)

<br>

### Create a Lambda Function

람다 > 함수 생성 

- 함수 이름 : SNSProcessor
- 런타임 : Python 3.6
- 실행 역할 : 기존 역할 사용
- 기존 역할 : LambdaRoleLA

![image](https://user-images.githubusercontent.com/77096463/109893858-c7608c80-7ccf-11eb-8f50-c4f0453d4d83.png)

<br>

SNS > 구독 생성

- 주제 ARN :  드롭다운 메뉴에서 선택
- 프로토콜 : AWS Lambda
- 엔드포인트 : 드롭다운 메뉴에서 선택

![image](https://user-images.githubusercontent.com/77096463/109893939-e5c68800-7ccf-11eb-822a-6562284f1342.png)

<br>

지금까지 총 3개의 구독이 만들어져야 한다. 목록 확인하기

![image](https://user-images.githubusercontent.com/77096463/109896029-941ffc80-7cd3-11eb-9c12-9e04df4057b9.png)

<br>

생성한 람다 함수 선택 > 코드 소스에 아래와 같은 코드 작성한 후 **Deploy** 버튼 누르기

- 람다 함수 핸들러 : https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/python-handler.html 참고

```
SNS topic notification을 통해서 람다 함수가 호출되는 경우, event 인자를 통해서 전달되는 값의 예
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:us-east-1:{{{accountId}}}:ExampleTopic",
      "Sns": {
        "Type": "Notification",
        "MessageId": "95df01b4-ee98-5cb9-9903-4c221d41eb5e",
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:ExampleTopic",
        "Subject": "example subject",
        "Message": "example message",	⇐ event['Records'][0]['Sns']['Message']
        "Timestamp": "1970-01-01T00:00:00.000Z",
        "SignatureVersion": "1",
        "Signature": "EXAMPLE",
        "SigningCertUrl": "EXAMPLE",
        "UnsubscribeUrl": "EXAMPLE",
        "MessageAttributes": {
          "Test": {
            "Type": "String",
            "Value": "TestString"
          },
          "TestBinary": {
            "Type": "Binary",
            "Value": "TestBinary"
          }
        }
      }
    }
  ]
}

```

![image](https://user-images.githubusercontent.com/77096463/109894032-13133600-7cd0-11eb-821c-268c50ec6600.png)
<br>

### Send Your SNS Topic to Multiple Endpoints

SNS > 주제 > mytopic 선택 > 메시지 게시 

- 제목 : An AWS Topic
- 메시지 본문 : Hello, this is our first message

![image](https://user-images.githubusercontent.com/77096463/109894131-45249800-7cd0-11eb-9304-6858718e075a.png)

<br>

위 과정을 마친다면 핸드폰과 이메일로 메시지를 받아볼 수 있고 람다 함수 구독은 람다 함수 > 모니터링 > CloudWatch에서 함수 로그를 확인한다.  

![image](https://user-images.githubusercontent.com/77096463/109898077-fd553f00-7cd6-11eb-9742-ce7e868e2656.png)

<br>

![image](https://user-images.githubusercontent.com/77096463/109894224-6d13fb80-7cd0-11eb-890a-2a8eb6757bee.png)

<br>

# Configuring SNS Push Notifications on S3 Bucket Events Inside of the AWS Console

### Create an S3 Bucket

S3 > 버킷 생성
- 버킷 이름 : sns-notifications-s3-test-errol
- 리전 : us-east-1
- 태그 : 키- CreatedBy / 값 - Errol

![image](https://user-images.githubusercontent.com/77096463/109901043-abfb7e80-7cdb-11eb-94a7-dd80d8d09057.png)

<br>

### Create an SNS Topic
SNS > 주제 생성

- 유형 : 표준
- 이름 : S3Events
- 태그 : 키 - CreatedBy / 값 - Errol

![image](https://user-images.githubusercontent.com/77096463/109901129-d77e6900-7cdb-11eb-98b2-71ff265f5d5f.png)

<br>

### Configure the Bucket — Part 1

S3 > 생성한 토픽 클릭 > 이벤트 알림 생성

- 이벤트 이름 : S3ObjectCreated

- 이벤트 유형 : 모든 객체 생성 이벤트 체크 -> S3에 파일이 등록되면 내가 만든 SNS 주제로 전송되게끔

![image](https://user-images.githubusercontent.com/77096463/109901441-6db28f00-7cdc-11eb-8f07-25b285c830b6.png)

<br>

대상 설정 -> **모든 설정을 마친 후 변경 사항 저장 버튼을 클릭하면 권한이 없기 때문에 알 수 없는 오류가 난다.** (현재 상황에서는 당연한 결과)

- 대상 : SNS 주제
- SNS 주제 지정 : SNS 주제에서 선택
- SNS 주제 : S3Events

![image](https://user-images.githubusercontent.com/77096463/109901467-7c00ab00-7cdc-11eb-85f3-cf00f9fc2215.png)

<br>

### Modify the SNS Topic Policy

SNS > 생성한 토픽 (S3Events) 클릭 >  액세스 정책 변경

- 4번째 줄의 `"Statement" :[` 다음에 다음의 코드 추가
- `SNS_ARN_REPLACE_ME` : S3의 ARN 으로 변경
- `S3_BUCKET_ARN_REPLACEME` : SNS의 ARN 으로 변경

```
{
      "Effect": "Allow",
      "Principal": {
          "AWS": "*"
  },
      "Action": "SNS:Publish",
      "Resource": "SNS_ARN_REPLACE_ME",
      "Condition": {
          "StringEquals": {
              "aws:SourceArn": "S3_BUCKET_ARN_REPLACEME"
          }
      }
  },
```

아래와 같이 코드 변경하기 

![image](https://user-images.githubusercontent.com/77096463/109901784-f4676c00-7cdc-11eb-887d-caf69c5638df.png)

<br>

### Configure the Bucket — Part 2

Configure the Bucket — Part 1의 과정을 다시 반복하면 이번에는 성공적으로 이벤트 알림을 설정할 수 있다. 

![image](https://user-images.githubusercontent.com/77096463/109902067-3e505200-7cdd-11eb-85fc-9d4f07477443.png)

<br>

### Create the Email Subscription

SNS > 구독 > 구독 생성

- 주제 ARN : 드롭다운 메뉴에서 선택
- 프로토콜 : 이메일
- 엔드포인트 : 본인 이메일 입력

![image](https://user-images.githubusercontent.com/77096463/109902193-72c40e00-7cdd-11eb-8564-bbea4eb8e31b.png)

<br>

confirmation 메일 확인

![image](https://user-images.githubusercontent.com/77096463/109902242-88d1ce80-7cdd-11eb-9171-23582cfaed22.png)

<br>

이메일 엔드포인트가 등록되었음을 최종 확인

![image](https://user-images.githubusercontent.com/77096463/109902268-912a0980-7cdd-11eb-9564-0c059b0a8531.png)

<br>

### Create the SMS Subscription

SNS > 주제 > S3Events > 메시지 게시

- 제목 :  HELLO
- 메시지 본문 : This is a test message

![image](https://user-images.githubusercontent.com/77096463/109902361-b585e600-7cdd-11eb-8ca6-4287fec2e4f6.png)

<br>

메시지를 게시하면 등록된 이메일 엔드포인트로 테스트 메시지가 왔음을 확인

![image](https://user-images.githubusercontent.com/77096463/109902399-c20a3e80-7cdd-11eb-8934-694152d776c1.png)

<br>

S3 > 생성한 버킷 (sns-notifications-s3-test-errol) > 파일 업로드 

![image](https://user-images.githubusercontent.com/77096463/109902558-f7169100-7cdd-11eb-88e4-a71178e1ecfa.png)

<br>

S3 버킷에 새로운 파일이 업로드되면 이메일로 알림 메시지가 전송됨을 알 수 있다.

![image](https://user-images.githubusercontent.com/77096463/109903002-8f147a80-7cde-11eb-915c-2180cbe73e4a.png)

<br>

### 메시지 로그 남기는 람다 함수 생성 후 확인

Lambda 함수를 생성하여 코드를 아래와 같이 수정 후, SNS 토픽에 구독 생성

![image](https://user-images.githubusercontent.com/77096463/109903637-5fb23d80-7cdf-11eb-85ee-6b7dc692270c.png)

<br>

구독 생성 후, 메시지 게시한 후 이메일 확인

![image](https://user-images.githubusercontent.com/77096463/109903670-6b056900-7cdf-11eb-9360-1d6cc6a64aa8.png)

<br>

CloudWatch 로그 확인하면 메시지가 제대로 남아있음을 확인

![image](https://user-images.githubusercontent.com/77096463/109903691-722c7700-7cdf-11eb-94ef-3b12bb5c7d8f.png)

<br>

### S3에 추가한 오브젝트 이름 출력하는 람다 함수

s3에 오브젝트를 추가했을 때 출력되는 모양은 아래와 같이 json editor를 활용하거나, <br>
코드소스 > 테스트 메뉴를 통해 확인 가능 

![image](https://user-images.githubusercontent.com/77096463/109904808-27136380-7ce1-11eb-8c9e-66f59bdf015d.png)

<br>

![image](https://user-images.githubusercontent.com/77096463/109913181-d0ae2100-7cf0-11eb-9b89-cb072d1676ab.png)

<br>

s3에 추가된 객체의 이름을 받아오기 위해 아래와 같이 코딩

![image](https://user-images.githubusercontent.com/77096463/109904846-372b4300-7ce1-11eb-8deb-009742dca08b.png)

<br>

S3 > 이벤트 알림을 등록해준다 (**대상 유형: Lambda 함수**)

![image](https://user-images.githubusercontent.com/77096463/109913916-32bb5600-7cf2-11eb-88e7-7504ea67076a.png)

<br>

s3에 210226.PNG 파일을 업로드 한 후 로그 이벤트를 확인하면 아래와 같이 출력됨을 확인 가능

![image](https://user-images.githubusercontent.com/77096463/109913838-07d10200-7cf2-11eb-9980-26ee2a9f8df2.png)

<br>

### S3 버킷에 파일이 업로드되었을 때 SMS 문자가 발송되도록 람다 함수를 수정

Lambda 함수의 코드 수정 (파일 이름 + SMS 문자 발송되는 코드)

![image](https://user-images.githubusercontent.com/77096463/109914750-fa1c7c00-7cf3-11eb-9ef5-de1913e9dada.png)

<br>

코드 수정 후 곧바로 S3에 파일 추가 후 로그를 보면 에러 발생 -> **람다 함수는 그러한 권한이 없기 때문**<br>따라서 람다 함수의 실행 역할로 이동

![image](https://user-images.githubusercontent.com/77096463/109914459-5e8b0b80-7cf3-11eb-9c08-66c2d8394103.png)

<br>

람다 함수에 권한을 부여하기 위해 정책 연결 -> 'SNS' 검색 후 Configuring_SNS_Push_Notifications~' 정책 연결

![image](https://user-images.githubusercontent.com/77096463/109914440-5632d080-7cf3-11eb-86e0-d957034f9f91.png)

<br>

람다 함수에 정책 연결 후 다시 앞의 과정을 반복하면 이번엔 로그가 제대로 출력되고 핸드폰으로도 메시지가 온다.

![image](https://user-images.githubusercontent.com/77096463/109914824-1ddfc200-7cf4-11eb-8447-c4d313c72241.png)

<br>

<br>

# **Working with AWS SQS Standard Queues**

**Queue**

- 메시지를 담는 공간
- 리전 별로 생성해야 함
- HTTP 프로토콜을 이용해서 다른 리전끼리 메세지를 주고 받을 수 있음
- 큐 이름은 모든 리전에서 유일해야 함
- 큐에 담을 수 있는 메시지의 개수는 무제한
- 연속해서 30일 동안 아무런 요청이 발생하지 않으면 AWS가 큐 삭제
- 같은 리전 안에서 큐를 이용한 데이터 전송은 무료
- 다른 리전에 있는 인스턴스와 메시지를 주고 받으면 데이터 요금이 과금

<br>

**Message** 

- XML 또는 JSON 형태
- 최대 256 GB
- 유니코드 문자 사용 가능
- 초 단위로 보관 기간 설정 가능
- 고유한 아이디가 부여됨
- 3~4KB의 메시지도 256KB로 책정 -> 용량이 작은 메시지를 개별 처리하는 것 보다는 메시지를 모아서 배치 API로 처리하는 것이 비용 효율적
  - Batch API : 한번에 최대 10개의 메시지 (최대 256KB) 동시에 처리

<br>

![image](https://user-images.githubusercontent.com/77096463/109920090-2092e500-7cfd-11eb-9632-4e9bea8b8052.png)

**Visibility Timeout** 

- 메시지를 받은 뒤 특정 시간 동안 다른 곳에서 동일한 메시지를 다시 꺼내 볼 수 있게 혹은 없게 하는 기능
- 큐 하나에 여러 서버의 메시지를 받을 때 동일한 메시지를 동시에 처리하는 것을 방지
- Message Available (Visible) -> 메시지를 꺼내서 볼 수 있는 형태
- Message in Fight (Not visible) -> 다른 곳에서 메시지를 보고 있어 현재 볼 수 없는 메시지의 개수

![image](https://user-images.githubusercontent.com/77096463/109921174-c6931f00-7cfe-11eb-8d90-5de90d553ebe.png)

<br>

**Delay Delivery** - 모두가 Access 불가

- 특정 시간 동안 메시지를 받지 못하게 하는 기능
- 지연되는 시간 동안 Message in Fight에 포함

<br>

**Dead Letter Queues** 

- 일반적으로 메시지를 받고 작업이 완료되면 메시지를 삭제하는데, 설정한 횟수를 초과하여 메시지를 받았는데 삭제되지 않고 남아있으면 처리 실패 큐 보냄
- 일반 큐 하나에 여러 처리 실패 큐 연결 가능, 일반 큐와 처리 실패 큐는 동일한 리전에 있어야 함

<br>

**Short Polling**

- 메시지 받기 요청을 하면 바로 결과를 받음
- 메시지가 있다면 메시지를 가져오고, 없으면 빠져 나옴
- ReceiveMessage 요청해서 WaitTimeSeconds를 0으로 했을 때
- Queue 설정의 ReceiveMessageWaitTimeSeconds를 0으로 했을 때 

<br>

**Long Polling**

- 메시지가 있으면 바로 가져오고 없으면 올 때까지 기다림
- 1초부터 20초까지 설정이 가능 (기본은 20초)
- ReceiveMessage 요청에서 WaitTimeSeconds를 0 보다 크면 큐 설정의 ReceiveMessageWaitTimeSeconds 값 보다 우선으로 처리

  <<<<<<< Updated upstream:AWS/210304_Messaging_in_AWS.md
  =======

<br>

### Lab 구성도

![image](https://user-images.githubusercontent.com/77096463/109923886-c85ee180-7d02-11eb-83ef-3293be6545cf.png)

<br>

### Create a Standard SQS Queue (First terminal)

터미널 접속 후 파일 목록 확인

```
[cloud_user@ip-10-0-1-10 ~]$ ls
create_queue.py   fast_data.json    purge_queue.py   slow_consumer.py  slow_producer.py
fast_consumer.py  fast_producer.py  queue_status.py  slow_data.json    sqs_url.py
```

<br>

create_queue.py 파일 실행 -> 표준 SQS 큐 생성

```
[cloud_user@ip-10-0-1-10 ~]$ python3 create_queue.py
https://queue.amazonaws.com/188564947610/mynewq
```

![image](https://user-images.githubusercontent.com/77096463/109924668-fb55a500-7d03-11eb-94a1-31b816236114.png)

<br>

https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sqs.html create_queue참고

```
[cloud_user@ip-10-0-1-10 ~]$ cat create_queue.py
import boto3

# Create SQS client
sqs = boto3.client('sqs')

# Create a SQS queue
response = sqs.create_queue(
    QueueName='mynewq',
    Attributes={
        'DelaySeconds': '5',
        'MessageRetentionPeriod': '86400'	#24시간
    }
)
```

<br>

생성된 큐 URL을 sqs_url.py 파일에 등록 -> 다른 애플리케이션에서 참조하기 위해서 사용

```
[cloud_user@ip-10-0-1-10 ~]$ sudo vi sqs_url.py
[cloud_user@ip-10-0-1-10 ~]$ cat sqs_url.py
QUEUE_URL = 'https://queue.amazonaws.com/188564947610/mynewq'
```

<br>

### Monitor the Queue (Second Terminal)

https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sqs.html get_queue_attributes 참고

```python
# queue_status.py
import boto3
import time
from sqs_url import QUEUE_URL

sqs = boto3.client('sqs')

i = 0

while i < 100000:
    i = i + 1
    time.sleep(1)
    response = sqs.get_queue_attributes(
        QueueUrl=QUEUE_URL,
        AttributeNames=[
            'ApproximateNumberOfMessages',
            'ApproximateNumberOfMessagesNotVisible',
            'ApproximateNumberOfMessagesDelayed',
        ]
    )
    for attribute in response['Attributes']:
        print(
            response['Attributes'][attribute] +
            ' ' +
            attribute
        )
    print('')
    print('')
    print('')
    print('')
```

<br>

queue_status.py 파일 실행 -> 생성한 큐가 어떤 상태를 보이는지 모니터링

```
[cloud_user@ip-10-0-1-23 ~]$ python3 queue_status.py
0 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
0 ApproximateNumberOfMessagesDelayed

0 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
0 ApproximateNumberOfMessagesDelayed
```

<br>

### Send Data (Third Terminal)

https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sqs.html send_message 참고

```python
# slow_producer.py
import boto3
import json
import time
from sqs_url import QUEUE_URL

# Create SQS client
sqs = boto3.client('sqs')

with open('slow_data.json', 'r') as f:
    data = json.loads(f.read())

for i in data:
    msg_body = json.dumps(i)
    response = sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=msg_body,
        DelaySeconds=10,
        MessageAttributes={
            'JobType': {
                'DataType': 'String',
                'StringValue': 'NewDonor'
            },
            'Producer': {
                'DataType': 'String',
                'StringValue': 'Slow'
            }
        }
    )
    print('Added a message with 10 second delay - SLOW')
    print(response)
    time.sleep(1)
```
slow_producer.py 파일 실행
```
[cloud_user@ip-10-0-1-10 ~]$ python3 slow_producer.py
Added a message with 10 second delay - SLOW
{'MD5OfMessageBody': '65481eeda1e7e1e059481a4ffb2aae3f', 'MD5OfMessageAttributes': '3d69062df9b93571f431c178b5c5ee60', 'MessageId': 'f62f3b5c-e3a4-43f0-9161-119c8f5f91bc', 'ResponseMetadata': {'RequestId': 'd21e4e52-3465-52fe-8a58-16e8ba2aafba', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'd21e4e52-3465-52fe-8a58-16e8ba2aafba', 'date': 'Thu, 04 Mar 2021 07:28:16 GMT', 'content-type': 'text/xml', 'content-length': '459'}, 'RetryAttempts': 0}}
Added a message with 10 second delay - SLOW
{'MD5OfMessageBody': '4edb012a279423a124f8d618689c0027', 'MD5OfMessageAttributes': '3d69062df9b93571f431c178b5c5ee60', 'MessageId': '25fa96cc-d5fc-4f9a-8f57-147cb4016c1b', 'ResponseMetadata': {'RequestId': '2625d987-d068-57c5-b36c-813d5e399a81', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '2625d987-d068-57c5-b36c-813d5e399a81', 'date': 'Thu, 04 Mar 2021 07:28:17 GMT', 'content-type': 'text/xml', 'content-length': '459'}, 'RetryAttempts': 0}}
```

```
0 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
8 ApproximateNumberOfMessagesDelayed




0 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
6 ApproximateNumberOfMessagesDelayed

...

9 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
9 ApproximateNumberOfMessagesDelayed




15 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
9 ApproximateNumberOfMessagesDelayed




16 ApproximateNumberOfMessages
0 ApproximateNumberOfMessagesNotVisible
9 ApproximateNumberOfMessagesDelayed
...
```

<br>

fast_producer.py 파일 실행















### Receive Messages and Extract Metadata (Fourth and Fifth Terminals)

https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sqs.html receive_message, delete_message 참고

```python
# fast_consumer.py
import boto3
import json
import time
from sqs_url import QUEUE_URL

# Create SQS client
sqs = boto3.client('sqs')

i = 0

while i < 10000:
    i = i + 1
    rec_res = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MessageAttributeNames=[
            'All',
        ],
        MaxNumberOfMessages=1,
        VisibilityTimeout=5,
        WaitTimeSeconds=10
    )
    del_res = sqs.delete_message(
        QueueUrl=QUEUE_URL,
        ReceiptHandle=rec_res['Messages'][0]['ReceiptHandle']
    )
    print("RECIEVED MESSAGE (FAST CONSUMER):")
    print('FROM PRODUCER: ' + rec_res['Messages'][0]['MessageAttributes']['Producer']['StringValue'])
    print('JOB TYPE: ' + rec_res['Messages'][0]['MessageAttributes']['JobType']['StringValue'])
    print('BODY: ' + rec_res['Messages'][0]['Body'])
    print("DELETED MESSAGE (FAST CONSUMER)")
    print("")
    time.sleep(2)
```

```python
# slow_consumer.py
import boto3
import json
import time
from sqs_url import QUEUE_URL

# Create SQS client
sqs = boto3.client('sqs')

i = 0

while i < 10000:
    i = i + 1
    rec_res = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MessageAttributeNames=[
            'All',
        ],
        MaxNumberOfMessages=1,
        VisibilityTimeout=20,
        WaitTimeSeconds=10
    )
    del_res = sqs.delete_message(
        QueueUrl=QUEUE_URL,
        ReceiptHandle=rec_res['Messages'][0]['ReceiptHandle']
    )
    print("RECIEVED MESSAGE (SLOW CONSUMER):")
    print('FROM PRODUCER: ' + rec_res['Messages'][0]['MessageAttributes']['Producer']['StringValue'])
    print('JOB TYPE: ' + rec_res['Messages'][0]['MessageAttributes']['JobType']['StringValue'])
    print('BODY: ' + rec_res['Messages'][0]['Body'])
    print("DELETED MESSAGE (SLOW CONSUMER)")
    print("")
    time.sleep(8)
```

<br>

<br>

# 

https://aws.amazon.com/ko/sqs/features/

![image](https://user-images.githubusercontent.com/77096463/109931976-ef221580-7d0c-11eb-9e91-6822d8823dce.png)
