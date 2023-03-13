### docker run --rm 옵션 없이 기동 후 수동 삭제 시
- docker stop [volume_name]; docker rm [volume_name] 시 생성된 볼륨 삭제되지 않음

    - 컨테이너 삭제 전

        ![image](https://user-images.githubusercontent.com/59682268/214531144-db71c3f5-088b-4f59-b411-ed4656e01562.png)

    - 컨테이너 삭제 후

        ![image](https://user-images.githubusercontent.com/59682268/214531273-3238cbfb-7953-45e9-a060-712c3fc05705.png)

### docker run --rm 옵션으로 기동 시
- docker stop [volume_name] 시(컨테이너가 종료됨과 동시에 삭제됨) 볼륨 삭제 됨

    - 컨테이너 종료 전

    ![image](https://user-images.githubusercontent.com/59682268/214530302-313b63bf-bdd2-4584-b89b-c95662af06b1.png)

    - 컨테이너 종료 후

    ![image](https://user-images.githubusercontent.com/59682268/214530524-0a378f61-d31a-494d-aeff-15cc9398ab4b.png)

## 도커 볼륨 여러 가지 실험

- Dockerfile 내부에서 VOLUME 작성 생략 시

    ```Dockerfile
    FROM node:14

    WORKDIR /app

    COPY ./package.json /app

    RUN npm install

    COPY . .

    EXPOSE 80

    # VOLUME ["/app/feedback"]

    CMD [ "node", "server.js" ]
    ```

    => 컨테이너 생성 시 볼륨 마운트 위치를 -v [volume_name]:[container_directory_path]와 같이 잘 작성하면 named 볼륨 정상 생성됨.

    - 단, 경로 지정하지 않고 -v [volume_name] 까지만 작성하게 되면 익명 볼륨으로 생성됨.
    
- Dockerfile 내부에서 VOLUME 작성 시

    ```Dockerfile
    FROM node:14

    WORKDIR /app

    COPY ./package.json /app

    RUN npm install

    COPY . .

    EXPOSE 80

    VOLUME ["/app/feedback"]

    CMD [ "node", "server.js" ]
    ```

    => 이 경우 Dockerfile 내부에 작성된 VOLUME path와 -v옵션의 named 볼륨 path를 일치시켜주지 않으면 
    ex) docker run --name feedback-app --rm -d -p 3000:80 **-v feedback-volume:/app** feedback-image
    (-v 옵션과 VOLUME의 path가 한 경로(?) 차이)

    일치하지 않는 path depth(또는 level) 만큼의 익명볼륨이 추가로 생성된다.

### bind mount 시 현재 경로(컨테이너 기동 시 호스트 폴더경로를 컨테이너에 마운트)

```bash
# linux/mac
-v $(pwd):/app
```

```powershell
# window
-v "%cd%":/app
```

### docker volume bind mount

```bash
docker run --name feedback-app --rm -d -p 3000:80 -v feedback:/app/feedback -v C:\Users\haseu\Desktop\Practice\udemy_docker\data-volumes-01-starting-setup\:/app -v /app/node_modules feedback
```

1. 컨테이너 내부경로 마운트
    ```pre
        -v feedback:/app/feedback
    ```

2. 동기화할 호스트의 마운트 path 설정
    ```pre
        -v C:\Users\haseu\Desktop\Practice\udemy_docker\data-volumes-01-starting-setup\:/app
    ```

3. node module 보호를 위한 추가 마운트
    ```pre
        -v /app/node_modules
    ```

=> 2번과 3번의 경우 컨테이너 내부에 mount할 path가 중복되는데 이 때 도커는 path가 더 긴 경로를 우선시한다. 이런 특성을 이용해서 /app/node_modules 경로에 node module들이 없는 호스트의 폴더를 컨테이너에 동기화 및 덮어씌우며 server.js를 동작시키기 위해 필요한 node modules를 저장하는 역할을 한다.

### docker bind mount : read-only

- bind mount 시 read-only 설정

    -v [host_path]:/[container_path]:ro

    ```bash
        -v C:\User:/app:ro
    ```

- read-only 설정 시 세부 path 제외

    -v /[container_except_path]

    ```bash
        -v /app/temp
    ```

    => 더 세부적인 경로 설정 시 우선 시행되므로 read-only 설정에서 제외됨


### Dockerfile 내부에서 COPY 구문을 사용하는 이유

=> bind mount를 실행함으로써 실행중인 컨테이너와 호스트 내부 작업폴더는 동기화 된다.
즉, COPY 명령 없이도 컨테이너 작업 디렉토리 내부에는 필요한 (파일 및 폴더에 대한)작업환경이 구성된다.

하지만, 이 기능은 개발 편의 상 실행중인 컨테이너 내부에 호스트 파일 변경사항을 바로 적용하도록 하기 위함이다.

따라서 개발이 끝난 상태에서는 bind mount 기능을 사용하지 않고 완성된 애플리케이션을 image로 저장하게 되므로 COPY를 사용한다.

- COPY가 없을 경우
    - host directory_env => X image, X copy to container
- COPY가 있을 경우
    - host directory_env => O image, O copy to container
    
tip) container 기동 시 copy 생략 옵션
    
    ```bash
        docker run [image_name:no-copy]
    ```



### Docker env variables

- ARG

    - docker build 실행 시 이므로 이미지 생성 시

    - 실행 중인 애플리케이션이나, cmd에서 참조 불가. Dockerfile 내부에서만 사용 가능. Dockerfile 내부에서도 CMD에는 사용 불가(running container 동작 시 실행되는 CMD는 ARG 사용범위를 벗어남)

    - 사용
        - Dockerfile에서 사용 시
            ```Dockerfile
                ARG DEFAULT_PORT=80
            ```
        - commandline 사용 시
            --build-arg [args]=[variables]

- ENV
    
    - docker run 실행 시 이므로 컨테이너 기동 시

    - Dockerfile 뿐만 아니라 실행 중인 애플리케이션에서도 참조 가능

    - 사용 
        - Dockerfile에서 사용 시 
            ```Dockerfile
                ENV PORT 80

                EXPOSE $PORT
            ```
        - commandline 사용 시
            --env [envs]=[variables] (--env 대신 -e 사용가능)
        - commandline에서 file 사용 시
            --env-file [filepath]
    
- 보안 tips
    !![image](https://user-images.githubusercontent.com/59682268/215265475-79cef9c8-1695-4c00-aedd-9214799244a1.png)
    
 ### CMD와 ENTRYPOINT 커맨드 작성 시 주의사항

- CMD 명령어만 사용할 경우

    ```dockerfile
    FROM ubuntu:20.04

    CMD ["echo", "CMD test"]
    ```

    ![image](https://user-images.githubusercontent.com/59682268/212460397-d38f47e7-c568-4786-b6b8-600aba339627.png)

- ENTRYPOINT만 사용할 경우

    ```dockerfile
    FROM ubunut:20.04

    ENTRYPOINT [ "echo", "ENTRYPOINT test" ]
    ```
    ![image](https://user-images.githubusercontent.com/59682268/212460459-c929fc9e-0766-4e95-99a4-97d9272c658a.png)

=> docker run을 통해 컨테이너를 생성 시 추가 명령어를 입력하지 않으면 CMD와 ENTRYPOINT는 같은 동작을 하지만,
추가 명령을 주게 될 경우 CMD는 작성된 명령어 대신 docker run 뒤에 붙게 되는 명령어로 대체된다. 반면 ENTRYPOINT는 반드시 ENTRYPOINT에 작성된 명령어를 실행하게 된다.

- CMD만 작성된 파일에서 docker run [추가 명령] 실행

    ![image](https://user-images.githubusercontent.com/59682268/212460627-4106a7f8-c07f-4b67-80e6-6aafcfa5803f.png)

- ENTRYPOINT만 작성된 파일에서 docker run [추가 명령] 실행

    ![image](https://user-images.githubusercontent.com/59682268/212460709-8b4a1c61-15ce-4383-9f2d-9f43ceba60ac.png)

=> 위의 결과를 통해 알 수 있듯이 CMD 파일을 통해 생성된 이미지에선 run 옵션 뒤의 명령어로 대체된 반면, ENTRYPOINT의 경우 작성된 명령어가 사라지지 않고 첫 번째로 실행된 뒤 run 옵션 뒤에 작성된 명령어는 단순히 문자열 출력으로 이어졌음을 알 수 있다.(echo hello라면 hello만 출력되어야 하지만 echo hello 전체가 출력됨.)

- ENTRYPOINT와 CMD가 같이 사용된 경우
    ```dockerfile
    FROM ubuntu:20.04

    CMD ["echo", "cmd test"]

    ENTRYPOINT [ "echo", "entry test" ]
    ```
    - docker run 기본 실행
    
        ![image](https://user-images.githubusercontent.com/59682268/212460984-4e753364-f387-4829-b5ee-24122d58dcc4.png)
    - docker run 추가 명령어 실행
    
        ![image](https://user-images.githubusercontent.com/59682268/212460937-02739cd5-f914-4ca2-88d9-0d1a9d9e28c5.png)
    
=> 위의 실행 결과에서 알 수 있듯이 ENTRYPOINT와 CMD가 같이 사용되면 ENTRYPOINT를 쉘 명령어로 인식하고 CMD에 작성된 명령은 일반 문자열로 인식해버린다. (Dockerfile의 CMD와 ENTRYPOINT 위치를 바꿔서 생성된 이미지로도 동일한 결과)
