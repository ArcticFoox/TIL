# DNS A 레코드와 CNAME 레코드
## A 레코드 (Address Record)
### 개념
- 도메인 이름 -> IPv4 주소를 직접 매핑
- 가장 기본적인 DNS 레코드
```shell
example.com → 203.0.113.10
```
### 특징
- DNS 조회 시 즉시 IP 반환
- 중간 단계가 없어 빠르고 단순
- 로드밸런서, EC2, 온프레미스 서버 등에 직접 연결할 때 사용

### 장점
- 해석 과정이 단순하여 지연이 없음
- 다른 레코드(MX, TXT 등)와 같은 이름으로 공존 가능

### 단점
- IP 변경 시 모든 A 레코드 수정 필요

# CNAME 레코드 (Canonical Name Record)
## 개념
- 도메인 이름 -> 다른 도메인 별칭(alias) 연결
```shell
www.example.com → example.com
```
해석 흐름
```shell
www.example.com
 → example.com
 → 203.0.113.10
```
## 특징
- 실제 IP는 최종 도메인의 레코드(A/AAAA)가 결정

## 장점
- 대상 도메인의 IP 변경 시 자동 반영
- CDN, 외부 서비스 연동에 매우 적합

## 단점
- CNAME이 설정된 이름에는 다른 레코드를 둘 수 없음
- 루트 도메인(example.com)에는 CNAME 사용 불가

