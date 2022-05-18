# Vagrant 학습

## What is Vagrant?

- 단일 작업 환경 내에서 가상 머신 환경을 빌드 및 관리하기 위한 도구입니다.

## Vagrant Install

- 해당 사이트 (https://www.vagrantup.com/downloads) 에서 다양한 OS 에서 설치할 수 있습니다.

## Initialize

- 작업 환경 생성

```shell
$ mkdir vagrant_getting_started
$ cd vagrant_getting_started

$ vagrant init hashicorp/bionic64 // 해당 명령어 입력시 hasicorp/bionic64 이미지 기반의 Vagrantfile 이 생성됩니다.
```

## Box

- Vagrant 에서 Box 는 템플릿 이미지와 동일한 역할을 합니다.

```shell
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_version = "1.0.282"
end
```

- 더 많은 Box 들을 확인하려면 해당 사이트 (https://vagrantcloud.com/boxes/search) 를 통해 확인할 수 있습니다.

## Boot an Environment

```shell

$ vagrant up // 가상 머신 생성

$ vagrant ssh $HOST_NAME // 가상 머신 SSH 접속

$ vagrant destroy // 가상 머신 삭제

```

## Synchronize Local and Guest Files

```shell
$ vagrant up

$ vagrant ssh $HOST_NAME

$ ls /vagrant
Vagrantfile

$ touch /vagrant/foo
$ exit

$ ls /vagrant
Vagrantfile foo


```

- Vagrantfile 이 있는 폴더의 경우 호스트와 가상 머신의 공유 폴더로 지정이 됩니다.
- 호스트 및 가상 머신 내에서 해당 폴더에 작업을 하는 경우 서로의 작업이 공유됩니다.

## Configure Network

```shell
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, guest: 80, host: 4567
end

$ vagrant reload
```

- config.vm.network :forwarded_port 기능을 이용하여 호스트와 가상 머신간의 포트 포워딩이 가능합니다.

```shell
Vagrant.configure("2") do |config|
  # ...
  config.vm.hostname = "myhost.local"
  config.vm.network "public_network", ip: "192.168.0.1", hostname: true
  config.vm.network "public_network", ip: "192.168.0.2"
end
```

- hostname 설정 및 public or private network 추가가 가능합니다.

```shell
Vagrant.configure("2") do |config|
  config.vm.network "private_network", type: "dhcp"
end

Vagrant.configure("2") do |config|
  config.vm.network "private_network", ip: "192.168.50.4"
end
```

- Private/Public Network 의 경우 dhcp 또는 static 으로 IP 설정이 가능합니다.

### Default Router

```shell
Vagrant.configure("2") do |config|
  config.vm.network "public_network", ip: "192.168.0.17"

  # default router
  config.vm.provision "shell",
    run: "always",
    inline: "route add default gw 192.168.0.1"

  # default router ipv6
  config.vm.provision "shell",
    run: "always",
    inline: "route -A inet6 add default gw fc00::1 eth1"

  # delete default gw on eth0
  config.vm.provision "shell",
    run: "always",
    inline: "eval `route -n | awk '{ if ($8 ==\"eth0\" && $2 != \"0.0.0.0\") print \"route del default gw \" $2; }'`"
end
```

- Vagrant 의 경우 가상 머신 생성시 디폴트로 NAT 네트워크 인터페이스를 하나 가지고 있습니다.
- 그렇기 때문에 새로운 default router 를 설정하기 위해서는 route add 명령어를 사용해주어야 하며, 기존 default router (NAT) 를 제거하기 위해서는 route del 명령어를 이용하여 제거해주어야 합니다.
