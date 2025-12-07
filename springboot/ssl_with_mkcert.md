# 설치
## mkcert 인증서 생성
```bash
mkcert -install
mkcert localhost
```
localhost.pem, localhost-key.pem 파일이 생성

## PEM -> PKCS#12 변환
```bash
openssl pkcs12 \
  -export \
  -in localhost.pem \
  -inkey localhost-key.pem \
  -out localhost.p12 \
  -name "spring-ssl" \
  -passout pass:1234
```

## Spring Boot 설정
```yaml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:cert/localhost.p12
    key-store-type: PKCS12
    key-store-password: 1234
    key-alias: spring-ssl
```

## 실행
```bash
https://localhost:8443 접속
```
mkcert가 생성한 인증서는 OS 신뢰 저장소에 추가되어 있으므로 브라우저에서 안전한 연결로 인식

