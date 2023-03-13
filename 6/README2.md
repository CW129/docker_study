# Docker-compose

![https://user-images.githubusercontent.com/65060314/221388327-5e3d1d27-af4c-46c7-988e-7e3c50bca3de.png](https://user-images.githubusercontent.com/65060314/221388327-5e3d1d27-af4c-46c7-988e-7e3c50bca3de.png)

## docker-compose 란 무엇일까?

docker-compose는 앞서 배운 docker build와 docker run을 과정을 하나의 manifest 파일을 통해 정의하여 ‘docker-compose up’ 명령어를 통해 일련을 과정을 자동화 해주는 모듈입니다 
**(하지만 모든 docker 명령어의 기능을 제공해 주는 것은 아님)**

**특징**

→ 여러 컨테이너를 관리하기에 쉬워집니다

→ source code처럼 형상관리에 용이해짐(docker 기능자체에서 형상관리를 한다는 의미는 아님 git이나 형상관리 프로그램에 의해 관리가 될 수 있는 항목이 된다는 점에서 작성)

→ container의 특정 변경사항이 존재하지 않을 경우 container 유지 (반대로 변경 사항이 존재하는 service만 recreating)

### 작성 방법

1. docker-compose 모듈에서 해당 manifest 파일을 파악하기위한 version 정보를 입력
2. container service 정의 (ex[monogo db , backend, frontend])
3. 해당 sercices들에서 사용할 자원 정의 (names volume, network 등)

```yaml
#docker-compose 모듈의 버전 정보를 기입
version: "3.8"
# container로 서비스될 항목들을 정의
services:
  mongodb:
		# docker hub 또는 local에서 이미지를 참조하게될 경우 이미지의 이름을 기입 
		# (*이미지 참조 우선도: 입력 받은 이미지 이름으로 로컬 환경에서 확인 후 docker hub를 통해 검색하여 pull)
    image: 'mongo'
		# docker named volume을 마운트 하는 설정 
    volumes: 
      - data:/data/db
    # environment: 
    #   MONGO_INITDB_ROOT_USERNAME: max
    #   MONGO_INITDB_ROOT_PASSWORD: secret
      # - MONGO_INITDB_ROOT_USERNAME=max
		# 환경변수를 정의한 파일을 참조하여 container 생성
    env_file: 
      - ./env/mongo.env
  backend:
		#build는 해당 container 생성 시 정의되어진 dockerfile을 통해 docker image를 생성 후 container 생성
    build: ./backend
    # build:
    #   context: ./backend
    #   dockerfile: Dockerfile
    #   args:
    #     some-arg: 1
	
		# port 바인딩 정보
    ports:
      - '80:80'

		#volume 마운트 정보
    volumes: 
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules

    env_file: 
      - ./env/backend.env
    depends_on:
      - mongodb

  frontend:
    build: ./frontend
    ports: 
      - '3000:3000'
    volumes: 
      - ./frontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on: 
      - backend

volumes: 
  data:
  logs:
```

> ‘ **services.{service}.build** ’ 와 ‘ **services.{service}.image ’**의 차이
앞의 정의는 docker-compose를 통해 container 생성 시 image를 build 하여 build된 이미지를 사용하겠다는 것이며 image는 기존의 생성되어진 이미지를 참고하여 사용하겠다는 뜻
> 

## Work Load

- workload : airflow 환경에서 component 별로 격리 시킬때 dag 파일에 대한 참조를 어떻게 해야하는가?
    
    → volume bind mount를 통해 같은 파일을 보게 할 것인가? 파일을 component 모두가 참조하여 image의 용량을 늘릴 것인가?
    
    → 공통화된 모듈은 같은 이미지파일로 격리하는것이?
    

![https://user-images.githubusercontent.com/65060314/221388928-00c686f0-09f1-43c5-be5c-52807db8ded3.png](https://user-images.githubusercontent.com/65060314/221388928-00c686f0-09f1-43c5-be5c-52807db8ded3.png)

## Potal Frontend

- frontend는 이미지 생성 시 project 디렉토리에 있는 ‘node_module’을 복사하는가? (image build 시 참조할 파일과 제외할 파일에대한 분리가 확실하게 필요)

![https://user-images.githubusercontent.com/65060314/221389183-f6de9988-5b21-4698-bec2-f55f16f36c0f.png](https://user-images.githubusercontent.com/65060314/221389183-f6de9988-5b21-4698-bec2-f55f16f36c0f.png)

- build 과정에서 왜 env 파일을 무조건 적으로 만들어내는가? (file io는 os에서 자원을 할당 받아 진행됨에 따라 많은 자원소모가 이루어짐…이미 존재하는 파일을 무조건적으로 다시 만드는 행위는 불필요하다 판단)