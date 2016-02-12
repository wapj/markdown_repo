# 로컬에서 chef server 테스트하기

vagrant는 VagrantFile 하나로 여러대의 서버를 한번에 띄울 수 있다.
관련된 설정은 아래의 링크를 참고하자.

[vagrant 멀티머신](https://docs.vagrantup.com/v2/multi-machine/index.html)


### 서버5대를 한번에 띄우기 위한 VagrantFile세팅
```
Vagrant.configure(2) do |config|

  config.vm.define :chef_server do |cfg|
    cfg.vm.box = "cent7"
    cfg.vm.network :private_network, ip:"192.168.10.10"
    cfg.vm.host_name = "chef-server"
  end

  config.vm.define :workstation do |cfg|
    cfg.vm.box = "cent7"
    cfg.vm.network :private_network, ip:"192.168.10.20"
    cfg.vm.host_name = "workstation"
  end

  config.vm.define :node1 do |cfg|
    cfg.vm.box = "cent7"
    cfg.vm.network :private_network, ip:"192.168.10.21"
    cfg.vm.host_name = "node1"
  end

  config.vm.define :node2 do |cfg|
    cfg.vm.box = "cent7"
    cfg.vm.network :private_network, ip:"192.168.10.22"
    cfg.vm.host_name = "node2"
  end

  config.vm.define :node3 do |cfg|
    cfg.vm.box = "cent7"
    cfg.vm.network :private_network, ip:"192.168.10.23"
    cfg.vm.host_name = "node3"
  end

end
```

### ssh 접속하기

아래의 커맨드로 각각의 가상머신에 접속이 가능하다.
`vagrant ssh <호스트명>`

예를 들어 chef_server에 접속하려면 아래의 커맨드를 실행하면 된다.

`vagrant ssh chef_sesrver`

마찬가지로 workstaion 에 접속하려면 아래와 같이 하면된다.

`vagrant ssh workstation`

만약에 ssh로 접속을 하고 싶다면 ~/.ssh/config 에 설정을 등록해 주면 되는데 하는법은 다음과 같다.

`$ vagrant ssh-config >> ~/.ssh/config`

이렇게 하면 한번에 5개 서버의 정보가 한꺼번에 추가된다.

추후에는 `ssh <호스트명>`  으로 서버에 바로 접속이 가능하다.

