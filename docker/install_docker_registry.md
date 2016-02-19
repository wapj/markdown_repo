# docker레지스트리 설치하기

보통은 서버의 형상관리를 `chef`로 관리하지만,
소스를 배포할 때에 하나의 이미지로 뭉쳐서 배포를 해보고 싶어서
`docker`를 도입해보려고 공부중이다.

그런데 내가 생성한 이미지를 `docker hub`라는 열린 `docker registry`에
올릴 수는 없어서, 찾아본것이 `docker registry`였다.

책이나 블로그에서 그다지 심도있게 다루고 있지 않아서, 쉬운가보다 했는데,
왠걸 설치가 생각보다 까다로웠다.

디지털오션의 글이 정말 많이 도움이 됐는데, 거기 있는 글을 보고
내용을 조금 추가, 수정 한것이라 보시면 될 것 같다.

모쪼록 `docker registry`를 도입하고자 하는 사람이 이 글을 보고 조금이나마 쉽게
구축 할 수 있으면 좋겠다.

### 전제조건

- ubuntu 14.04를 사용
- sudo 권한이 있는 not root 유저를 사용
- docker와 docker compose는 설치가 되어 있어야함
- docker registry에서 사용할 도메인이 있어야함

### docker 설치

#### 설치전에 해야할 것들

뭔가 귀찮은게 많이 있는데, 그냥 빨리 설치하고 싶은 사람은 아래에 `설.치.`로 바로 이동하자.
시간이 있는 사람들은 봐두면 나중에 도움이 되는 부분은 있을 것 같다.

```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ sudo cd /etc/apt/sources.list.d
$ sudo touch docker.list
```

docker.list파일에 아래의 내용을 추가
```
### ubuntu 12.04
deb https://apt.dockerproject.org/repo ubuntu-precise main

### ubuntu 14.04
deb https://apt.dockerproject.org/repo ubuntu-trusty main

### ubuntu 15.10
deb https://apt.dockerproject.org/repo ubuntu-wily main
```

```
### `APT` 패키지의 인덱스를 업데이트
$ sudo apt-get update

### 오래된 버전 삭제
$ sudo apt-get purge lxc-docker

### `APT`확인하기
$ sudo apt-cache policy docker-engine
```

#### linux-image-extra 설치

```
$ sudo apt-get update
$ sudo apt-get install linux-image-extra-$(uname -r)
$ sudo apt-get install apparmor
```

#### 설.치.

위에 주저리 주저리 많은데, 보통은 이것만 해도 설치가 된다.
```
$ sudo apt-get update
$ sudo apt-get install docker-engine
$ sudo service docker start
$ sudo docker run hello-world
```

요즘에 도커를 설치할 일이 많아서, 아래의 링크에 설치 스크리트를 정리해 두었다.

