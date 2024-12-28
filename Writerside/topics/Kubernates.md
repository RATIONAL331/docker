# Kubernetes
* 컨테이너를 배포하고 오케스트레이션 하기 위한 도구
  * 자동 배포
  * 스케일링 & 로드 밸런싱
  * 컨테이너 관리
* 여러 머신을 위한 Docker Compose 로 편하게 생각할 수 있음

## Core Concept & Architecture

![kubernetes.png](kubernetes.png)

### Pod
* 단순히 컨테이너를 보유
  * 여러 컨테이너를 보유할 수 있음
* 컨테이너를 실행하기 위한 구성 뿐 아니라 볼륨과 같은 것들도 포함
* Master Node에 의해 생성되고 관리됨
* 내부적으로 최종적으로는 컨테이너를 수행 (`docker run`)

### Worker Node
![workerNode.png](workerNode.png)

* Pod를 실행하는 주체
  * 즉 컨테이너를 실행하는 주체
  * 하나의 호스트 머신, 가상 인스턴스로 생각할 수 있음
    * 어디선가 실행 중인 EC2 인스턴스라고 생각해되 됨
* 여러개의 Pod를 가질 수 있으며 실행할 수 있음
  * 트래픽 분산을 위한 Pod의 복사본일 수 있음
  * 완전히 다른 작업을 위한 Pod일 수도 있음
* Pod에 필요한 `docker`, `kubelet`, `kube-proxy`
  * `kubelet`은 Worker Node와 Master Node 간의 통신 장치
  * `kube-proxy`은 들어오고 나가는 트래픽을 처리하는 책임을 가지는 장치

### Proxy / Config
* Worker Node의 Pod 네트워크 트래픽 제어를 설정하는 또 다른 도구
* Pod 및 내부에서 실행되는 컨테이너를 외부에 어떻게 접근할 수 있는지 제어

### Master Node
![masterNode.png](masterNode.png)

#### API Server
* 워커 노드에서 실행되는 `kubelet`에 대한 통신 통로
#### Scheduler
* Pod를 지속적으로 관찰 새 Pod이 생성되어야 하는지 관찰
* Worker Node를 실행 해야하는지 관찰
#### Kube Controller Manager
* Worker Node 전체를 감시하고 제어
* 적당한 수의 Pod를 가동 중에 있는지 확인
#### Cloud Controller Manager
* 기본적으로 `Kube Controller Manager`와 동일한 작업을 수행
* 특정 프로바이더 (AWS, Azure, GCP, ... )에게 무엇을 해야하는지 알려주는 역할
#### Control Plane
* Worker Node 들과 상호 작용하며 제어하는 컨트롤 센터
  * 일반적으로 개발자가 직접 Worker Node를 직접 제어하지는 않음
  * 개발자는 해당 Control Plane을 이용하여 Worker Node를 제어
* Master Node에서 실행되는 다양한 서비스의 다양한 도구 모음

### Cluster
* Master Node, Worker Node 들이 실행되는 모든 것의 컬렉션

