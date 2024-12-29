# Kubernetes Essential
* k8s는 `Pods`를 생성하고 관리하지만 `Cluster`와 `Node`들은 개발자가 직접 생성해야 함
* 개발자는 `API Server`, `kubelet`, `kube-proxy`, `scheduler`와 같은 것들은 직접 설정해줘야 함
  * 쿠버네티스가 위와 같은 모든 것들은 설치하지는 않음
    * Kubermatic 또는 EKS와 같은 관리형 서비스를 사용하면 해당 내용을 자동으로 챙겨줌
* k8s는 `Pods`와 컨테이너를 모니터링 및 스케일링하는 관리에만 관심을 가짐
* k8s는 `Pod`, `Deployment`, `Service`, `Volume`등과 같은 `Object`를 이해하고 해당 `Objects`에서 명령이 수행됨

## kubectl

* k8s와 소통할 수 있는 창구
  * 마스터 노드와 해당 클러스터의 `Control Plane`

### kubectl create deployment
![kubectl.png](kubectl.png)
* 명령어를 실행하면 k8s에 있는 `Master Node(Control Plane)`으로 전송
* 해당 `Master Node`는 클러스터에 필요한 것을 생성
  * `Worker Node`에 `Pod`를 배포하는 일 등등
* `Scheduler`가 현재 실행 중인 `Pod`를 분석하여 새롭게 생성된 `Pod`에 적합한 `Node`를 찾음
  * 찾은 노드에서 kubelet 서비스를 얻어냄
    * 해당 `kubelet`에서 `Pod`를 모니터링

### Pod
* k8s와 상호작용할 수 있는 가장 작은 단위
* 하나의 컨테이너 또는 여러개의 컨테이너를 가질 수 있음
* 볼륨과 같은 공유 리소스를 가질 수 있음
* 클러스터 내부 IP 주소 포함
  * `Pod` 안에 있는 컨테이너들은 `localhost`로 통신이 가능함
* k8s에 의해 교체되거나 제거되면 모든 리소스들은 사라짐

### Deployment
* 여러개의 Pod들은 관리
  * 생성하고 관리해야하는 `Pod` 수와 컨테이너 수에 대한 지침 제공
* 일시 중지, 삭제, 롤백 가능
* 오토 스케일링 기능 사용 
  * 더 많은 수의 `Pod`가 생성되거나 생성된 `Pod`를 삭제할 수 있음
* `Deployment` 하나로 한 종류의 `Pod`를 관리
  * `Pod`를 직접 생성하지 않고, 대신 `Deployment`를 사용하여 생성 및 관리

#### Deployment Key Command
* `kubectl create deployment {deploymentName} --image={imageName}`
  * `Deployment` 객체를 만드는 명령어
  * 이미지 이름은 로컬 머신이 아닌 레지스트리에서 찾을 수 있어야 함
* `kubectl delete deployment {deploymentName}`
  * 생성된 `Deployment`를 삭제하기 위한 명령어
* `kubectl get deployments`
  * 생성된 `Deployment`를 확인하기 위한 명령어
    * READY 상태를 확인하여 정상적인지 확인 가능
* `kubectl get pods`
  * `Deployment`에 의해 생성된 Pod 정보 확인 명령어
    * READY 및 STATUS를 확인하여 정상적인지 확인 가능
* `kubectl scale deployment/{deploymentName} --replicas={replicationNum}`
  * 해당 명령을 수행하게 되면 같은 `Pod`로 replicationNum개 생성됨
  * 로드밸런서가 있다면 위 여러개 `Pod`로 트래픽이 분산됨
* `kubectl set image deployment/{deploymentName} {beforeImageName}={newImageName}`
  * 새로운 이미지로 교체하는 명령어
  * 이미지가 변경되었어도 태그(버전 정보)가 변경되지 않으면 반영되지 않음
  * 만약 해당 명령어로 배포에 실패한다면 `kubectl get pods` 실행했을 때 실패한 `Pod` 정보가 보여짐
* `kubectl rollout status deployment/{deploymentName}`
  * `Deployment`가 업데이트 되었는지 확인하는 명령어
* `kubectl rollout undo deployment/{deploymentName}`
  * 최신의 `Deployment`가 되돌려짐 
  * 배포 실패한 후 해당 명령 실행후 `kubectl get pods` 실행했을 때 실패한 `Pod` 정보가 사라짐
  * --to-revision=(revisionNum) 옵션을 붙이면 특정 버전으로 되돌리기 가능
* `kubectl rollout history deployment/{deploymentName}`
  * 배포 히스토리 확인 가능
  * --revision=(revisionNum) 옵션을 붙이면 상세 정보를 확인할 수 있음

### Service
* `Pod` 또는 `Pod`에서 실행되는 컨테이너에 접근하기 위한 객체
  * Pod 간 노출 또는 클러스터 외부와 `Pod` 간 노출 포함
* `Pod`는 기본적으로 내부 IP 주소를 가지고 있음
  * 클러스터 외부에서는 사용할 수 없음
  * 교체될 때 마다 변경됨
* `Service`는 `Pod`를 그룹화하고, 공유 IP 주소를 제공
  * 해당 IP는 변경되지 않음
  * 클러스터 외부에서도 접근가능하게 끔 할 수 있음
