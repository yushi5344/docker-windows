# 在windows的虚拟机上安装docker #

**安装前关闭Hyper-V**

1. 下载安装VirtualBox和vagrant
1. 创建一个新的文件夹CentOS
1. 进入文件夹，打开ps   


	PS G:\CentOS7> vagrant init centos/7
	A `Vagrantfile` has been placed in this directory. You are now
	ready to `vagrant up` your first virtual environment! Please read
	the comments in the Vagrantfile as well as documentation on
	`vagrantup.com` for more information on using Vagrant.

此时，在CentOS文件夹下会有一个vagrantfile文件，然后

	PS G:\CentOS7> vagrant up

已经有vagrantfile的情况下，直接 vagrant up 


	boxes = [
	    {
	        :name => "docker-node1",
	        :eth1 => "192.168.205.10",
	        :mem => "1024",
	        :cpu => "1"
	    }
	]
	Vagrant.configure("2") do |config|
	  config.vm.box = "centos/7"
	  boxes.each do |opts|
	    config.vm.define opts[:name] do |config|
	      config.vm.hostname = opts[:name]
	      config.vm.provider "vmware_fusion" do |v|
	        v.vmx["memsize"] = opts[:mem]
	        v.vmx["numvcpus"] = opts[:cpu]
	      end
	      config.vm.provider "virtualbox" do |v|
	        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
	        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
	      end
	      config.vm.network :private_network, ip: opts[:eth1]
	    end
	  end
	end
进入虚拟机

	PS G:\CentOS7> vagrant ssh

开始安装docker
 安装之前先卸载旧版本

	sudo yum remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-selinux \
      docker-engine-selinux \
      docker-engine

安装requirements

	sudo yum install -y yum-utils \
	  device-mapper-persistent-data \
	  lvm2


添加repo

	sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
安装docker

	sudo yum install docker-ce


启动docker 

	[root@localhost vagrant]# service docker start

查看docker版本

	[root@localhost vagrant]# sudo docker version
	Client:
	 Version:           18.06.0-ce
	 API version:       1.38
	 Go version:        go1.10.3
	 Git commit:        0ffa825
	 Built:             Wed Jul 18 19:08:18 2018
	 OS/Arch:           linux/amd64
	 Experimental:      false
	
	Server:
	 Engine:
	  Version:          18.06.0-ce
	  API version:      1.38 (minimum version 1.12)
	  Go version:       go1.10.3
	  Git commit:       0ffa825
	  Built:            Wed Jul 18 19:10:42 2018
	  OS/Arch:          linux/amd64
	  Experimental:     false


关闭虚拟机

	PS G:\CentOS7> vagrant halt
	==> default: Attempting graceful shutdown of VM...

### docker-machine安装docker ###

使用docker-machine快速安装一个带有docker的虚拟机环境

	PS G:\CentOS7> docker-machine create docker-vm

docker-machine 的一些命令

	PS G:\CentOS7> docker-machine ls
	NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
	docker-vm   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.0-ce

进入虚拟机

	PS G:\CentOS7> docker-machine ls
	NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
	docker-vm   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.0-ce
	PS G:\CentOS7> docker-machine ssh docker-vm
	                        ##         .
	                  ## ## ##        ==
	               ## ## ## ## ##    ===
	           /"""""""""""""""""\___/ ===
	      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
	           \______ o           __/
	             \    \         __/
	              \____\_______/
	 _                 _   ____     _            _
	| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
	| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
	| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
	|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
	Boot2Docker version 18.06.0-ce, build HEAD : 1f40eb2 - Thu Jul 19 18:48:09 UTC 2018
	Docker version 18.06.0-ce, build 0ffa825

退出虚拟机

	docker@docker-vm:~$ exit
	exit status 127

停止docker

	PS G:\CentOS7> docker-machine stop docker-vm
	Stopping "docker-vm"...
	Machine "docker-vm" was stopped.
	PS G:\CentOS7> docker-machine ls
	NAME        ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
	docker-vm   -        virtualbox   Stopped                 Unknown

删除docker-machine创建的虚拟机

	PS G:\CentOS7> docker-machine rm docker-vm
	About to remove docker-vm
	WARNING: This action will delete both local reference and remote instance.
	Are you sure? (y/n): y
	Successfully removed docker-vm
	PS G:\CentOS7> docker-machine ls
	NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS
	PS G:\CentOS7>
