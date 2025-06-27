#### Jitsi Docker 실행 방법
1. https://github.com/jitsi/docker-jitsi-meet/releases/tag/stable-6433 에 접속하여 하단의 Source code(zip)을 다운받는다.
2. 다운받은 파일을 압축 해제 후 해당 폴더로 이동 한다.
3. .env.example 파일을 .env 파일의 이름으로 복사한다.(명령어 하단 참조)

```
cp env.example .env
```

4. ./gen-passwords.sh 쉘 파일을 실행합니다.(명령어 하단 참조)

```
./gen-passwords.sh
```

5. CONFIG 폴더를 생성한다.(명령어 하단 참조)

```
# 리눅스 일 경우
mkdir -p ~/.jitsi-meet-cfg/{web/crontabs,web/letsencrypt,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}

# 윈도우 일 경우
echo web/crontabs,web/letsencrypt,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri | % { mkdir "~/.jitsi-meet-cfg/$_" }
```

6. .env 설정파일에서 알맞게 설정을 변경한다.
  - Let's Encrypt 사이트에서 인증서를 발급 받을 경우 80 포트를 HTTP_PORT, 443 포트를 HTTPS_PORT 에 포트포워딩을 해 준다.

```
# Exposed HTTP port
HTTP_PORT=8000

# Exposed HTTPS port
HTTPS_PORT=9443

# System time zone
TZ=Asia/Seoul

# Public URL for the web service (required)
# 도메인이 없다면 https://www.freenom.com 에서 무료 도메인 발급
PUBLIC_URL=https://eda-dev.tk:9443

# IP address of the Docker host
DOCKER_HOST_ADDRESS=192.168.0.83

# Media port for the Jitsi Videobridge
JVB_PORT=10000

# Enable Let's Encrypt certificate generation
# Let's Encrypt 사이트의 무료 SSL 인증서 발급 유무(1 발급, 0 발급안함)
ENABLE_LETSENCRYPT=1

# Domain for which to generate the certificate
# Let's Encrypt 사이트의 무료 SSL 인증서 발급 할 도메인
#LETSENCRYPT_DOMAIN=eda-dev.tk

# E-Mail for receiving important account notifications (mandatory)
# Let's Encrypt 사이트의 이메일 공지 받을 주소
#LETSENCRYPT_EMAIL=phkaa@edacom.co.kr
```

7. 포트를 열어준다.
```
# 리눅스 일 경우
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --permanent --add-port=4443/tcp
sudo firewall-cmd --permanent --add-port=10000/udp
sudo firewall-cmd --reload
```

8. docker-compose 명령어를 실행한다.(명령어 하단 참조)

```
docker-compose up -d 
```

9. 정상적으로 실행이 되었으면 https://localhost:8443 로 접속하면 된다.
---

#### Jitsi Components
- Jitsi Meet: Jitsi Videobridge를 사용하여 화상회의를 제공하는 WebRTC 호환 Javascript 응용 프로그램이다. 
- Jitsi Videobridge(JVB): 회의 참가자 간에 비디오 스트림을 라우팅하도록 설계된 WebRTC 호환 서버 이다.
- Jitsi Conference Focus(jicofo): 미디어 세션을 관리하고 각 참가자와 비디오 브릿지 간의 로드 밸런서 역할을 하는 Jitsi Meet에서 사용되는 서버 측 포커스 구성 요소이다.
- Jitsi Gateway to SIP(jigasi): SIP 클라이언트가 Jitsi Meet에 참여 할 수 있도록 하는 서버측 애플리케이션이다.
- Jitsi Broadcasting Infrastructure(jibri): 가상 프레임 버퍼에서 렌더링된 크롬 인스턴스를 시작하고 ffmpeg로 출력을 캡처 및 인코딩하여 작동하고 Jitsi Meet 녹화 및 스트리밍을 위한 도구 이다.
- Prosody: 시그널링에 사용되는 XMPP 서버이다.

---

#### Jitsi Architecture
![ps 이미지](https://raw.githubusercontent.com/jitsi/handbook/master/docs/assets/ArchitectureDiagram.png)

---

#### 이슈
1. jitsi docker stable-6433 릴리즈 버전 구동시 아이폰 사파리 브라우저 이슈
(오류 내용: WebSocket network error: OSStatus Error -9807)
  - SSL 적용이 안된 웹소켓 연결일 경우 방에 입장하기 전 닉네임 설정에서 서버 재연결 이슈 발생한다.(메시지: 네트워크 연결을 확인하고 있습니다. {{seconds}} 초 내에 다시 연결중입니다…)
  - 위의 임시 해결 방법은 아이폰 > 설정 > Safari > 고급 > Experimental Features > NSURLSession WebSocket 활성화 후 접속하면 된다.
  - [참고 - URL](https://stackoverflow.com/questions/37898048/websocket-network-error-osstatus-error-9807-invalid-certificate-chain)

2. 1번 웹소켓 재접속 이유를 해결하더라도 KeyboardAvoider.js 오류 발생
  - 서버에 SSL 인증서를 설정하면 해결된다. 즉 SSL 인증서 적용해야 된다.

---

#### WebRTC 디버깅
- 크롬: chrome://webrtc-internals/
- 엣지: edge://webrtc-internals/
- [관련 정보](https://docs.remotemonster.com/web/web-debug-inside)
---

#### Jitsi 튜닝관련 링크
- [https://blog.daum.net/sakwon/283](https://blog.daum.net/sakwon/283)
- [https://blog.daum.net/sakwon/294](https://blog.daum.net/sakwon/294)

---

#### 출처 및 참고
- [출처 - Jitsi Architecture](https://jitsi.github.io/handbook/docs/architecture)
- [출처 - Jitsi Docker Self-Hosting](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)
- [출처 - SDP 개념](https://nexpert.tistory.com/497)