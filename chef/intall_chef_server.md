# Chef Server 12를 사용해보기

![chef-server](http://gyus.me/wp-content/uploads/2016/02/chef-server.png)

2년전에 chef-server로 게임서버들을 잘 구성해서 사용했었다.
knife-solo로 하면 10대 정도까지는 그냥 관리가 가능하긴 한데,
role이나 databag같은 메타데이터를 다루게 되면,
그때부터는 `chef-server`를 사용하는 것도 좋은 것 같다.

2년 전에도 쉐프 서버 구축할 때 이리저리 고생을 했던 기억이 나는데,
정리를 제대로 안해놨더니, 했던 삽질을 다시하면서 나의 멍청함을 뇌로 되새기고 있다.

아무튼 다시는 그런일이 벌어지지 않게 하기 위해서,
좀 정성들여서 정리를 하는중이고, 특히 에러가 나는 포인트를 계속해서 정리중이다.

chef서버와 client는 virtual box를 사용해서 2대를 실행했고,
workstation은 mac에 chefdk를 설치해서 돌리고 있다.

사실 chef server 이외에는 chef client & knife가 깔리면 되는거라서, 뭘 어떻게 하든 크게 관계는 없을 것 같다.


### 기본지식
- chef-server-core만 인스톨한 후에는 chef-server-ctl커맨드로 reconfigure을 실행해주어야한다. 설정파일이 변경되면 reconfigure를 실행해야한다.
- chef-server-core관련 파일은 /etc/opscode에 있다.
- 로그는 /var/log/opscode에 있다
- chef서버의 로그를 실시간으로 보려면 `chef-server-ctl tail` 명령어를 사용하면 된다.


`chef server 12`에서는 여러가지 기능이 추가된 것 같긴한데, 기본만 설치하자.
- chef-server-core
- manage ui

### chef-server-core 설치

아래 링크에서 `chef-server-core_12.4.1-1_amd64.deb`를 받는다.
https://packagecloud.io/chef/stable/packages/ubuntu/trusty/chef-server-core_12.4.1-1_amd64.deb/download

아래 명령어를 실행

```
$ sudo dpkg -i chef-server-core_12.4.1-1_amd64.deb
$ sudo chef-server-ctl reconfigure
```

### user/org를 생성

```
### user생성
chef-server-ctl user-create user_name first_name last_name email password --filename ADMIN_PEM_FILE_NAME

### org생성
chef-server-ctl org-create short_name full_organization_name --association_user user_name --filename ORG_VALIDATION_KEY_FILE_NAME
```

/etc/opscode/admin.pem
/etc/opscode/orgname-validator.pem
등의 이름이 뭘하는 파일인지 알아보기 쉽다.

### chef manage ui 설치

이번 버전부터 ui가 엄청 이뿌게 변했다.
소스를 대충 보니 bootstrap을 사용한것 같은데, 개발자스럽긴하지만, 이쁜 UI이다.

```
$ sudo chef-server-ctl install opscode-manage
$ sudo opscode-manage-ctl reconfigure
```

## workstation 구축

chef를 설치

```
$ sudo curl -L https://www.opscode.com/chef/install.sh | sudo bash
```

[chefdk(chef development kit)](https://downloads.chef.io/chef-dk/)  라는 녀석을 설치해도 된다.

chefdk에는 `chef`, `Test Kitchen`, `ChefSpec`, `Foodcritic`, `Chef Client`, `Knife`, `Ohai`, `Chef Zero` 등이 함께 들어 있어서 쉐프 레시피개발 및 테스트에 좋다.

### knife 설정

```
$ knife configure
Where should I put the config file? [/home/USER/.chef/knife.rb]
Please enter the chef server URL: [https://xxx:443] https://chef-server/organizations/org
Please enter an existing username or clientname for the API: [user] <USER>
Please enter the validation clientname: [chef-validator] <ORG>-validator
Please enter the location of the validation key: [/etc/chef-server/chef-validator.pem] /Users/wapj/chef/.chef/<ORG>-validator.pem
Please enter the path to a chef repository (or leave blank): <CHEF-REPO-PATH>
```

위와 같이 하면 `.chef` 디렉토리 아래에 knife.rb 파일이 생긴다.
해당 파일은 아래와 같이 생겼다.
```
### knife.rb
log_level                :info
log_location             STDOUT
node_name                'wapj'
client_key               '/Users/wapj/chef/.chef/admin.pem'
validation_client_name   'treenod-validator'
validation_key           '/Users/wapj/chef/.chef/org-validator.pem'
chef_server_url          'https://wapj-VirtualBox/organizations/wapj'
syntax_check_cache_path  '/Users/wapj/chef/.chef/syntax_check_cache'
cookbook_path ["/Users/wapj/chef/site-cookbooks"]
```

`admin.pem`파일과 `org-validator.pem` 파일은 쉐프 서버에서 `scp` 명령어 등으로 잘 가져와서 해당디렉토리에 복사하자.


### chef서버와 잘 통신되는지 확인

아래와 같은 명령어로 확인이 가능하다.

```
$ knife ssh check
```

### trouble shooting

#### knife.rb의 chef_server_url설정에 아이피를 넣은 경우

아래와 같은 에러가 난다.
```
Connecting to host 192.168.10.111:443
ERROR: The SSL cert is signed by a trusted authority but is not valid for the given hostname
ERROR: You are attempting to connect to:   '192.168.10.111'
ERROR: The server's certificate belongs to 'wapj-VirtualBox'

TO FIX THIS ERROR:

The solution for this issue depends on your networking configuration. If you
are able to connect to this server using the hostname wapj-VirtualBox
instead of 192.168.10.111, then you can resolve this issue by updating chef_server_url
in your configuration file.

If you are not able to connect to the server using the hostname wapj-VirtualBox
you will have to update the certificate on the server to use the correct hostname.
```

이 경우 `/etc/hosts`에 server의 domain혹은 QFDN을 아래와 같은 식으로 넣어주어야 한다.

```
# hosts
192.168.10.111 wapj-VirtualBox
```

#### 아래와 같은 에러가 날 때
```
$knife status
ERROR: The object you are looking for could not be found
Response: <!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" />
  <title>Chef - 404 Not Found</title>
  <link media="all" rel="stylesheet" type="text/css" href="/css/all.css" />
  <!--[if lt IE 7]><link rel="stylesheet" type="text/css" href="/css/lt7.css" /><![endif]-->
</head>
<body>
  <div class="header-block">
    <div id="header">
      <strong class="logo"><a href="http://www.getchef.com">Chef</a></strong>
    </div>
  </div>
  <div id="wrapper">
    <div id="main">
      <div class="mybox">
        <div id="content">
          <h1>404 - Not Found</h1>
          <p>Sorry, I can't find what you are looking for.</p>
        </div>
      </div>
    </div>
  </div>
  <div class="footer-block">
    <div id="footer">
      <div class="mybox">
      </div>
      <div class="footer-bottom">
        <span>&copy; 2010&thinsp;&ndash;&thinsp;2014 Chef Software, Inc. All Rights Reserved</span>
      </div>
    </div>
  </div>
</body>
</html>
```


`knife.rb` 설정의 chef_server_url에 정말로 serverURL만 넣은 경우에 나는 에러이다.

아래와 같은 식으로 되어있다면,
```
chef_server_url https://wapj-VirtualBox/
```

뒤에 `organizations/<ORGANIGATION>`을 붙여주자.

```
chef_server_url https://wapj-VirtualBox/organizations/wapj
```

### 쿡북 만들기

아래 명령어로 hello라는 쿡북을 site-cookbooks에 만들었다.
레시피

```
$ knife cookbook create hello -o site-cookbooks
```

간단히 아래 파일을 변경하자.
```
site-cookbooks/hello/recipes/defau1t.rb
```

```
### default.rb
log "Hello , Chef!"
```

### 쿡북을 서버로 업로드하기
아래의 커맨드를 사용하면 모든 쿡북이 서버로 업로드 된다.
```
$ knife cookbook upload -a
```

하나의 쿡북만 업로드 하고 싶은 경우는 아래의 커맨드를 사용하자.

```
$ knife cookbook upload <쿡북이름> -o <해당쿡북의 폴더가 있는 경로>
$ knife cookbook upload hello -o ./site-cookbooks/
```


#### 쿡북 업로드시에 SSL 에러가 나는 경우
```
WARN: Found a directory hello in the cookbook path, but it contains no cookbook files. skipping.
ERROR: SSL Validation failure connecting to host: 192.168.10.111 - hostname "192.168.10.111" does not match the server certificate
```
자가 인증서로 검증하기 때문에 나는 당연한 에러(?!) 라서, 검증하지 말라고 하면 된다..;;

아래 한줄을 `knife.rb`에 추가해주자
```
 ssl_verify_mode :verify_none
 ```


## chef-client

### chef 설치

- 서버와의 인증에는 비밀키를 사용한다.
- chef server는 client별로 비밀키를 발행하고, 공개키를 chef-server에 등록한다.
- client별로 비밀키를 발행해야 하기 때문에, 미리 준비된 `validator-key`를 사용한다.
- client는 처음에는 `validator-key`로 통신해서 chef server에 client로 등록한후 이후의 통신에 필요한 비밀키가 발행된다.
- workstation에서 삽질했던것 처럼 `hosts`에 chef server의 ip를 등록해야한다.


### knife bootstrap으로 client 등록하기

**전제 조건**
1. `chef server`에서 `client`서버로 비밀번호 없이 root계정으로 로그인을 할 수 있어야한다.
2. bootstrap 을 실행할 서버의 /etc/hosts에 `chef server`의 ip를 등록한다.

1번 조건은 root의 홈디렉토리에 chef server의 퍼블릭키를 `authorized_keys`파일에 등록하면 된다.
2번은 위의 workstation 설정시에 보았던, `/etc/hosts`에 `chef server`의 ip를 등록하면 된다.

> chef server가 만약에 해킹당하게 되면, 모든 서버에 접속 할 수 있으므로 이 방법을 사용할 때에는,
> chef server의 보안에 특히 주의 해야한다.
> 나 같은 경우는 chef server를 사용할 때만 켰다가 끄는 방식으로 사용했다.

위의 조건이 만족되면, 아래의 커맨드로 바로 노드를 등록할 수 있다.
```
$ knife bootstrap <client server ip> -x <user> -N <node name>
```

성공하게 되면 아래와 같은 메세지가 나오게 된다.

```
### workstation의 chef repo 디렉토리에서 실행하면 된다.

$knife bootstrap 192.168.10.101 -x root -N node1
Doing old-style registration with the validation key at /Users/wapj/chef/treenod-chef/.chef/treenod-validator.pem...
Delete your validation key in order to use your user credentials instead

Connecting to 192.168.10.101
192.168.10.101 -----> Existing Chef installation detected
192.168.10.101 Starting the first Chef Client run...
192.168.10.101 Starting Chef Client, version 12.7.2
192.168.10.101 Creating a new client identity for node1 using the validator key.
192.168.10.101 resolving cookbooks for run list: []
192.168.10.101 Synchronizing Cookbooks:
192.168.10.101 Compiling Cookbooks...
192.168.10.101 [2016-02-16T21:02:32+09:00] WARN: Node node1 has an empty run list.
192.168.10.101 Converging 0 resources
192.168.10.101
192.168.10.101 Running handlers:
192.168.10.101 Running handlers complete
192.168.10.101 Chef Client finished, 0/0 resources updated in 02 seconds
```

### node가 등록되었는지 확인

```
### workstation의 chef repo에서 실행
$ knife node list
```

잘 등록되었다면 아래와 같이 나올 것이다.

```
node1
```


### node에 레시피 등록 하기

노드는 등록되었지만, 레시피가 아무것도 없으므로 실행해봐야 아무것도 안나온다.
레시피를 등록해보자.

해당 작업은 chef server의 WEB UI에서도 가능하므로, 웹사이트에 들어가서 해도 된다.

```
###  workstation의 chef repo에서 실행
$ knife node run_list add <노드명> 'recipe[레시피명]'
$ knife node run_list add node1 'recipe[hello]'
```

### node에서 레시피 실행하기

레시피의 실행은 각 노드 서버에 들어가서 `$ sudo chef-client` 를 실행해주면 되지만,
귀찮게 일일이 들어가면 굳이 서버를 설치한 이유가 없으니 서버에서 실행할 수 있는 방법을 찾아보자.

바로 `knife search node`를 이용하는 방법인데,

아래의 명령어를 실행하면, 조건을 줘서 chef client를 찾을 수 있다.
```
$ knife search node "fqdn:*"
```

그리고 조건은 `knife ssh`에도 동일하게 사용가능 한데, 조건과 조합하여 여러서버에 커맨드를 날릴수있다.
예를 들어 아래와 같이 실행하게 되면, 모든 서버에 `echo "hello"` 명령을 날리게 된다.
```
$ knife ssh "fqdn:*" 'echo "hello"'
```

이를 응용하여 아래와 같이 실행하게 되면, 모든 클라이언트에 `sudo chef-client`를 실행 할 수 있게 된다.

```
$ knife ssh "fqdn:*" "sudo chef-client"
```

#### hosts에 정보가 없는 경우
아래와 같은 에러가 난다.
```
WARNING: Failed to connect to node1 -- SocketError: getaddrinfo: nodename nor servname provided, or not known
```

이 경우는 workstation의 /etc/hosts에 node1의 ip를 등록해야한다.

```
### workstaions의 /etc/hosts
192.168.10.101 node1
```

다시 실행해보면~
```
$ knife ssh "fqdn:*" "sudo chef-client"
```

아직도 에러가 난다;;;
```
node1
node1 Starting Chef Client, version 12.7.2
node1 resolving cookbooks for run list: ["hello"]
node1 Synchronizing Cookbooks:
node1 [2016-02-16T21:45:15+09:00] ERROR: SSL Validation failure connecting to host: wapj-virtualbox - hostname "wapj-virtualbox" does not match the server certificate
node1 [2016-02-16T21:45:15+09:00] ERROR: SSL Validation failure connecting to host: wapj-virtualbox - hostname "wapj-virtualbox" does not match the server certificate
node1 [2016-02-16T21:45:15+09:00] ERROR: SSL Validation failure connecting to host: wapj-virtualbox - hostname "wapj-virtualbox" does not match the server certificate
node1   ================================================================================
node1   Error Syncing Cookbooks:
node1   ================================================================================
node1
node1   Unexpected Error:
node1   -----------------
node1   OpenSSL::SSL::SSLError: SSL Error connecting to https://wapj-virtualbox/bookshelf/organization-7479a4a25da1226f99983095c54052ba/checksum-40dcaffc3a2cdc8968a18f61672dab63?AWSAccessKeyId=49472f43a12b78c05ea083a9e82ac20de4ca4463&Expires=1455655515&Signature=ftp5IYQKkyvjyXYPG%2BlPltfh/18%3D - hostname "wapj-virtualbox" does not match the server certificate
node1   Running handlers complete
node1 [2016-02-16T21:45:40+09:00] ERROR: Exception handlers complete
node1   Chef Client failed. 0 resources updated in 27 seconds
node1 [2016-02-16T21:45:40+09:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
node1 [2016-02-16T21:45:40+09:00] ERROR: SSL Validation failure connecting to host: wapj-virtualbox - hostname "wapj-virtualbox" does not match the server certificate
node1 [2016-02-16T21:45:40+09:00] FATAL: Please provide the contents of the stacktrace.out file if you file a bug report
node1 [2016-02-16T21:45:40+09:00] ERROR: SSL Error connecting to https://wapj-virtualbox/bookshelf/organization-7479a4a25da1226f99983095c54052ba/checksum-40dcaffc3a2cdc8968a18f61672dab63?AWSAccessKeyId=49472f43a12b78c05ea083a9e82ac20de4ca4463&Expires=1455655515&Signature=ftp5IYQKkyvjyXYPG%2BlPltfh/18%3D - hostname "wapj-virtualbox" does not match the server certificate
node1 [2016-02-16T21:45:40+09:00] FATAL: Chef::Exceptions::ChildConvergeError: Chef run process exited unsuccessfully (exit code 1)
```

에러는 인증서가 매치가 안된다는 말인데, 자가 인증서이기 때문에 안되는 문제이다.

`/etc/chef/client.rb`에 `ssl_verify_mode :verify_none`을 또 추가해주자.

아래와 같은 형태로 `client.rb`가 되어 있으면 된다.
```
### node1 서버 client.rb
log_location     STDOUT
chef_server_url  "https://chef-server/organizations/treenod"
validation_client_name "treenod-validator"
node_name "node1"
trusted_certs_dir "/etc/chef/trusted_certs"
ssl_verify_mode :verify_none
```

다시 실행해보자
```
$ knife ssh "fqdn:*" "sudo chef-client"
```

아래와 같이 실행된다.
```
node1
node1 Starting Chef Client, version 12.7.2
node1 resolving cookbooks for run list: ["hello"]
node1 Synchronizing Cookbooks:
node1   - hello (0.1.0)
node1 Compiling Cookbooks...
node1 Converging 1 resources
node1 Recipe: hello::default
node1   * log[Hello Chef!] action write
node1
node1
node1 Running handlers:
node1 Running handlers complete
node1 Chef Client finished, 1/1 resources updated in 01 seconds
```

개발PC에서 `cookbook`을 만들어서 올리고, chef의 API를 사용해서 실행하는 것을 보여주기위해
workstation이라는 것을 예로 들었지만, workstation을 굳이 만들지 않아도 되고, `chef server`에 `chef`를 설치해서 하는 것이 더 간단한 방법일 수도 있다.
개념을 파악하고 구조를 파악해서, 각자의 환경에 맞게 잘 사용하면 될 것 같다.

좀 길었지만, chef server 설치중에 내가 만난 문제들은 다 정리가 된것 같다.
몇가지 수동으로 했던 부분들을 자동화하게 되면, 훌륭한 인프라 툴이 될 것이다.
