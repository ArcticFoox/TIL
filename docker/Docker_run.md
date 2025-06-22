# Docker run

# docker run with volume

![image.png](/docker/img/image2.png)

## Dokcer volume

docker cli 를 사용해 volume 이라는 리소스를 직접 관리

host에서 Docker area(/var/lib/docker) 아래에 특정 디렉토리를 생성한 다음, 해당 경로를 container에 mount

## Bind mount

host의 특정 결로를 container에 mount

```docker
// docker volume
docker run \
    -v my_volume:/app \
    nginx:latest
    
// bind mount
docker run \
    -v /home/user/some/path:/app \
    nginx:latest
```

# docker run with resource limit

기본적으로 docker container는 host OS의 cpu, memory 자원을 fully 사용할 수 있음

host OS의 자원 상황에 따라서 OOM 등의 이슈로 비정상적 종료가 발생할 수 있음

이 문제 해결을 위해 docker container 실행 시, cpu와 memory의 사용량 제한을 걸 수 있음

```docker
docker run -d -m 512m --memory-reservation=256m --name 512-limit ubuntu sleep 3600
docker run -d -m 1g --memory-reservation=256m --name 1g-limit ubuntu sleep 3600
```

이후 docker stats 커맨드를 통해 사용량 확인 가능

```docker
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT   MEM %     NET I/O       BLOCK I/O   PIDS
4ea1258e2e09   1g-limit    0.00%     300KiB / 1GiB       0.03%     1kB / 0B      0B / 0B     1
4edf94b9a3e5   512-limit   0.00%     296KiB / 512MiB     0.06%     1.11kB / 0B   0B / 0B     1
```

# docker run with restart policy

특정 컨테이너를 계속 running 상태 유지시킬 경우,

종료되자마자 바로 재시작 시도할 수 있는 —restart=always 옵션 제공

# docker run as a background process

도커 컨테이너를 실행할 때 기본적으로 foreground process로 실행

컨테이너를 실행한 터미널이 해당 컨테이너에 자동으로 attach 되어,

다른 명령을 실행할 수 없음

-d 옵션을 통해 백그라운드에서 실행하도록 할 수 있음