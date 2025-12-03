
---

# mkcert 정리

##  mkcert란?

`mkcert`는 **로컬 개발 환경에서 브라우저가 신뢰하는 HTTPS 인증서를 쉽게 발급하는 도구**이다.
자체 Local CA(Local Certificate Authority)를 생성하고 이를 OS/브라우저 신뢰 저장소에 등록하여, 경고 없는 HTTPS 환경을 만들 수 있다.

---

##  mkcert 동작 방식

### 1) Local CA 생성 (`mkcert -install`)

* Root CA (`rootCA.pem`, `rootCA-key.pem`) 생성
* 이를 OS 및 브라우저 인증서 저장소에 자동 등록
  (macOS Keychain, Windows Cert Store, Linux trust store, Firefox 저장소 등)

### 2) HTTPS 인증서 발급

```bash
mkcert localhost
mkcert api.mydev.local
mkcert 127.0.0.1 ::1
```

생성되는 파일:

* `<name>.pem` — 인증서
* `<name>-key.pem` — 개인키

### 3) 개발 서버에 적용

Nginx, Spring Boot, Node.js 등에 HTTPS 인증서로 연결하여 사용한다.

---

##  왜 로컬에서 HTTPS 경고가 발생하는가?

브라우저는 다음 두 조건을 만족해야 “안전한 인증서”로 인정한다.

1. **공인 CA 발급 인증서일 것**
2. **인증서의 도메인이 URL과 정확히 일치할 것**

하지만 로컬 개발 환경에서는…

* `localhost`, `127.0.0.1` 같은 도메인은 **공인 CA가 발급하지 않음**
* 로컬 전용 도메인은 검증할 방법이 없음
* 무료 SSL(Let's Encrypt)도 로컬 도메인은 **발급 불가**

→ 결과적으로 브라우저 경고가 뜸
→ **mkcert는 Local CA를 직접 만들어 이 문제를 해결**

---

##  mkcert의 장점

###  1) HTTPS 경고 제거

로컬에서 완전히 브라우저가 신뢰하는 HTTPS 환경 구축

###  2) 설치와 사용이 매우 쉬움

한 줄로 Root CA 생성, 한 줄로 인증서 생성

###  3) 여러 도메인 한 번에 발급 가능

```bash
mkcert example.local "*.example.local" localhost 127.0.0.1
```

###  4) OS·브라우저의 신뢰 저장소 자동 처리

추가 설정 없이 바로 사용 가능

---

##  mkcert의 한계

###  1) 운영 환경에서는 절대 사용 불가

Local CA는 외부에서 신뢰할 수 없으므로
운영 서버에서는 반드시 Let's Encrypt 등 공인 인증서를 사용해야 한다.

###  2) 팀 개발 시 Root CA 공유 필요

각 개발자가 CA를 개별 생성하거나 공유해야 한다.

###  3) Docker 같은 환경은 추가 설정 필요

컨테이너 내부에도 CA를 복사해야 한다.

---

##  다양한 환경에서의 사용 예시

---

### Docker + Nginx

1. 인증서 생성:

```bash
mkcert myapp.local
```

2. nginx.conf 설정:

```nginx
ssl_certificate     /etc/ssl/myapp.local.pem;
ssl_certificate_key /etc/ssl/myapp.local-key.pem;
```

3. 컨테이너에 Root CA 복사:

```bash
mkcert -install
docker cp "$(mkcert -CAROOT)/rootCA.pem" mycontainer:/usr/local/share/ca-certificates/
```

---

###  Spring Boot 적용

`application.yml`

```yaml
server:
  ssl:
    enabled: true
    key-store: classpath:localhost.p12
    key-store-password: ""
    key-store-type: PKCS12
```

mkcert 인증서 → PKCS12 변환:

```bash
openssl pkcs12 -export -inkey localhost-key.pem -in localhost.pem -out localhost.p12
```

---

##  mkcert 설치

### macOS (Homebrew)

```bash
brew install mkcert
brew install nss     # Firefox 지원용
```

### Linux

```bash
sudo apt install mkcert
```

### Windows (Chocolatey)

```powershell
choco install mkcert
```

---

##  자주 사용하는 명령어

```bash
mkcert -install        # Root CA 생성 및 등록
mkcert localhost       # 인증서 생성
mkcert 127.0.0.1 ::1   # IP 기반 인증서
mkcert example.local "*.example.local"
mkcert -uninstall      # Root CA 삭제
```

---

##  정리

| 문제                      | mkcert 사용 시 해결 여부 |
| ----------------------- | ----------------- |
| 로컬 HTTPS 브라우저 경고        | ✔ 해결              |
| 공인 인증서가 localhost 발급 불가 | ✔ 가능              |
| 브라우저가 인증서를 신뢰하지 않음      | ✔ CA 자동 등록        |
| 여러 도메인 테스트 어려움          | ✔ SAN 포함          |

---

