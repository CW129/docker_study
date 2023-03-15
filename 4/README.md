Docker, Podman 등 모든 컨테이너 도구는 리눅스 커널의 기능을 활용해 컨테이너를 생성 및 관리*

###Docker Network###

-도커의 동작 흐름-

![image](https://user-images.githubusercontent.com/104714337/218274811-8623c5b7-81b8-450d-81c6-3df85295a24b.png)
  1. Docker Cli 입력
  2. 입력된 CLI가 Docker Engine(중계 역할) -> Docker Daemon으로 전달
  3. Docker Daemon에서 실제 Host OS(Container Runtime CLI를 수행하기 위한 명령들을 실행)
    
-7계층 역할-

  OSI 7 Layer Physical Layer(1) : 통신에 필요한 전기적,기계적인 신호 담당 / 들어온 데이터를 0과 1로 바꾸는 역할(Bit)

  Data Link Layer(2) : MAC 주소를 통해서 통신 / 데이터 링크 계층에서 데이터 단위는 프레임(Frame)

  Network Layer(3) : 라우팅 기능을 맡고 있는 계층(Packet)

  Transport Layer(4) :데이터 전송 / 포트 번호 사용(Segment)

  Session Layer(5) : 통신 장치 간 상호작용 및 동기화

  Presentation Layer(6) : 데이터 표현 방식 지정(암호화,압축) 세가지의 기능 1. 송신자에서 온 데이터를 해석하기 위한 응용계층 데이터 부호화, 변화 2. 수신자에서 데이터의 압축을 풀수 있는 방식으로 된 데이터 압축 3. 데이터의 암호화와 복호화

  Application Layer(7) : 프로세스 간의 정보 교환

-Linux network layer-

![image](https://user-images.githubusercontent.com/104714337/218293029-97ccde41-3d8b-4a9f-8cbd-6c9c081e9632.png)


  User Space : 
    
    Application이 전송할 데이터를 생성하여 write 시스템 콜 호출(커널 영역으로 넘어감)
    
  Kernel Space : 
  
    File Layer - 파일(file) 레이어는 단순한 검사만 하고 파일 구조체에 연결된 소켓 구조체를 사용해서 소켓 함수를 호출
    
    Socekts - Write 시스템 콜을 호출하면 유저 영역의 데이터가 커널 메모리로 복사되고, send socket buffer 의 뒷부분에 추가
    
    TCP - TCP segment, 즉 패킷을 생성한다. Flow control 같은 이유로 데이터 전송이 불가능하면 시스템 콜은 여기서 끝나고, 유저 모드로 돌아간다(애플리케이션으로 제어권이 넘어간다)
          (TCP segment는 TCP 헤더와 TCP Payload 두가지 정보가 있음, TCP Payload가 실제 데이터)
          
    IP - TCP segment 에 IP 헤더를 추가하고, IP routing 을 한다(IP routing 이란 next hop을 찾는 것)
    
    Ethernet - ARP(Address Resolution Protocol)를 사용해서 next hop IP 의 MAC 주소 찾음(Ethernet 헤더를 패킷에 추가)
    
    Driver - 드라이버-NIC 통신 규약에 따라 패킷 전송을 요청
    
  device Space : 
  
    요놈이 실제 선으로 bit를 전송

-Docker Network Types-

  Bridge: 
  
    default Network Driver - Link Layer 네트워크로 Host의 커널 하드웨어 혹은 소프트웨어 디바이스를 사용 Layer 2 (Data Link)

  host: 
    
    Docker daemon의 인터페이스(가상) 을 거치지 않고 호스트의 인터페이스에 직접 통신 Layer 3 (Network) Mac, Window 는 사용할 수 없음 (가상머신에 도커를 설치하기 때문에) Network Namespace는 호스트와 공유, 다른 Namespace는 동일하게 격리

  overlay: 
    
    swarm 에서 사용할 수 있는 Network Type, 분산 네트워킹을 위한 기술로 도커 클러스터 구성할 때 필요 docker network create -d overlay overlay_test

  ipvlan: 
    
    Linux kernel 기술로 하나의 물리 인터페이스에 여러 가상 인터페이스를 만듬 L2 Mode : 가상 장치는 ARP 요청을 수신/응답, 가장 단순한 통신(성능이 좋지만 네트워크 트래픽에 대한 제어가 약함) L3 Mode : L3 이상의 트래픽만 처리(ARP 요청에 응답하지 않음) \ iptables의 실제 네트워크에 container를 직접 연결

  macvlan: 
    
    하나의 인터페이스에 여러개의 Mac주소를 가지는 네트워크 인터페이스로 분리하여 사용하는 기술 IPvlan과의 차이점은 Macvlan은 고유의 Mac주소 할당, IPvlan은 호스트와 동일한 Mac주소 사용 * Macvlan에선 호스트 NIC가 Promiscuous mode를 사용해야함(패킷을 컨테이너로 전달하지 않음)

  none: 
     
    단어 그대로 의미

  Network Plugins: 
    
    넘겨

-Docker Network 이해에 필요한 네임스페이스 개념-

  UTS namespace: 
    
    Host name, Domain name을 위한 격리 왜 사용하는가? 
    하나의 Host에서 여러개의 애플리케이션을 올리는 상황에서 여러개의 애플리케이션이 같은 포트와 IP를 사용할지만 각각의 hostname으로 식별이 필요한 경우

  Network namespace:
    
    리눅스 시스템에서 네트워크는 하나만 존재하는 글로벌 자원. 
    네트워크 인터페이스, 라우팅테이블 등은 하나만 있으며 시스템의 모든 계정이 이 자원을 공유하기 때문에 네트워크 정보가 변경될 경우 시스템 전체에 영향 네트워크 네임스페이스는 이런 문제 때문에 '네트워크 공간'을 격리하기 위한 기능, 여기에 인터페이스 추가, 네트워크를 설정해야함 (네트워크 네임스페이스는 virtual Ethernet만 할당 할 수 있음)


  

-- 참고-- 
  리눅스에서 네임스페이스를 분리하는 경우를 제외하면 모든 프로세스는 init(1) 번 프로세스의 네임스페이스를 공유해서 사용

  도커--link option : 
    
    컨테이너간의 Channel을 생성, Channel은 Pipe, shared memory, other communication mechanism ( socket is coomonlly used )

  Promiscuous mode : 
    
    네트워크 인터페이스는 패킷이 도착할때 패킷의 2계층 목적지 주소를 확인 > 목적지가 해당 인터페이스의 주소 혹은 브로드캐스트 주소가 아닐경우 패킷을 폐기 Promiscuous mode는 패킷의 목적지 주소가 폐기조건에 해당되도 패킷을 내부로 전달하는 방식
    
  Docker LibNetwork 모델 디자인 참고 https://github.com/moby/libnetwork/blob/master/docs/design.md

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!굉장히 좋은 참고자료!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! 
꼭 읽어보길
http://cloudrain21.com/container-networking-model
