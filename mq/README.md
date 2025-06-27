### NATS
- 오픈 소스 메세지 큐 서비스의 종류이다.
- Go 언어로 구현되어 있고, 클라이언트는 다양한 언어에 대한 라이브러리들을 지원한다.
- at most once 의 QoS 를 제공한다.
- subject를 통해서 메세지를 전송한다.
- subject에 대한 subscriber가 없는 경우, 해당 subject로 송신되는 메세지들은 전송되지 않는다.
- core는 fire-and-forget messaging system으로 메모리에 있는 메세지만을 사용하고, 디스크에 메세지를 저장하지 않는다.
- NATS Streaming를 사용하여 메세지 기록/이벤트 등을 저장 할 수 있다.
- Acks, Sequence Numbers와 같은 기능들을 제공한다.
- 메세지 송신시 인코딩을 하여 전달하고 수신시 디코딩하여 처리한다.

#### 메세지 전송 구조 및 기능
- Subject Based Messaging
  - Message 송신과 수신으로 구성되어 있다.
  - 송수신 모두 subject를 통해서 어떤 메세지를 topic에 포함 할지 결정한다.
  - subject는 문자열로 구현되어 있고 "." 를 통해서 subject의 계층 구조를 생성 할 수 있다.

- Publisher and Subscriber
  - publish-subscribe message distribution model for one-to-many 통신으로 구현되어 있다.
  - publisher가 특정 subject 메세지를 송신하는 경우, 해당 subject를 수신하는 subscriber가 메세지를 수신한다.

- Request and Reply
  - 분산 구조에서 많이 사용되는 패턴이다.
  - Request한 시스템이 이에 대한 Reply를 기다리는 구조이다.
  - NATS도 publish 와 subscribe에 이 패턴을 사용하고 있다.
  - 메세지 전송 시에 reply subject를 포함해서 메세지를 송신하고, subscriber는 이에 대한 응답 reply subject를 송신한다.
  - reply subject는 inbox라는 requstor를 동적으로 가리키는 특수한 subject이다.

- Queue Group
  - 로드밸런싱을 위해 분산 queue 기능을 제공한다.
  - queue group에 포함된 subscriber들에게 메세지들을 분산 전송하여 로드밸런싱을 한다.
  - queue group을 생성하기 위해서는 subscriber가 queue 이름을 등록한다.
  - queue 이름으로 등록된 subscriber들 끼리 queue를 구성한다.
  - queue의 subject에 해당하는 메세지를 송신하는 경우 queue 맴버들 중 랜덤하게 한 곳으로 메세지가 전송된다.
  - queue group 설정은 서버쪽에는 특별한 설정을 할 필요가 없기 때문에 시스템 확장에 유리하다.
  - queue에 새로운 subscriber를 등록하고 싶은 경우 새로운 application에서 queue 이름으로 등록한 하면 된다. 만약 제외하고 싶다면 application에서 지워주면 된다.

- QoS
  - NAT는 기본적으로 At most once로 서비스를 운영한다.
  - At most once 에서는 메세지의 유실이 발생할 수 있다. 하지만 request reply 구조를 사용한다면 timeout과 ack를 통해서 네트워크 문제를 해결 할 수 있다.
  - NATS의 ack는 빈 메세지로 payload를 점유하지 않는 매우 작은 메세지로 구현된다.
  - QoS 란? 다른 응용 프로그램, 사용자, 데이터 흐름 등에 우선 순위를 정하여, 데이터 전송에 특정 수전의 성능을 보장하기 위한것이다.

- Sequence Numbers
  - One to Many 의 통신 구조에서는 네트워크의 문제로 유실되거나 지연되는 경우가 있다. 이러한 문제를 해결하기 위해 메세지에 Sequence Id를 사용 할 수 있다.
  - 클라이언트는 수신받은 메세지의 Sequence Id를 통해서 메세지의 유실 여부를 판단 할 수 있다.

#### NATS Docker 설치 및 실행
1. NATS 이미지를 다운로드 받는다.
```
#> docker pull nats:2.8.4-alpine3.15
```

2. 컨테이너를 생성한다.
```
#> docker run -d -p 4222:4222 --name "nats" -ti nats:2.8.4-alpine3.15
```

3. 생성한 컨테이너 로그를 확인하여 정상적으로 실행이 되었는지 확인한다.
```
#> docker logs nats

[1] 2022/06/07 07:14:01.396668 [INF] Starting nats-server
[1] 2022/06/07 07:14:01.396729 [INF]   Version:  2.8.4
[1] 2022/06/07 07:14:01.396731 [INF]   Git:      [66524ed]
[1] 2022/06/07 07:14:01.396733 [INF]   Cluster:  my_cluster
[1] 2022/06/07 07:14:01.396735 [INF]   Name:     NCQGDII6M3LDNUZDYKIVYQJWJDJJGTFLU3FVYPWUIOTAHLZRKPPJ5LYT
[1] 2022/06/07 07:14:01.396739 [INF]   ID:       NCQGDII6M3LDNUZDYKIVYQJWJDJJGTFLU3FVYPWUIOTAHLZRKPPJ5LYT
[1] 2022/06/07 07:14:01.396744 [INF] Using configuration file: /etc/nats/nats-server.conf
[1] 2022/06/07 07:14:01.397104 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2022/06/07 07:14:01.397212 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2022/06/07 07:14:01.397464 [INF] Server is ready
[1] 2022/06/07 07:14:01.397544 [INF] Cluster name is my_cluster
[1] 2022/06/07 07:14:01.397583 [INF] Listening for route connections on 0.0.0.0:6222
```

4. 컨테이너 접속은 아래와 같다.
```
#> docker exec -it nats sh
```

#### NATS 자료 참조
- [NATS Docs](https://docs.nats.io/)
- [개념정리](https://jammdev.tistory.com/40)