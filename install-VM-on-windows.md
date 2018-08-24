# 使用Hyper-V安装虚拟机  #

创建虚拟交换网络

打开hyper-v管理器，点击虚拟交换机管理器，在新建虚拟交换机页面，连接类型选择外部。

下载centos7 的vagrant安装镜像  
地址 

	https://cloud.centos.org/centos/7/vagrant/x86_64/images/

下载名称中带有HyperV的box.


将下载好的box添加到vagrant的box列表


	vagrant box add box名称 下载好的box路径

比如 
	
	vagrant box add centos7-hyperv H:\vagrant-box\CentOS-7-x86_64-Vagrant-1804_02.HyperV.box  


查看box列表
	
	vagrant box list
	centos/7       (virtualbox, 1804.02)
	centos7-hyperv (hyperv, 0)


创建文件夹  

	mkdir centos7

创建vagrantfile文件

	touch Vagrantfile

Vagrantfile内容如下  

	ENV['VAGRANT_DEFAULT_PROVIDER'] = 'hyperv'
	
	Vagrant.require_version ">= 1.6.0"
	
	boxes = [
	    {
	        :name => "docker-host",
	        :mem => "1024",
	        :cpu => "1"
	    }
	]
	
	Vagrant.configure(2) do |config|
	
	  config.vm.box = "centos7-hyperv"
	    config.vm.define opts[:name] do |config|
	    config.vm.hostname = opts[:name]
	    config.vm.provider "hyperv" do |v|
	    	v.cpus=opts[:cpu]
	      	v.memory=opts[:mem]
	      	v.ip_address_timeout = 240
	      end
	      
	    end
	  end
	end


[Vagrantfile完整内容](./centos-hyperv/Vagrantfile)

启动 虚拟机 

	vagrant up  

停止虚拟机  

	vagrant halt

移除虚拟机  

	vagrant destroy 

注意 

	Hyper-V also requires that you execute Vagrant with administrative privileges. 
	Creating and managing virtual machines with Hyper-V requires admin rights. 
	Vagrant will show you an error if it does not have the proper permissions.