#### Service Key Command
* `kubectl expose deployment {deploymentName} --type=LoadBalancer --port {portNum}`
  * `Service`를 생성하여 `Deployment`에 의해 생성된 Pod 노출
  * `ServiceName`은 `DeploymentName`으로 지정
  * type 종류는 다음과 같음
    * `ClusterIP`의 경우 기본값이며 기본적으로 클러스터 내부에서만 연결할 수 있음(교체될 때 변경되지는 않음)
    * `NodePort`의 경우 `Worker Node`의 IP주소를 통해 노출(외부와 엑세스 가능)
    * `LoadBalancer`는 클러스터 인프라에 존재하는 경우에 로드 밸런서 활용
      * 들어오는 트래픽을 `Service`의 일부인 모든 `Pod`에게 고르게 분산
      * 클러스터와 클러스터가 실행되는 인프라가 지원하는 경우에만 사용 가능
        * AWS는 사용 가능
* `kubectl get services`
  * 생성된 `Service`를 확인하기 위한 명령어
  * CLUSTER-IP는 내부적으로 사용되는 IP이기 때문에 외부에서 접근 불가
  * EXTERNAL-IP로 접근 가능
* `kubectl delete service {serviceName}`
  * 생성된 `Service`를 삭제하기 위한 명령어

## Declarative Style
### deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment # 생성하고자 하는 k8s object(Deployment, Service, Job, ...) 선언
metadata:
  name: first-app-deploy # Deployment Object 이름 설정
spec:
  selector:
    matchLabels:
      # Deployment와 일치시키려는 Pod 레이블의 Key-Value 쌍 추가
      # 이 때 하나라도 충족하지 않으면 Deployment에 의해 관리되지 않음 (모든 조건을 충족해야 함)
      app: first-app
      tier: backend
  # 동일한 컨테이너를 포함하는 n개의 pod 시작할 때 구동 (미 지정시 기본 1)
  replicas: 3 # 처음에 Pod가 실행되는 것을 원하지 않는다면 0도 가능
  template: # Pod 내부 생성 선언
    metadata:
      labels:
        # key는 아무거나 원하는 것으로 지정 가능 (selector의 matchLabels와 동일하기만 하면 됨)
        app: first-app
        tier: backend # Pod 객체 하나에 하나 이상의 레이블 부여 가능
    spec:
      containers:
        - name: first-node # 컨테이너 이름
          image: {imageName} # 레지스트리 이미지 이름
          imagePullPolicy: Always # 태그가 동일하더라도 항상 최신의 이미지 가져오기
          livenessProbe: # 실행 중인지 검사하는 설정
            httpGet:
              path: /
              port: 8080
#              httpHeaders:
            periodSeconds: 10 # 빈도
            initialDelaySeconds: 5 # 처음 상태를 확인하기 까지의 지연 시간
      # - name: ...
      #   image: ...
```
* `kubectl apply -f=deployment.yml`을 수행하여 `Pod`, `Deployment` 모두 생성
  * -f는 여러개 선언 가능
* `kubectl delete -f=deployment.yml`을 수행하여 생성된 리소스 삭제 가능

### service.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector: # 제어하고자 하는 Pod를 선택 
    # 여러개를 지정하면 조건을 모두 만족하는 Pod를 선택
    app: first-app
  ports:
    - protocol: 'TCP'
      port: 80 # 외부 포트
      targetPort: 8080 # 컨테이너 내부 포트
#    - protocol: 'TCP'
#      port: 443
#      targetPort: 443
  type: LoadBalancer # (ClusterIP가 기본 값)
```
* `kubectl apply -f=service.yml`을 수행하여 `Service` 생성
* `kubectl delete -f=deployment.yml -f=service.yml`을 수행하여 생성된 리소스 삭제 가능

### master-deployment.yml
```yaml
# 서비스 객체를 먼저 생성하는 것이 권장됨
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector: # 생성되고 삭제되는 것을 지속적으로 모니터링
    # selector의 레이블과 일치하는 새로운 부분이 생성되면 서비츠에 추가됨
    app: first-app
  ports:
    - protocol: 'TCP'
      port: 80 
      targetPort: 8080 
  type: LoadBalancer
--- # 구분 기호로 반드시 - 3개
apiVersion: apps/v1
kind: Deployment
metadata:
  name: first-app-deploy
spec:
  selector:
    matchLabels:
      app: first-app
      tier: backend
  replicas: 3 
  template: 
    metadata:
      labels:
        app: first-app
        tier: backend
    spec:
      containers:
        - name: first-node
          image: {imageName}
```
* `kubectl apply -f=master-deployment.yml`을 수행하여 `Deployment`, `Pod`, `Service` 생성

#### matchExpressions, label
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: first-app-deploy
  labels:
    # 레이블 생성 가능
    group: example
spec:
  selector:
#    matchLabels:
#      app: first-app
#      tier: backend
  matchExpressions:
    - {key: app, values: [first-app], operator: In}
  replicas: 3 
  template: 
    metadata:
      labels:
        app: first-app
        tier: backend
    spec:
      containers:
        - name: first-node
          image: {imageName}
```

* 좀 더 복잡하게 Pod를 선택해야하는 경우 `matchExpressions 사용 가능`
  * [Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    # 레이블 생성 가능
    group: example
spec:
  selector: 
    app: first-app
  ports:
    - protocol: 'TCP'
      port: 80 
      targetPort: 8080 
  type: LoadBalancer 
```

* `kubectl delete deployments,sevices -l group=example`
  * `Deployment`와 `Service` 중 `group=example` 레이블을 가지는 리소스 삭제