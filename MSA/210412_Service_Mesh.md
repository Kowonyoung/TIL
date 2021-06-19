### 1. Microservice Architecture

Cloud Native Application을 작동시키는 환경이 Cloud Native Architecture<br>
클라우드 서비스는 가상화가 기본이며, 그런 가상화를 다루는 게 쿠버네티스<br>
**Service Mesh**는 그 중에서도 핵심! <br>
Service Router / Load Balancing : 사용자가 어디로 갈 지 방향을 잡아줌<br>
Service Discovery (=Service Registry) : 각각의 서비스가 무엇인지, 작업을 위해 어떤 서비스를 실행해야 하는지 등의 서비스와 관련된 정보를 담당<br>
- Service Discovery 역할을 하는 sw가 Service Registry 역할도 하며, 둘 다 동일한 작업 실행
- 서비스가 등록되어야(Service Registry) 서비스 관련 작업을 수행(Service Discovery)할 수 있음

Config.Store : 시스템에서 사용될 수 있는 문자열 데이터 등을 보관하는 외부 스토리지 (하드코딩 X)

<br>

![image](https://user-images.githubusercontent.com/77096463/114326933-078a1900-9b72-11eb-917b-534bc8dc5a99.png)

CLI/API : 클라이언트에 의해 발생하는 요청 (curl, RESTful 등) <br>
컨트롤 플레인 : 데이터 플레인 관리<br>
데이터 플레인 : 실제 데이터를 넘겨 받아 작업을 처리<br>
각각의 인스턴스는 독립적으로 실행 > 즉, 모두 다른 ip를 가지고 있음

<br>

**Service Mesh의 종류**

![image](https://user-images.githubusercontent.com/77096463/114327275-57b5ab00-9b73-11eb-882b-fd2f6ec9df97.png)

<br>

**Service Discovery**

![image](https://user-images.githubusercontent.com/77096463/114327467-212c6000-9b74-11eb-9e44-bc9e60df644f.png)

각 서비스가 자신의 서비스를 Service Discovery에 등록<br>
클라이언트의 요청이 들어오면 Load Balancer가 요청을 어디로 보낼지 결정

<br>

**Consul**

HashiCorp Consul (vagrant 만든 회사)

- Distributed Architecture
- Service Discovery
- Health Checking
- Key/Value Store
- Secure Communication

가능하면 linux 혹은 MacOS에서 진행하는 걸 권장

<br>

구성

- Consul Server (Control Plane)
- Consul Agent (Data Plane)

![image](https://user-images.githubusercontent.com/77096463/114327607-b4fe2c00-9b74-11eb-91b0-b8a22a8201d6.png)

