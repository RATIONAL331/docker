# Kubernetes Data & Volume
* k8s 의 Volume은 여러 타입과 드라이버를 제공하지만 Pod이 제거되면 같이 제거됨

```yaml
apiVersion: v1
kind: Service
metadata:
  name: story-service
spec:
  selector:
    app: story
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story-img
          image: {imageName}
          volumeMounts:
            - mountPath: /app/story # 마운트 할 컨테이너 내부 경로
              name: story-volume  # 사용할 볼륨 명 지정
      volumes: # Pod 내부의 모든 컨테이너는 해당 볼륨을 사용할 수 있음
        - name: story-volume
          # type: config 형식으로 입력
          # emptyDir: {}
          hostPath: # 바인드 마운트와 유사
            path: /data # 호스트 경로 
            type: DirectoryOrCreate # hostPath Type Doc 확인
```
* 사용할 수 있는 볼륨 타입은 문서 참고
  * [Documentation](https://kubernetes.io/docs/concepts/storage/volumes/)
* `emptyDir`
  * Pod가 시작될 때 단순히 새로운 빈 디렉토리 생성
  * Pod가 살아있는 한 해당 디렉토리를 활성 상태로 유지하고 데이터를 저장 (Pod가 재실행 되면 사라짐)
  * Pod마다 각각 가지고 있기 때문에 Replica 설정을 하였을 때 볼륨을 Pod마다 가짐 
* `hostPath`
  * 여러 Pod가 호스트머신에서 하나의 동일한 경로를 공유하는 타입
  * path를 지정할 때 호스트 경로를 입력
  * type을 `DirectoryOrCreate` 을 사용하면 없으면 경로를 생성
  * 동일한 노드의 Pod만 해당 데이터에 엑세스 가능

## Persistent Volume (PV)
![persistentVolume.png](persistentVolume.png)
* Pod와 노드로부터 독립적인 볼륨
### host-pv.yml
* hostPath 타입을 사용하는 영구 볼륨 PV
  * 단일 노드 환경에서만 동작
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  hostPath: # 해당 PV가 hostPath 타입임을 선언
    path: /data
    type: DirectoryOrCreate
  capacity:
    storage: 4Gi
  volumeMode: Filesystem # Block 으로도 지정 가능
  storageClassName: standard
  accessModes:
    - ReadWriteOnce # 해당 볼륨이 단일 노드에 의해 읽기/쓰기 볼륨으로 마운트 가능
#    - ReadOnlyMany # 읽기 전용이지만 여러 노드에 의해 요청될 수 있음
#    - ReadWriteMany # 여러 노드에 의해 요청될 수 있음
```
* volumeMode와 관련된 문서는 해당 참고 [Documentation](https://www.computerweekly.com/feature/Storage-pros-and-cons-Block-vs-file-vs-object-storage)
* `kubectl apply -f=host-pv.yml`
### host-pvc.yml (claim)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  volumeName: host-pv # 정적 클레임; 동적으로 할당 하는 것은 문서 참고
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 4Gi # PV에서 설정한 값과 같거나 작아야 함 (다른 Pod에서도 Claim 할 수 있음)
```
* `kubectl get sc`
  * `storage class`는 k8s에서 스토리지 관리방법과 볼륨 구성 방법을 세부적으로 제어할 수 있게 해줌
* `kubectl apply -f=host-pvc.yml`
### deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story-img
          image: {imageName}
          volumeMounts:
            - mountPath: /app/story # 마운트 할 컨테이너 내부 경로
              name: story-volume  # 사용할 볼륨 명 지정
      volumes: 
        - name: story-volume
          persistentVolumeClaim: # pvc 설정
            claimName: host-pvc
```
* `kubectl apply -f=deployment.yml`
* `kubectl get pv`
  * 모든 PV 가져오는 명령어
* `kubectl get pvc`
  * 모든 PVC 가져오는 명령어

## environments
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story-img
          image: {imageName}
          env: # 환경 변수 설정 가능
            - name: STORY_FOLDER
              value: 'story'
          volumeMounts:
            - mountPath: /app/story 
              name: story-volume 
      volumes: 
        - name: story-volume
          persistentVolumeClaim:
            claimName: host-pvc
```
### ConfigMaps
#### environment.yml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: data-store-env
data:
  # key: value
  folder: 'story'
```
* `kubectl get configmap`
  * configMap 확인하는 명령어
* `kubectl apply -f=environment.yml`
  * configMap 설정 명령어

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story-img
          image: {imageName}
          env: # 환경 변수 설정 가능
            - name: STORY_FOLDER
              valueFrom: # configMap을 이용하여 환경 설정 값 불러오기
                configMapKeyRef:
                  name: data-store-env # configMap 이름
                  key: folder # configMap 내부에 설정된 key 값
          volumeMounts:
            - mountPath: /app/story 
              name: story-volume 
      volumes: 
        - name: story-volume
          persistentVolumeClaim:
            claimName: host-pvc
```