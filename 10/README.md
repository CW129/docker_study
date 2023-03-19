# Docker

---


## Docker란?

### 개념

![https://user-images.githubusercontent.com/65060314/226149379-04612705-f68b-4f21-9fd4-75d70e32d355.png](https://user-images.githubusercontent.com/65060314/226149379-04612705-f68b-4f21-9fd4-75d70e32d355.png)

Docker는 **container**라는 기능을 통해 어플리케이션(Application)을 같은 OS 상에서 **격리된 형태**로 서비스를 제공해 줄 수 있도록 하는 서비스입니다 

<aside>
💡 Container 란?
[Docker 기술 블로그](https://www.docker.com/blog/getting-started-with-docker-desktop/) 에서 ‘docker는 무엇인가요?’라는 질문을 했을 떄 ‘container’를 먼저 말할 정도로 docker의 핵심기술이라고 할 수 있습니다 `container란 source code와 해당 프로젝트와 관계된 모든 프로그램들의 패키지를 의미하며 해당 패키지는 하나의 컴퓨팅 시스템 환경(하나의 OS 시스템)에서 독립된 형태로 서비스를 제공`하도록 지원합니다

container의 상세기능은 [IBM-’Waht is contaienr’](https://www.ibm.com/topics/containers) 링크를 확인하시길….

</aside>

### Docker를 무엇을 할 수 있을까?

- 소프트웨어 프로토타이핑 및 패키징 : 프로젝트에 대한 패키징을 `image`라는 기능을 통해 구현할 수 있음
- 마이크로서비스 아키텍처 구현 : container를 통한 독립적인 공간을 확보하며 `container 생성 삭제의 유연성`을 통해 MSA 구현
- 네트워크 모델링 : `namespace`를 통한 격리 네트워크 생성을 통한 자유로운 네트워크 모델링 가능
- 지속적인 통합 및 제공 : image 기능의 `레이아웃 기능`을 통한 지속적인 통합 제공
- 디버깅 오버헤드 줄이기 :
- 동일한 하드웨어에서 더 많은 워크로드 실행 : `같은 OS 내에 다른 서비스를 제공`하여 더 많은 하드웨어 시스템을 효율적으로 사용할 수 있음

---

## Docker 기능

### Image란?

‘**서비스를 제공하기위한 최소한의 내용들이 패키징 되어진 통합 디렉토리’**

이미지는 여러 레이아웃을 통해 하나의 사용자 이미지로 만들어 지게 됩니다 기본적인 ‘Base Image’ 아래에 Dockerfile에서 명시되어진 한줄 한줄이 ‘Read Only Layer’라는 레이아웃을 통해 합쳐지게 됩니다 
사용자에 이해 container 내에 정보가 변경 될 경우를 대비하여 ‘Write Layer’를 따로 두어 관리합니다

![https://user-images.githubusercontent.com/65060314/226150172-29340b30-4264-4baf-9934-34e9a883372f.png](https://user-images.githubusercontent.com/65060314/226150172-29340b30-4264-4baf-9934-34e9a883372f.png)

- dockerfile 작성 방법과 명령어

dockerfile의 작성은 기본이 되는 base image 레이어를 ‘FROM’에 명시한 뒤 사용자에 의해 추가할 레이어가 명시되어집니다 한줄한줄 명시되어진 내용은 각각 하나의 레이어로 구성되어지며 모든 사용자 명시가 완료되었을떄 하나의 docker image로서 사용할 수 있습니다

|  | Description |
| --- | --- |
| FROM | 후속 명령을 위해 https://docs.docker.com/glossary/#base-image |
| ENV | container내의 환경변수 지정 |
| ARG | 이미지 빌드 시 값을 입력받아 변수로서 사용 |
| EXPOSE | Container의 기본 서비스 포트 지정 |
| CMD | Container 생성 시 사용되는 명령어  |
| RUN | 명령 RUN은 현재 이미지 위에 있는 새 레이어에서 모든 명령을 실행하고 결과를 커밋합니다 |
| ENTRYPOINT | container 최초 생성 시 무조건 실행되는 명령어 |
- 이미지 만들기

dockerfile 작성 : dockerfile 내용을 규격에 맞게 작성합니다

```bash
FROM python:3.11.1-slim

ENV PYTHONUNBUFFERED 1
ENV TZ=Asia/Seoul

EXPOSE 8000
WORKDIR /home/cucuridas/chatbot_tg

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get install vim curl build-essential -y && \
    apt-get install -y --no-install-recommends netcat && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    pip install poetry && poetry config virtualenvs.in-project true 

#COPY pyproject.toml ./
COPY . ./
RUN poetry update
#poetry update --no-dev # 빌드 후 젠킨스에서 test 수행하기 위해 모두 설치

CMD poetry run alembic upgrade head && \
    poetry run python3 ./app/main.py
#    poetry run gunicorn --bind 0.0.0.0:8000 -w 1 --timeout 120 -k uvicorn.workers.UvicornWorker app.core.server:app
# poetry run uvicorn app.core.server:app --host 0.0.0.0 --port 8000 --workers 1
```

docker 명령어를 통해 이미지 생성 - dockerfile 이 존재하는 디렉토리에서 아래와 같은 명령어를 실행합니다

```bash
docker build -t test .
```

이미지의 레이어 형태가 궁금하다면? 

[내가 생성한 이미지 레이어는 어따구로 생겼는가?….](Docker%20a775c5b2abb549929e129cd14a9ca9f8/%E1%84%82%E1%85%A2%E1%84%80%E1%85%A1%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5%20%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%8B%E1%85%A5%E1%84%84%E1%85%A1%E1%84%80%E1%85%AE%E1%84%85%E1%85%A9%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%80%E1%85%A7%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20f83b7c088fc3497f9dc15624444d76af.md)

🤔사용자가 변경한 레이어는 어디에?

참조자료 : [Docker-Image에 대한 고찰](https://phantasmicmeans.tistory.com/entry/Docker-Container-Image-%EA%B3%A0%EC%B0%B0) 

---

### Volume

Container Image를 통해 생성된 이미지의 저장소는  container 삭제와 함께 없어지는 휘발성 저장소입니다 

Docker는 이러한 점을 보완해주기위해 Docker volume이라는 기능을 통해 container가 종료되거나 삭제되어도 저장소를 안전하게 유지시켜주는 기능을 제공합니다

- volume의 종류

아래의 이미지는 docker document에서 제공되는 이미지입니다 제공되는 mount 기능으로서 `bind, volume tmpfs` 등을 지원합니다

![https://user-images.githubusercontent.com/65060314/226150821-97a3be81-40b7-4610-9ed0-2d3e1a6088f6.png](https://user-images.githubusercontent.com/65060314/226150821-97a3be81-40b7-4610-9ed0-2d3e1a6088f6.png)

<aside>
💡 bind : 사용자가 지정한 특정 경로를 container 내부의 특정 디렉토리와 연결합니다 (해당 디렉토리는 권한은 원본 디렉토리의 권한을 따라지정됩니다)
volume : ‘docker volume create’ 명령어를 통해 생성되어진 volume을 container 내부의 특정 디렉토리와 연결합니다 (해당 디렉토리의 권한은 docker group의 user가 가지게 됩니다)
tmpfs : 메모리에 바인딩되어 container 삭제시 같이 삭제되는 volume입니다

</aside>

- volume mount 방법
    
    명령어를 통해 volume을 마운트 하기
    
    ```bash
    # bind mount
    docker run -d \
      -it \
      --name devtest \
      --mount type=bind,source="$(pwd)"/target,target=/app \
      nginx:latest
    
    # volume mount
    # volume 생성
    docker volume create test
    # volume mount
    docker run -d \
      --name devtest \
      --mount source=test,target=/app \
      nginx:latest
    ```
    
    🤔익명 볼륨이 뭘까요?
    

---

### Network

Docker 네트워크의 기능은 container 끼리의 통신 또는 os 시스템에서 전달받은 network 환경을 container와 연결해주기 위한 기능을 제공합니다

- Docker Network의 종류

| Type | Description |
| --- | --- |
| bridge | Host의 커널 하드웨어 혹은 소프트웨어 디바이스를 사용 |
| host | Docker daemon의 인터페이스(가상) 을 거치지 않고 호스트의 인터페이스에 직접 통신 (Production 환경에서 잘 사용하지 않을듯?) |
| overlay | swarm 에서 사용할 수 있는 Network Type, 분산 네트워킹을 위한 기술로 도커 클러스터 구성할 때 필요 |
| ipvlan | Linux kernel 기술로 하나의 물리 인터페이스에 여러 가상 인터페이스를 만듬 |
| macvlan | 하나의 인터페이스에 여러개의 Mac주소를 가지는 네트워크 인터페이스 |
| none | 단어 그대로 네트워크를 연결하지 않음을 의미 |
- Docker Network 생성 및 연결 방법

```bash
# bridge
docker run --rm -d --network bridge --name my_nginx nginx
# host
docker run --rm -d --network host --name my_nginx nginx
# overlay
docker network create -d overlay my-overlay #생성
docker run --rm -d --network my-overlay --name my_nginx nginx

```

🤔 ipvaln, macvaln에 대한 정보는 링크를 참조하세요!

[ipvaln](https://docs.docker.com/network/ipvlan/) , [macvlan](https://docs.docker.com/network/macvlan/), [overlay](https://docs.docker.com/network/overlay/)

---

## Docker-compose

[Docker Compose](https://docs.docker.com/compose/) 는 다중 컨테이너 애플리케이션을 정의하고 공유하는 데 도움이 되도록 개발된 도구입니다. Compose를 사용하면 YAML 파일을 생성하여 서비스를 정의하고 단일 명령으로 모든 것을 돌리거나 모두 해체할 수 있습니다.

 YAML 파일에 작성되는 내용은 앞서 설명한 ‘docker 기능’ 들에 대한 명세가 작성되어집니다 

- docker-compose.yaml 파일 작성 방법
    
    파일 작성 방법은 아래와 같은 포맷을 가지게됩니다
    
    ```bash
    version: "3.8"
    
    services: 
    	service_name:
    		image: test_image
      ...
    	...
    	...
    volumes:
    	...
    	...
    network:
    	...
    ```
    
    services 하위 목록으로 정의되어지는 내용은 continaer들의 세부사항이 정의되어지게 됩니다 container의 이름을 어떻게 지정할 것인지 container 내부 환경변수 설정은 어떻게 할 것인지 참조되는 base 이미지는 어떻게 지정될 것인지 사용하는 volume은 어떤 volume을 마운트 시키는지 등 container에 대한 전반적인 설정이 service 목록에 정의 되어지게 됩니다
    
    <aside>
    💡 docker-compose의 version의 의미
    YAML 파일 내부에서 정의되어지는 version의 의미는 해당 파일의 형상관리에 대한 버전이 아닌 YAML파일을 해석하기위한 docker-compose 모듈의 version을 의미합니다
    
    </aside>
    
    [ex] chatbot_tg 이미지를 통해 container 생성
    
    ```bash
    version: '3' # YAML 파일 해석을 담당하는 모듈의 버전정보를 정의
    services: # 서비스의 모음들이 정의되는 구역
      app: # 서비스 이름
        image: chatbot:latest # continaer 생성할 image 이름
        restart: on-failure # 재시작 설정
        container_name: chatbot # container 이름
        ports: # container port 정보와 host의 port정보를 연결 (dockerfile 파일에서 정의한 EXPOSE를 정보)
          - "8000:8000"
        env_file: # 참조할 env(환경 변수)파일 정의
          - .env
        environment: # 환경변수 값 따로 지정
          TZ: Asia/Seoul
        volumes: # volume 정보 마운트
          - ./schedule_log:/home/cucuridas/chatbot_tg/schedule_log
          - $WORK_REPORT_SAVE_POINT:/home/cucuridas/chatbot_tg/workReport
    	...
    	...
    	postgres:
    		image: postgres:15
    		...
    		...
    		...
    
    volumes: #docker volume 생성
    	conteinar_volume:
    		driver: local
    network: #docker network 생성
    	container_network:
    		name: container_network_test
    ```
    
- 다중 컨테이너 환경 실행 방법

```bash
# 다중 컨테이너 실행
doker-compose up 

# 다중 컨테이너 종료 및 삭제
docker-compose down
```

<aside>
💡 docker-compose 명령어 사용시 하나의 continaer만 관리 할 수 있을까?
’docker-compose help’ 라는 명령어를 실행 시켰을 때 docker-compose create라는 명령어가 존재합니다 해당 명령어를 통해 docker-compose 내부에 정의되어진 단일 컨테이너를 관리 할 수 있도록 기능을 제공합니다
추가적으로 docker-compose가 제공하는 여러 명령어를 통해 자원관리를 할 수 있습니다

</aside>