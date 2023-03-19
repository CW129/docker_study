# Multi Container Application 

> ###  **목표** 
> docker network와 volume을 활용해 다중 어플리케이션을 생성하고 효율적인 개발환경을 설정해 보는 것

<br />

## 1. 어플리케이션 구성 및 개발환경 
<br />

### 1) Database(mongdb)
* 컨테이너 내부의 데이터 지속(mongodb 컨테이너 삭제해도 data는 유지)
* 보안 설정(엑세스 제한)

### 2) Backend(node.js 기반 Rest API)
* 컨테이너 내부의 데이터 지속(로그 파일)
* 소스 코드 변경사항 즉시 반영

### 3) Frontend(react)
* 소스 코드 변경사항 즉시 반영


<br />

##  2. 동일한 네트워크 환경 구성
<br />

```
docker network create ~networkname~
```

### 1) Database(mongdb)
: mongodb 컨테이너를 실행할 때 mongodb 이미지 실행을 위해 사용해야 했던 포트(27017)을 제시할 필요 없이 생성한 네트워크를 통해 실행시킨다.

### 2) Backend(node.js 기반 Rest API)
* 컨테이너에서 mongodb와 통신할 수 있도록 사용한 ```host.docker.internal```이 아닌 생성한 네트워크에 연결된 mongodb 컨테이너 이름을 사용
  * mongodb 컨테이너를 실행할 때 mongodb에서 제안한 포트를 사용하지 않았기 때문
* backend 컨테이너를 실행할 떄 생성한 네트워크를 추가해 실행시킨다.
* 또한 frontend와의 연결을 위해 backend에 연결될 포트 또한 명령어에 추가해야 한다.

### 3) Frontend(react)
* frontend는 backend api와 상호작용하는 부분에서 도커 컨테이너 환경이 아닌 브라우저에서 실행되기 때문에 브라우저가 컴파일 할 수 있는 backend 주소를 사용해야 한다.
* 생성한 네트워크를 명령어에 추가할 필요없다.


<br />

##  3. 컨테이너 데이터 지속성
<br />

### 1) Database(mongdb)
* 컨테이너를 삭제 후 다시 실행시키면 기존에 저장되었던 데이터 또한 삭제, 볼륨을 사용해 데이터를 컨테이너와 분리해 하드 드라이버에 저장

* mongodb 이미지의 내부 경로를 hub에서 확인해 이름을 가진 볼륨을 생성
  * [docker hub - mongo](https://hub.docker.com/_/mongo)

### 2) Backend(node.js 기반 Rest API)
* log 파일을 지속적으로 저장할 이름을 가진 볼륨을 생성
* 바인드 마운트으로 소스 코드 전체에서 변경된 사항을 바로 확인
  * nodemon을 사용

> 보안과 엑세스 방지를 위해 환경변수 적용 <br />
> backend에서 mongodb를 연결하는 코드에 ```username:password@mongodbnmae```과 문자열 끝에 ```authSource=admin```을 추가 <br />
또한 Dockerfile에 ENV로 username과 password 환경변수를 작성해준다.<br />
(출처 : docker hub - mongo)

### 3) Frontend(react)
* 바인드 마운트으로 소스 코드 전체에서 변경된 사항을 바로 확인
* 도커 실행 과정에서 package.json을 실행하고 COPY하는 과정에서 똑같은 node_modules 파일을 덮어쓰는데 dockerignore에 node_modules을 작성해 COPY과정에서 제외시킨다.
  * 같은 동작을 반복 할 뿐만 아니라 오래된 코드를 COPY할 수 있기 때문


