# docker 명령어들

도커로 apache php mysql환경을 구성중이다.
apache와 php를 하나의 환경으로 묶기로 했고, mysql은 따로 하나더 묶을 예정이다.

이 글은 위의 환경을 구축중에 공부하고 있는 것들을 정리하는 글인데, 위의 환경 구축이 완료되면
관련해서 글을 하나 더 쓸 수 있을 것 같다.

도커가 간단하네 심플하네 빠르네, 이런 것들로 홍보를 하고 있지만,
알아야 될 것 들이 생각보다 많다.
명령어를 도저히 다 외울 수가 없어서 그냥 정리하기로 했다.

도커를 공부하면서 좋은 것 하나는 bash와 자동화에 대한 지식이 뽀너스로 늘어나는 것이 좋은 것 같다.
물론 알아야되는게 많아서 짜증나는 점이기도 하다.

오픈소스 홈페이지들에 올라가 있는 정보는 내가 보기에는 너무 상세하고 스펙적인게 대부분이기 때문에,
그냥 이렇게 케이스별로 정리를 하는게 나에게는 도움이 많이 되었던것 같다.

찾아보면 대부분 나오는 것들이겠지만,
도커를 하면서 조금이라도 삽질을 줄이려면, 자신만의 커맨드라인 리스트 같은게 있으면
개발할 때 좋을 것 같다.

아직 공부중이기 때문에 해당 포스트는 추후에 좀 더 내용이 추가될 가능성이 있다.

### 도커설치(우분투)

```
$ sudo apt-get update
$ sudo apt-get install docker.io
```

### 도커설치(Centos)

```
$ sudo yum update
$ curl -fsSL https://get.docker.com/ | sh
$ sudo service docker start
```


### docker 실행시 관리자 권한으로 실행되게 하기
매번 실행할때마다 sudo 넣는게 귀찮은 사람은 실행하면 좋다.


```
$ sudo usermod -aG docker ${USER}
$ sudo service docker restart
### 잘되는지 테스트
$ docker run hello-world
```

`docker run hello-world`실행시 에러가 나는 경우는 서버를 재시작해야한다.


### docker 프로세스 모두 죽이기

```
$ docker stop $(docker ps -qa)
```

### docker 컨테이너 모두 삭제하기

가상머신에서 도커이미지를 만들게 되면, 용량이 부족하다고 가끔 뜨는데 그때 필요하다.

```
$ docker rm $(docker ps -qa)
```

### docker 이미지 모두 삭제하기
요것도 가상머신에서 용량이 부족하다고 뜰때
```
$ docker rmi $(docker images -qa)
```

### 도커 컨테이너 데몬모드로 시작시키기

```
docker run -d image명
```

### 도커 컨테이너에 배쉬연결

```
docker run -it image명
```


### 기존 ENTRYPOINT를 무시하고 entrypoint설정하기

```
docker run --entrypoint="/bin/bash"
```

### 컨테이너끼리 연결하기

```
docker run --link="db:db" image명
```

### 데몬모드로 띄우면서 포트, 볼륨, 환경설정까지 해보기

```
docker run -d -p 8080:80 -v /home/user/www:/var/www -e PHP_ERROR_REPORTING='E_ALL & ~E_STRICT'
```

### 도커 이미지 의존성 그림으로 보기

**참고**

https://speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes


```
$ docker images -viz | dot -Tpng -o docker.png
$ python -m SimpleHTTPServer
(on browser) http://machineip:8000/docker.png
```

### 도커 레지스트리 설치 및 실행

설치
```
$ docker pull registry:latest
```

실행

```
$ docker run -d -p 5000:5000 --name my-registry -v /tmp/registry:/tmp/registry registry
```
