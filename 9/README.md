도커 배포

### 멀티 스테이지 빌드 ###
  1. Docker Image를 더 경량화 시키기 위한 방법(Image가 작을수록 빌드,배포의 시간이 짧아짐)
  2. 이미지 내부에서 용도를 분리시킬수 있음 ex)builder,test
  3. FROM을 기준으로 스테이지 분리


### 도커 빌드 Task ###
  1. Dockerfile을 한줄씩 읽음
  2. run, copy, add 등의 명령어를 인식할때마다 컨테이너를 생성하고 명령어 실행
  3. 명령어 실행 이후의 데이터를 새로운 레이어로 지정
  4. Dockerfile에 더이상의 명령어가 없을때 새롭게 생성된 레이어를 lowerlayer에 추가하여 새로운 이미지가 완성됨


### install.sh ###
    FROM alpine:latest
    # Set a default value for the environment variable
    ARG DEPENDENCY_TYPE=npm
    # Copy the install script to the image
    COPY install.sh /
    # Run the install script, passing in the value of the environment variable
    RUN /install.sh $DEPENDENCY_TYPE
    # Copy the application files to the image
    COPY app /app
    # Set the default command to start the application
    CMD ["./app"]

  install.sh
    if [ "$1" = "npm" ]; then
    # Install Node.js and npm
    apk add --no-cache nodejs npm
    elif [ "$1" = "pip" ]; then
        # Install Python and pip
        apk add --no-cache python3 py3-pip
    else
        echo "Invalid dependency type specified"
        exit 1
    fi



### 빌드 전략 ###
  Rolling Strategy
  
  ![image](https://user-images.githubusercontent.com/104714337/224522441-a7db5cac-3f54-4f35-a74d-21c117fdf2e6.png)
  
  
  1. v2 컨테이너 하나를 배포
  2. v2 컨테이너 하나가 정상동작할때 v1 컨테이너 하나 삭제 이후 v2를 다시 배포
  3. 최종적으로 v2만 남을때까지 반복
  
  
  Blue/Green Strategy
  
  ![image](https://user-images.githubusercontent.com/104714337/224522481-dc3a4a14-51aa-4994-81b0-fa0c8a1da2c1.png)
  
  
  1. v2 컨테이너를 배포 (기존 v1 유지)
  2. v2 컨테이너가 완전히 실행된 이후 트래픽을 v2 컨테이너로 전환
  3. 안정적이지만 자원을 두배로 사용
  
  Canary Strategy
  
  ![image](https://user-images.githubusercontent.com/104714337/224522979-284c585c-767c-4df6-b946-94ce759cb1b0.png)
  
  
  1. 정상 동작중인 v1 컨테이너들 사이에 v2 컨테이너를 혼합
  2. 테스트에 유리함
  
