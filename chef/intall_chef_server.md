# 쉐프서버 인스톨하기

아래 페이지에서 `chef-server-core_12.4.1-1_amd64.deb`를 받는다.
https://downloads.chef.io/chef-server/ubuntu/

### 설치

```
$ sudo dpkg -i chef-server-core_12.4.1-1_amd64.deb
```

### 설정

```
$ sudo chef-server-ctl reconfigure
```

### 관리자추가

```
$ chef-server-ctl user-create USERNAME   FIRST_NAME LAST_NAME EMAIL PASS --filename FILE_NAME
$ chef-server-ctl user-create stevedanno Steve Danno steved@chef.io 'abc123' --filename /path/steve_private_key.pem
```

### 추가기능 설치

```
$ chef-server-ctl install opscode-manage
```