[도커 설치 스크립트](https://github.com/wapj/scripts/blob/master/install/install_docker.sh)


### docker compose 설치

```
$ sudo apt-get -y install python-pip
$ sudo pip install docker-compose
```

####  docker compose 테스트

```
$ cd ~/docker-registry/
$ mkdir hello-world
$ cd hello-world
$ vi docker-compose.yml
```

`docker-compose.yml`을 아래와 같이 만들자.

```
test-compose:
    images: hello-world
```

**테스트**
```
$ docker-compose up
```

아래와 같이 나오면 성공이다.
```
Creating helloworld_test-compose_1
Attaching to helloworld_test-compose_1
test-compose_1 |
test-compose_1 | Hello from Docker.
test-compose_1 | This message shows that your installation appears to be working
.......... 블라블라.................
test-compose_1 |
helloworld_test-compose_1 exited with code 0
```

`docker-registry` 설치에 `docker-compose`가 이용되는 것이므로
`docker-compose`에 대해 더 알고 싶은 사람은 아래 링크를 참고하자.

[Docker compose](https://docs.docker.com/compose/overview/)

#### 서버와 클라이언트의 버전이 다르다고 나올 때
아래와 같은 메세지가 나오는 경우가 있는데,
아래의 경우 클라이언트가 서버보다 새로운 버전이라 나는 에러이니
설치된 오래된 docker를 삭제하고 새로운 버전으로 다시 설치하자.
```
ERROR: client and server don't have same version (client : 1.21, server: 1.18)
```

아래의 메세지로 도커에 대한 정보를 알 수 있다.
```
$ docker version
$ docker info
```

### 보안을 위한 패키지 설치

`apache2-utils`에 있는 `htpasswd` 를 사용하기위한 패키지 다운로드

```
### kr.archive.ubuntu.com 에서는 패키지를 잘 못받아 오는 경우가 있으므로 변경해줌
$ sudo sed -i 's/kr.archive.ubuntu.com/ftp.daum.net/g' /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -y install apache2-utils
```

### Docker Registry 설치 및 설정하기

두개의 컨테이너를 연결할때 `docker-compose`를 사용할 것이다.
`docker-compose`는 `.yml`에 정의 된 내용을 설정 파일로 사용한다.

도커 레지스트리에서 사용할 디렉토리를 생성한다.
```
$ mkdir ~/docker-registry && cd $_
$ mkdir data
```

`docker-registry`를 만들기위한 `docker-compose.yml` 파일을 아래와 같이 생성한다.

```
registry:
    image: registry:2
    ports:
        - 127.0.0.1:5000:5000
    environment:
        REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
        - ./data:/data
```

아래의 명령어로 `docker-registry` 를 띄우자.
```
$ cd ~/docker-registry
$ sudo docker-compose up
```

아래와 같은 메세지들이 나온다.
```
Attaching to dockerregistry_registry_1
registry_1 | time="2016-02-19T02:34:50Z" level=warning msg="No HTTP secret provided - generated random secret. This may cause problems with uploads if multiple registries are behind a load-balancer. To provide a shared secret, fill in http.secret in the configuration file or set the REGISTRY_HTTP_SECRET environment variable." go.version=go1.5.3 instance.id=840ceb15-6c68-41b2-a793-967ee3974a20 version=v2.3.0
registry_1 | time="2016-02-19T02:34:50Z" level=info msg="redis not configured" go.version=go1.5.3 instance.id=840ceb15-6c68-41b2-a793-967ee3974a20 version=v2.3.0
registry_1 | time="2016-02-19T02:34:50Z" level=info msg="using inmemory blob descriptor cache" go.version=go1.5.3 instance.id=840ceb15-6c68-41b2-a793-967ee3974a20 version=v2.3.0
registry_1 | time="2016-02-19T02:34:50Z" level=info msg="listening on [::]:5000" go.version=go1.5.3 instance.id=840ceb15-6c68-41b2-a793-967ee3974a20 version=v2.3.0
registry_1 | time="2016-02-19T02:34:50Z" level=info msg="Starting upload purge in 55m0s" go.version=go1.5.3 instance.id=840ceb15-6c68-41b2-a793-967ee3974a20 version=v2.3.0
```

`No HTTP secret provided`라는 문구에 놀라지 말라고 한다. 정상이라고~

아무튼 저 메세지가 나오면 잘 된것이니 `Ctrl + C`를 눌러서 빠져 나가자.
`registry` 하나 올리려고 `docker-compose`를 깐게 아니다.


### Nginx 설정하기

```
$ mkdir ~/docker-registry/nginx
```

`docker-compose.yml`파일을 다시 열어서  아래의`nginx` 관련 설정을 추가하자.

```
nginx:
    image: "nginx:1.9"
    ports:
        - 5443:443
    links:
        - registry:registry
    volumes:
        - ./nginx/:/etc/nginx/conf.d
```

결과적으로 `docker-compose.yml`은 아래와 같은 형태가 된다.

```
nginx:
    image: "nginx:1.9"
    ports:
        - 5443:443
    links:
        - registry:registry
    volumes:
        - ./nginx/:/etc/nginx/conf.d
registry:
    image: registry:2
    ports:
        - 127.0.0.1:5000:5000
    environment:
        REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
        - ./data:/data
```

그런데 nginx의 설정파일이 빠져있으므로 추가하자.

#### nginx 설정파일 추가하기
아래의 경로에 설정파일을 추가하자.
```
$ vi ~/docker-registry/nginx/registry.conf
```

설정은 아래와 같다.
```
### registry.conf

upstream docker-registry {
    server registry:5000;
}

server {
    listen 443;
    server_name docker.gyus.me;

    client_max_body_size 0;
    chunked_transfer_encoding on;

    location /v2/ {
        ### docker 1.5 버전 이하에서 접속 안되게 함.
        if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
            return 404;
        }

        proxy_pass                            http://docker-registry;
        proxy_set_header Host                 $http_host;
        proxy_set_header X-Real-IP            $remote_addr;
        proxy_set_header X-Forwarded-For      $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto    $scheme;
        proxy_read_timeout                    900;
    }
}
```

`$ docker-compose up`으로 서버를 올릴 때 아래와 같은 에러가 났는데,

```
ERROR: failed to register layer: Untar re-exec error: unexpected EOF: output:
```

`sudo`로 권한을 높여주니 해결 됐다. 압축을 풀 권한이 없는 곳에 압축을 풀려다가 에러가 난것 같다.

```
$ sodo docker-compose up
```

`nginx` 랑 잘 연동이 되었는지 `curl`로 확인해보자.

```
$ curl http://localhost:5000/v2/
```

`{}` 로 결과 값이 나오면 성공이다.


https가 바인딩되는 포트로 사용될 `5443` 포트로도 잘 되는지 확인해 보자.

```
$ curl http://localhost:5443/v2/
```

역시나 `{}` 로 결과 값이 나오면 성공이다.

### 인증 설정하기

SSL만으로 불안한 사람은 이걸 하는게 좋은 것 같다.
나는 귀찮아서 사실 패스했다.

`htpasswd` 파일을 생성해보자.
```
$ cd ~/docker-registry/nginx
$ htpasswd -c registry.password <유저명>

### 패스워드를 입력해야한다. (안입력해도 되긴된다.)
New password:
Re-type new password:
```

`nginx`설정파일인 `registry.conf` 파일에 인증관련 설정을 추가해주자.
`location` 항목 아래에 설정해주어야한다.

```
location /v2/ {
    #### docker 1.5 버전 이하에서 접속 못하도록 막기.
    if ($http_user_agent ~ "^(docker\1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
        return 404;
    }

    #### 인증 설정
    auth_basic "registry.localhost";
    auth_basic_user_file /etc/nginx/conf.d/registry.password;
    add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;

    proxy_pass                              http://docker-registry;
    proxy_set_header Host                   $http_host;
    proxy_set_header X-Real-IP              $remote_addr;
    proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto      $scheme;
    proxy_read_timeout                      900;
}
```

### SSL 세팅하기

nginx 설정파일인 `~/docker-registry/nginx/registry.conf` 파일에 SSL 인증 관련 설정을 추가하자.
SSL인증을 위해서는 해당도메인의 인증서 파일인 `crt`파일과 `key`파일이 필요하다.

```
server {
    listen 443;
    server_name deploy.gyus.me;

    # SSL
    ssl on;
    ssl_certificate /etc/nginx/conf.d/domain.crt;
    ssl_certificate_key /etc/nginx/conf.d/domain.key;
```

인증서와 키파일은 [startssl](https://www.startssl.com) 사이트에서 공짜로 얻을 수도 있고,
다른 인증기관에서 구매도 가능하다.

임시적으로 사용하기 위해서 자가 인증서를 발급 하는 방법도 있는데, 아래에서 알아보자.

### 자가 인증서 발급하기

도커에서는 자가인증서만 가지고는, SSL를 사용할 수가 없고
자가 인증서를 인증해주는 자가 인증기관도 우리 서버에 세팅을 해야한다;;

자가인증기관에서 인증해줄수 있도록 루트 인증키를 만들자.
```
$ cd ~/docker-registry/nginx
### 루트 키를 만들자.
$ openssl genrsa -out rootCA.key 2048
### 루트 인증키를 만들자. (물어보는거는 대충 알아서 잘 적도록하자.)
$ openssl req -x509 -new -nodes -key rootCA.key -days 10000 -out rootCA.crt
```

이제 우리 서버의 인증서를 만들어보자.

```
### 키파일을 만든다.
$ openssl genrsa -out gyusme.key 2048
### 인증서 서명 요청 파일을 만듬 (csr)
$ openssl req -new -key gyusme.key -out gyusme.csr
### 인증서를 생성
$ openssl x509 -req -in gyusme.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out gyusme.crt -days 10000
```

인증서 서명 요청 파일 (csr)을 만들때에 이것 저것 물어보는데, 다른 건 몰라도 도메인은 잘 입력해주도록 하자.
그리고 패스워드를 물어보면 그냥 **엔터** 를 치자

```
Country Name (2 letter code) [AU]:KO
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:Busan
Organization Name (eg, company) [Internet Widgits Pty Ltd]:gyus,inc.
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:docker.gyus.me
Email Address []:
A challenge password []:
An optional company name []:
```

여기까지 했다면 `nginx`디렉토리에는 아래와 같은 파일들이 있을 것이다.

```
-rw-rw-r-- 1 ubuntu ubuntu 1229 Feb 19 17:29 gyusme.crt
-rw-rw-r-- 1 ubuntu ubuntu 1054 Feb 19 17:28 gyusme.csr
-rw-rw-r-- 1 ubuntu ubuntu 1679 Feb 19 17:26 gyusme.key
-rw-rw-r-- 1 ubuntu ubuntu  831 Feb 19 17:10 registry.conf
-rw-rw-r-- 1 ubuntu ubuntu 1269 Feb 19 17:25 rootCA.crt
-rw-rw-r-- 1 ubuntu ubuntu 1675 Feb 19 17:25 rootCA.key
-rw-rw-r-- 1 ubuntu ubuntu   17 Feb 19 17:29 rootCA.srl
```

우리의 서버가 인증기관인것 처럼 동작하게 하는 세팅을 해주자.

```
$ sudo mkdir /usr/local/share/ca-certificates/docker-cert
$ sudo cp rootCA.crt /usr/local/share/ca-certificates/docker-cert
$ sudo update-ca-certificates
```

### SSL 테스트하기

테스트를 해보려면, SSL 설정까지 마치고 에러없이 `docker registry`와 `nginx`가 올라가야 한다.
```
$ cd ~/docker-registry
$ docker-compose up
```

```
### 기본인증 사용X, 자가 인증서 X 인 경우
$ curl https://domain:5443/v2/

### 기본인증 사용 O, 자가 인증서 X 인 경우
$ curl https://USERNAME:PASSWORD@domain:5443/v2/

### 기본인증 사용 O, 자가 인증서 O 인 경우
$ curl https://USERNAME:PASSWORD@domain:5443/v2/ -k
```

### 도커 레지스트리 서비스로 등록하기
도커 레지스트리를 운영하는 서버를 재시작 할 경우가 생길 수 있는데,
이때에 필요한 프로세스를 일일이 찾아서 올리는거는 많은 정신력을 필요로 한다.

그런일을 당하지 않도록 서비스로 등록해보자.

`Upstart` 스크립트를 작성해서, 시스템이 부팅될때 시작되도록 해보자.

```
$ cd ~/docker-registry
### 지금 올라간 컨테이너를 삭제
$ docker-compose rm
$ sudo mv ~/docker-registry /docker-registry
$ sudo chown -R root: /docker-registry
```

`Upstart`스크립트를 작성해보자.

```
$ sudo vi /etc/init/docker-registry.conf
```

```
### docker-registry.conf
description "Docker Registry"

start on runlevel [2345]
stop on runlevel [016]

respawn
respawn limit 10 5

chdir /docker-registry

exec /usr/local/bin/docker-compose up
```

이제 `docker-registry`를 실행 시켜보자.
```
$ sudo service docker-registry start
```

아래와 같이 나오면 성공이다.
```
docker-registry start/running, process 27241
```

아래의 명령어로 또한 확인이 가능하다.
```
$ docker ps
```

로그는 아래의 명령어로 확인이 가능하다.

```
$ sudo tail -f /var/log/upstart/docker-registry.log
```

`curl` 명령어로 확인을 하면 `nginx_1`이 앞에 붙어서 로그가 찍힐 것이다.
```
$ curl https://USERNAME:PASSWORD@domain:5443/v2/
```

### 클라이언트 머신에서 docker registry 접근하기

다른 머신에서 도커 레지스트리에 접속하려면 루트인증서 파일을 해당 서버에 등록해주어야 한다.


```
$sudo cat /docker-registry/nginx/rootCA.crt
```

위의 명령어를 실행하면 아래와 같은 형식의 암호화 문자열이 나올것이다.
```
-----BEGIN CERTIFICATE-----
MIIDfTCCAmWgAwIBAgIJAKJf+hKlYzotMA0GCSqGSIb3DQEBCwUAMFUxCzAJBgNV
.... 중략...
ShmxzUJQAcQwdI2Wrivv9QG4kcRPw73lrMTNU8FfYnfQ
-----END CERTIFICATE-----
```

이걸 `docker registry`서버의 머신에서 해주었던 것 처럼 등록해주어야 한다.

#### 우분투의 경우

```
$ sudo mkdir /usr/local/share/ca-certificates/docker-dev-cert
$ sudo cp rootCA.crt /usr/local/share/ca-certificates/docker-dev-cert/
$ sudo update-ca-certificates
```

#### Centos의 경우

```
$ sudo cp rootCA.crt /etc/pki/ca-trust/source/anchors/
$ sudo update-ca-trust enable
$ sudo update-ca-trust extract
```

루트 인증서를 추가후 도커를 재시작해준다.

```
$ sudo service docker restart
```

인증을 사용하는 경우 `docker login`을 사용해야 하는데, 사용법은 아래와 같다.

```
$ docker login https://<domain>
```

`Username`과 `Password` `Email`을 물어보는데, `Email`은 그냥 아무것도 입력 하지 않으면 된다.

성공인 경우 아래와 같은 메세지가 뜬다.

```
Login Succeeded
```


### 이미지를 올려보자.

가장 쉬운 `hello-world`로 해보자.

```
$ sudo docker run hello-world
$ sudo docker tag hello-world deploy.gyus.me:5443/hello-world
$ sudo docker push deploy.gyus.me:5443/hello-world
```

위와 같은 스크립트를 실행시키면 성공시에 아래와 같은 메세지들이 나온다.

```
The push refers to a repository [deploy.gyus.me:5443/hello-world]
5f70bf18a086: Pushed
b652ec3a27e7: Pushed
latest: digest: sha256:fea8895f450959fa676bcc1df0611ea93823a735a01205fd8622846041d0c7cf size: 708
```


### 다른 머신에서 이미지를 땡겨와보자.
땡겨오는 법은 그냥 `push` 대신 `pull`을 쓰면 된다.
물론 인증서 작업은 완료된 상태이어야 한다.

```
$ docker pull deploy.gyus.me:5443/hello-world
```

```
$ docker run deploy.gyus.me:5443/hello-world
```

잘 실행된다!

```
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

### 느낀점

처음에는 좀 어렵다 귀찮다라는 느낌이 강했는데,
차근 차근 설치해보니, 크게 어려운 점은 없었다.

`https`로 통신을 해야하니 자가인증기관과 자가인증서를 만들어 주는 부분을
처음 해보는거라서 좀 이상하긴 했지만, 이 부분은 나중에 또 공부를 해야되는 부분은로
남겨두기로 한다.

### 참고 링크
아래의 링크를 대부분 참고했고, 따라하면서 내용을 조금씩 추가하거나 생략 하거나 했다.
[디지탈 오션](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04)
