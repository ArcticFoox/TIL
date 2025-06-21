# Docker Advanced

## 이미지 생성 시 주의 점

Dockerfile을 활용하여 도커 이미지를 만들 때 명령어의 순서가 중요

도커 이미지는 여러 개의 Read-Only Layer로 구성되어,

이미지를 빌드할 때 이미 존재하는 레이어는 캐시되어 재사용됨

이를 고려한다면 빌드 시간을 단축시킬 수 있음

Dockerfile에서 `RUN`, `ADD`, `COPY` 명령어 하나가 하나의 레이어로 저장됨

```java
# Layer 1
FROM ubuntu:latest

# Layer 2
RUN apt-get update && apt-get install python3 pip3 -y

# Layer 3
RUN pip3 install -U pip && pip3 install torch

# Layer 4
COPY src/ src/

# Layer 5
CMD python src/app.py
```

Dockerfile로 빌드된 이미지를 실행하기까지를 다음과 같은 레이어로 표현 가능

![image.png](/docker/img/image.png)

최상단 R/W Layer는 이미지에 영향을 주지 않음

컨테이너 내부에서 작업한 내역은 휘발성

하단의 레이러가 변경되면, 그 위 레이어는 모두 새로 빌드 됨

→ 자주 변경되는 부분은 취대한 뒤쪽으로 정렬하는 것을 추천 ex) COPY src/ …

<br>

거의 변경되지 않지만 자주 쓰이는 부분은 공통화하여,

별도의 이미지를 미리 만든 다음, 베이스 이미지로 활용하는 것이 좋음

예를 들어, 다른 건 거의 똑같은데, tensorflow-cpu 를 사용하는 이미지와, tensorflow-gpu 를 사용하는 환경을 분리해서 이미지로 만들고 싶은 경우에는 다음과 같이 할 수 있음

python과 기타 기본적인 패키지가 설치된`ghcr.io/makinarocks/python:3.8-base`를 만들어두고,

tensorflow cpu 버전과 gpu 버전이 ****설치된 이미지 새로 만들때는, 

위의 이미지를 `FROM`으로 불러온 다음, 

tensorflow install 하는 부분만 별도로 작성해서,

Dockerfile을 2개로 분리하여 관리한다면 가독성도 좋고 빌드 시간도 줄일 수 있음

## ENTRYPOINT vs CMD

둘다 컨테이너의 실행 시점에서 어떤 명령어를 실행시키고 싶을 때 사용

### 차이점

CMD: docker run 을 수행할 때, 쉽게 변경하여 사용

ENTRYPOINT: `—entrypoint` 를 사용해야 변경 가능

```java
FROM ubuntu:latest

# 아래 4 가지 option 을 바꿔가며 직접 테스트해보시면 이해하기 편합니다.
# 단, NO ENTRYPOINT 옵션은 base image 인 ubuntu:latest 에 이미 있어서 테스트해볼 수는 없고 나머지 v2, 3, 5, 6, 8, 9, 11, 12 를 테스트해볼 수 있습니다.
# ENTRYPOINT echo "Hello ENTRYPOINT"
# ENTRYPOINT ["echo", "Hello ENTRYPOINT"]
# CMD echo "Hello CMD"
# CMD ["echo", "Hello CMD"]
```

위 내용에 대한 결과는 다음과 같음

|  | **No ENTRYPOINT** | **ENTRYPOINT a b** | **ENTRYPOINT ["a", "b"]** |
| --- | --- | --- | --- |
| **NO CMD** | Error! | /bin/sh -c a b | a b |
| **CMD ["x", "y"]** | x y | /bin/sh -c a b | a b x y |
| **CMD x y** | /bin/sh -c x y | /bin/sh -c a b | a b /bin/sh -c x y |

출처

https://mlops-for-all.github.io/docs/prerequisites/docker/advanced/