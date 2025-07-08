# Connection Timeout & Read Timeout

## Connection Timeout

종단 간 연결하는데 소요되는 최대 시간을 의미

이 시간을 넘기게 되면 연결할 수 없는 것으로 판단하고 에러를 발생

이 때 연결이란 TCP 3 way handshake를 통해 TCP 연결이 생성되는 것을 의미

## Read Timeout

연결된 종단 간에 데이터를 주고 받을 때 소요되는 최대 시간을 의미

![image.png](/backend/img/timeout/image.png)

## 타임아웃 설정 기준

네트워크 상에서 패킷 유실은 꼭 장애 상황이 아니라도 언제든 발생할 수 있음

네트워크 상에서 문제가 발생했다면 가능한 빨리 인지해야 함

위 두 가지 조건을 바탕으로 적당한 값을 찾아야 함

**한번의 패킷 유실 정도는 재전송을 통해 해결할 수 있는 수준의 타임아웃**

### Connection Timeout 값 설정

TCP 3way handshake 시 발생 가능한 경우를 생각해야 함

SYN 패킷 유실, SYN + ACK 패킷 유실, ACK 패킷 유실 세가지로 분류

<br>

**SYN 패킷 유실**

SYN 패킷이 유실되면 재전송을 판단하기 위해 InitRTO 타임아웃 발생

패킷의 유실 여부를 판단하는 타임아웃 값을 RTO라고 하는데,

RTO는 두 종단 간의 RTT(Rount Trip Time)을 기준으로 생성

패킷이 두 종단 간을 흘러가는 데 필요한 최소한의 물리적인 시간을 기반으로 RTO 계산

최초로 연결을 맺은 경우 종단 간 RTT 값을 알 수 없기 때문에 특정 시간에 대한 설정이 필요하고,

이것이 InitRTO. (리눅스 상에서는 1초로 하드코딩 되어 있음)

<br>

**SYN + ACK 패킷 유실**

![image.png](/backend/img/timeout/image%201.png)

SYN 패킷 유실과 같은 방식 

<br>

**ACK 패킷 유실**

RTT를 알게 되었기에 더는 InitRTO의 적용을 받지 않고 RTT를 기반으로 계산된 RTO 적용

### Read Timeout 값 설정

영향을 주는 패킷 재전송을 위한 타임아웃 값인 RTO가 RTT를 기준으로 만들어져 보통 1초보다 짧음

![image.png](/backend/img/timeout/image%202.png)

RTO의 최소값은 200ms이기 때문에 200ms 이상으로 예상

다른 고려점으로 RTT와 요청을 받은 쪽에서 요청을 처리하기 위해 소요되는 프로세싱 타임

출처

https://alden-kang.tistory.com/20