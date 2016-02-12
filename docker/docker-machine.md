### Docker machine

-     도커 호스트가 설치된 VM이라고 생각하면 됨.
-    VM이므로 개발PC, 클라우드, 데이터센터등 아무데나 설치가능
-    클라우드 서비스의 경우는 해당 클라우드의 서비스에 특성에 맞춰주는 driver가 존재함. 
-    로컬에 설치하는 경우는 virtualbox, vmware를 지원함.
-    보통 Dooker toolbox로 설치하면 들어있음
-    명령어는 docker-machine 임
-    실제로 docker-machine을 만들고 start를 하면 mac에서는  virtualbox의 VM이 시작됨.

### Docker machine 생성하기
```
$ docker-machine create --driver virtualbox dev
```
create 명령으로 생성하는데, driver는 virtualbox가 기본값이고, aws, azure, digital ocean등등이 있다. 왜 이름이 드라이버인지는 아직 모르겠다.

```
Running pre-create checks...
Creating machine...
(dev) Copying /Users/wapj/.docker/machine/cache/boot2docker.iso to /Users/wapj/.docker/machine/machines/dev/boot2docker.iso...
(dev) Creating VirtualBox VM...
(dev) Creating SSH key...
(dev) Starting the VM...
(dev) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect Docker to this machine, run: docker-machine env dev
`
실행결과를 살펴보면,

-    boot2docker.iso를 카피한다음에 VM을 만들고
-    ssh키를 만들고
-    VM을 기동시키고, 네트웤이 올라오는걸 기다린다.
-    올라오면 boot2docker를 프로비저닝하고
-    인증키를 local -> remote 로 복사하고
-    remote(여기서는 VM)에 있는 Docker daemon에 필요한 도커 설정을한다.
-    그뒤에 도커에 잘 접속되는지 체크한다.

docker-machine에 접속하기위한 정보를 보기위해서는

```
$ docker-machine env dev
```

를 실행하면 되고, 그 결과는 아래와 같다.

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.101:2376"
export DOCKER_CERT_PATH="/Users/wapj/.docker/machine/machines/dev"
export DOCKER_MACHINE_NAME="dev"
# Run this command to configure your shell:
# eval $(docker-machine env dev)
```

도커 호스트의 ip와 포트 인증서의 위치, 도커 머신의 이름등이 나온다.

생성한 도커 머신을 확인하기
```
$ docker-machine ls
```

위의 명령어로 확인 하면 된다.

그리고 도커 머신과 정보를 주고받으려면 (이걸 공홈에서는 대화한다고 표현) 해당 도커머신에 대한 환경설정을 해야한다.

```
$ eval "$(docker-machine env dev)"
$ docker ps
```


해당 설정을 하지 않으면 도커 머신에 접속할 수 없다는 에러가 뜨게 된다.
Cannot connect to the Docker daemon. Is the docker daemon running on this host?

### 도커 머신에 이미지 실행하기

생성된 도커 머신에 이미지를 다운받고 실행해보자.

```
$ docker run busybox echo hello world
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
583635769552: Pull complete
b175bcb79023: Pull complete
Digest: sha256:c1bc9b4bffe665bf014a305cc6cf3bca0e6effeb69d681d7a208ce741dad58e0
Status: Downloaded newer image for busybox:latest
hello world
```

로컬에는 해당 이미지가 없기 때문에 docker hub에서 받아오게 (pull)되고, 해당 이미지를 실행한다.
도커머신의 ip를 알기 위해서는 아래와 같은 명령을 실행하면 되는데,
요즘 세상에는 머신의 아이피가 바뀌는건 부지기수로 있는 일이니 머신 이름으로 아이피를 받아오는 방법을 잘 알아놓는 것이 좋을 것이다.

```
$docker-machine ip dev
192.168.99.101
```

그럼 이제 nginx이미지를 받아서 실행해보고 잘되는지 확인해보자.
```
$ docker run -d -p 8000:80 nginx
```

위의 이미지를 실행하면 nginx를 다운받게 되고, -p 옵션으로 도커 호스트의 8000번 포트로 들어가게 되면, 컨테이너의 80번 포트로 포워딩을 해주게된다.

curl로 확인해보자

```
$ curl `docker-machine ip dev`:8000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


도커 머신은 말그대로 VM이므로 여러개 생성이 가능하다.

아래의 기본적인 명령어는 일단 숙지를 하고 다니자.

```
$ docker-machine create --driver virtualbox [이름]
$ docker-machine ls #리스트 보기
$ docker-machine stop [이름] # 정지시키기
$ docker-machine start [이름] # 시작하기
$ docker-machine rm [이름] # 삭제하기
```