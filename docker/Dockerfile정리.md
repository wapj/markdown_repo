# Dockerfile 정리

### FROM
어떤 이미지를 기반으로 도커 이미지를 생성할지 설정함.

```
FROM ubuntu:14.04
```

### VOLUME

데이터를 컨테이너에 저장하지 않고 호스트에 저장하도록 설정.

호스트의 특정디렉토리와 연결되도록 설정하려면 `docker run` 명령에서 `-v` 옵션을 사용하면됨

```
$ sudo docker run -v /home/user/html:/var/www
```

### WORKDIR

RUN, CMD, ENTRYPOINT가 실행될 디렉토리를 지정

```
WORKDIR /etc/apache2/sites-enabled
```

### RUN
쉘 명령어를 실행

```
RUN useradd seungkyoo -d /home/seungkyoo
```

### CMD, ENTRYPOINT
컨테이너가 시작될때 명령을 실행함


```
CMD ["echo", "hello"]
```

```
ENTORYPOINT /run
```

ENTRYPOINT의 경우는 run실행시 --entrypoint 옵션으로 줄수 있음

```
$ docker run hello --entrypoint="echo" hello world
```

### EXPOSE
컨테이너의 포트를 개방

```
EXPOSE 22 80
```

