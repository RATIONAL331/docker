# 04. Docker Networking
![dockerNetworkBasic.png](dockerNetworkBasic.png)

## Host Special Domain
* `host.docker.internal`

```javascript
mongoose.connect(
  // localhost는 도커 내부의 IP를 나타내지만 이를 로컬 호스트 머신의 IP로 바꿀 수 있음
  // 'mongodb://localhost:27017/swfavorites',
  'mongodb://host.docker.internal:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

## Container Communication
### Basic Idea
* `docker inspect {containerName}`
```json
{
  "Networks": {
    "bridge": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": null,
      "MacAddress": "02:42:ac:11:00:02",
      "DriverOpts": null,
      "NetworkID": "57f8a296ceea0cd8835c9e8b031e674b908dc9e38b3e0b96a2aa2d0ef36ebebb",
      "EndpointID": "f7ecc32468370d0146366c10884ac9f7d6636a9648cf5659b00f16a39df12198",
      "Gateway": "172.17.0.1",
      "IPAddress": "172.17.0.2",
      "IPPrefixLen": 16,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "DNSNames": null
    }
  }
}
```

* `docker inspect`로 몽고 컨테이너의 IP를 확인한 후 변경

```javascript
mongoose.connect(
  // 위의 inspect 정보로 얻은 IP
  'mongodb://172.17.0.2:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

### Container Networks
#### Key Command
* `docker network create {networkName}`
  * 새로운 네트워크 생성
* `docker network ls`
  * 네트워크 목록 확인

#### Compose network
![dockerNetwork.png](dockerNetwork.png)
* `docker create favorites-net`로 네트워크 생성
* `docker run --network {networkName} {image}`으로 생성된 네트워크 명 지정
  * `docker run -d --rm --name mongodb --network favorites-net mongo`

```javascript
mongoose.connect(
  // 위에서 컨테이너 이름을 --name mongodb 로 지정
  // 'mongodb://{containerName}:27017/swfavorites',
  'mongodb://mongodb:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```
* 같은 네트워크에 속하도록 지정
  * `docker run -d --rm --name favorites -p 3000:3000 --network favorites-net favorites-node`

## Docker Network IP Resolving
* 위의 `Host Special Domain` 또는 `Container Networks` 방식에서 도커는 소스코드를 고치지 않음
  * `host.docker.internal`, `'mongodb://{containerName}:27017/swfavorites'` 등에서 소스코드를 바꾸지 않음
* 도커는 컨테이너에서 Network 관련된 요청을 전송할 경우에 이를 인식하여 해당 시점에 실제 IP 주소로 변환
  * 요청이 컨테이너에서 전송되지 않은 경우 IP 변환은 이루어지지 않음

## Docker Network Driver
### Bridge Driver (Default Driver)
* 일반적인 네트워크 드라이버
  * 컨테이너가 동일한 네트워크에 있을 경우 컨테이너 이름을 사용하여 통신할 수 있는 기능 제공
### Other Drivers
* 네트워크 생성시 --driver 옵션을 추가하여 설정가능
  * `docker network create --driver {driverName} {networkName}`
  * `bridge` 드라이버의 경우에 디폴트이기 때문에 생략 가능
* 다음과 같은 드라이버 제공 (대부분의 경우 `bridge`)
  * `host`: 컨테이너와 호스트 시스템 간의 격리 제거 (즉 로컬 호스트 머신 자체를 네트워크로 공유함)
  * `overlay`: 여러 Docker 데몬이 서로 연결될 수 있음. 구식 연결 방식의 Swarm 모드에서만 작동
  * `macvlan`: 커스텀 MAC 주소 설정 가능. 해당 주소를 해당 컨테이너와 통신하는데 사용할 수 있음
  * `none`: 네트워킹 비활성화
  * `(third-party)`: 타사 플러그인 설치