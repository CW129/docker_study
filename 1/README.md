# Container Runtime

docker를 시작하기 전 Container에 대해서 좀 더 알아가면 좋을 거 같아  container를 만들기 위한 요소와 container runtime에 대한 정리

# Container를 생성하기 위한 요소들

- cgroup
    
    단일 또는 태스크 단위의 프로세스 그룹에 대해 자원할당을 제어하는 요소 
    
    | sub process name | Descriptions |
    | --- | --- |
    | cpu | cgroups는 시스템이 busy 상태일 때 CPU 공유를 최소화 즉 사용량을 제한 할 수 있습니다. 이 서브시스템은 CPU에 cgroup 작업 액세스를 제공하기 위한 스케줄러(Documentation/scheduler/sched-design-CFS.txt)를 제공합니다. |
    | cpuacct | 프로세스 그룹 별 CPU 자원 사용에 대한 분석 통계를 생성 및 제공합니다. (Documentation/cgroup-v1/cpuacct.txt) |
    | cpuset | 개별 CPU 및 메모리 노드를 cgroup에 바인딩 하기 위해 사용하는 서브시스템입니다(Documentation/cgroup-v1/cpusets.txt.) |
    | memory | cgroup 작업에 사용되는 메모리(프로세스, 커널, swap)를 제한하고 리포팅을 제공하는 서브시스템입니다. (Documentation/cgroup-v1/memory.txt) |
    | blkio | 특정 block device에 대한 접근을 제한하거나 제어하기 위한 서브시스템입니다. block device에 대한 IO 접근 제한을 설정할 수 있습니다. (Documentation/cgroup-v1/blkio-controller.txt) |
    | devices | cgroup의 작업 단위로 device에 대한 접근을 허용하거나 제한합니다. whitelist와 blacklist로 명시되어 있습니다. |
    | freezer | cgroup의 작업을 일시적으로 정지(suspend)하거나 다시 시작(restore)할 수 있습니다. (Documentation/cgroup-v1/freezer-subsystem.txt.) |
    | net_cls | 특정 cgroup 작업에서 발생하는 패킷을 식별하기 위한 태그(classid)를 지정할 수 있습니다. 이 태그는 방화벽 규칙으로 사용되어 질 수 있습니다. (Documentation/cgroup-v1/net_cls.txt.) |
    | net_prio | cgroup 작업에서 생성되는 네트워크 트래픽의 우선순위를 선정할 수 있습니다. (Documentation/cgroup-v1/net_prio.txt.) |
    | hugetlb | HugeTLB에 대한 제한을 설정할 수 있습니다. |
    | pid | cgroup 작업에서 생성되는 프로세스의 수를 제한할 수 있습니다. (Documentation/cgroup-v1/pids.txt.) |
- namesapce
    
    하나의 리눅스 커널에서 독립적인 공간을 할당하여 격리되어 시스템이 운영되어 질 수 있도록 지원하는 기능
    
    LXC 에서 제공하는 namespace 기능
    
    |  |  |
    | --- | --- |
    | mnt | mount 포인트를 격리하는 기능 |
    | uts | hostname과 domain 이름을 격리하는 기능 |
    | ipc | system ipc 격리 및 Posix message queue 시스템 확립 |
    | pid | process id 를 격리하는 기능 |
    | cgroup | cgroup에 대해 관리하는 기능 |
    | usr | user와 GID 를 격리하는 기능 |
    | net | network를 격리하는 기능 |

> ipc : inter process Comunication 의 약자로서 process끼리의 데이터를 주고 받는 행위를 뜻함
posix : 서로다른 unix os 의 이식성을 높이기 위해 정의한 application interface 규격
> 
- image

패키징과 비슷한 개념 → image는 container를 생성하기 위한 환경설정, 환경 변수, source code 파일 등을 모아 하나의 파일로 패키징되어 있는 것을 의미

**하드 디스크 레이아웃 이란?**

리눅스에는 모든 시스템이 파일을 통해 관리되고 연결되는 FS(file system)입니다 이에 따라 물리적 disk, network, 등 자원들이 mount라는 기능에 의해 특정 directory또는 file에 연결되어 os상에서 관리되어지게 됩니다 

하드 디스크 레이아웃이란 이러한 file과 directory를 사용자가 지정하여 하나의 묶음으로 만들 수 있음을 의미합니다 특정 mount point를 다른 특정 point의 mount 시킴으로 인해서 파일 트리구조로 생성하게되며 이렇게 패키징된 하나의 묶음을 이미지라고 칭합니다

## 저수준 런타임과 고수준 런타임

**나누어지게 된 이유** 

기존의 container 시장의 첫발을 디딘 docker는 container를 관리하기위한 모든 component들이 모놀리식 구조로 되어있어 많은 불편을 야기했습니다 이에 따라 docker는 기능들에 따라 분리하는 작업을 진행하게 되었습니다

container 생성에 대한 모듈들은 분리하는데에 문제없이 진행이 되었지만 docker 는 image를 관리하는 부분에 대한 기능이 지속적으로 변경되었습니다

이에 따라 의존적이던 application(k8s와 같은)들의 수정사항이 크게 변하게 되었고 CNCF는 이를 위해 CRI라는 container runtime interface 규격을 정의하여 표준으로 정합니다 

해당 cri를 통해 container만을 관리하게 되는 저수준 런타임과 이미지를 통해 container를 생성하고 관리하기위한 모듈인 고 수준 런타임이 생성되게 되었습니다

> **저수준과 고수준을 나누는 기준
런타임을 나누는 기준은 image를 핸들링하는지에 대한 여부입니다 이미지에 대한 생성 관리 및 해당 이미지를 통하 container 생성까지의 생명주기를 관리하는 역할은 고수준 rumtime 이 진행하게 됩니다
기존의 LXC를 통한 container관리는 k8s의 출현으로 CRI 표준을 만들어내게 되었고 CRI 표준의 기본이 되는 이미지 관리는 지원하는 runtime을 고수준 runtime이라 칭합니다**
> 

| 런타임 종류 | 관리 항목 | 종류 |
| --- | --- | --- |
| 저수준  | container를 생성하기 위한 기본적인 요소들 (LXC를 통한 namespace isolation, hardDisk layout, cgroups) | runc |
| 고수준 | 이미지를 통한 container 생성 및 pod 관리를 위한 CRI 인터페이터 규격 지원 | containerd, CRI-o |