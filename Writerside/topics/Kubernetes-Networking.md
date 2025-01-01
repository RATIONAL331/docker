# Kubernetes Networking

## Pod Internal Communication (Between Container)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users-app
  template:
    metadata:
      labels:
        app: users-app
    spec:
      containers:
        - name: users-container
          image: {userImageName}
          env:
            - name: AUTH_ADDRESS
              # 동일한 Pod에서 실행되는 2개의 컨테이너간 서로 통신하는 경우 localhost 사용 
              value: localhost 
        # 이미지 여러개인 Pod
        - name: auth-container
          image: {authImageName}
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users-app
  type: LoadBalancer
  ports:
    - protocol: 'TCP'
      port: 8080
      targetPort: 8880
```

* 동일 Pod 내부에서 컨테이너 간 통신은 `localhost`를 사용 

## Cluster Internal Communication (Between Pod)

### auth-deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-app
  template:
    metadata:
      labels:
        app: auth-app
    spec:
      containers:
        - name: auth-container
          image: {authImageName}
```

#### auth-service.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth-app
  type: ClusterIP # 외부로 노출 하지 않고 클러스터 내부에서만 접근 가능
  ports:
    - protocol: 'TCP'
      port: 80
      targetPort: 80
```
* k8s 클러스터 내부적으로 `service`의 이름을 모두 대문자 그리고 `-`는 `_`로 바꾼 후 `SERVICE_HOST`를 붙인 값을 주소로 사용 가능
* `auth-service`는 `AUTH_SERVICE_SERVICE_HOST`로 사용 가능

### users-deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users-app
  template:
    metadata:
      labels:
        app: users-app
    spec:
      containers:
        - name: users-container
          image: {userImageName}
```
* User Deployment 에서는 환경 변수 `AUTH_SERVICE_SERVICE_HOST` 값을 알아서 치환
* 도커 컴포즈 파일에서도 `AUTH_SERVICE_SERVICE_HOST` 환경 변수 값은 설정 해야 함
#### CoreDNS
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users-app
  template:
    metadata:
      labels:
        app: users-app
    spec:
      containers:
        - name: users-container
          image: {userImageName}
          env:
            - name: AUTH_ADDRESS
              # CoreDNS를 이용한 방법
              value: 'auth-service.default'
```
* 따로 환경 변수 설정해주지 않는 방법
* CoreDNS 라는 것을 이용할 수도 있는데 규칙은 다음과 같음
  * `serviceName` + `.` + `namespace`
* `kubectl get namespaces` 를 수행하면 현재 어떤 네임스페이스가 있는지 학인 가능


#### users-service.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users-app
  type: LoadBalancer
  ports:
    - protocol: 'TCP'
      port: 8080
      targetPort: 8880
```
