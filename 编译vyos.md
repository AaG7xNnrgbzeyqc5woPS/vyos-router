
**教程,源码都在这里:  
**https://github.com/vyos/vyos-build

# 1. 准备合适的虚拟机, 
    - linode上创建, 在北美,下载速度会比国内快很多. 
    - 首先在已有的vps上构建, git下载没有问题, 
    - 但是 docker build 出现问题,第一次搞这个,没有太大信心.
    - 此vps 是 centos 8, 看提示,有的组件说 不建议用 root用户. 
    - 重复几次还是 不行,换方案,新建一个 干净,更合适的 vps
    - 新建的vps 是 debain 8, Debian 8 "Jessie" 

# 2. 准备 vps Debian 8 "Jessie" 
    - linode 1 CPU, 1G内存, 20G SSD硬盘,
    - Debian 8, 使用  ssh key 登录
    - adduser user1
    - 授予 user1 sudo权限
    - $ usermod -aG sudo user1
    - 使用 user1 用户登录OK
    - 让 user1 使用 ssh key登录 ok
    
# 3. 安装一般工具
     - tmux, git,lsb_release
    
# 4. Set up the repository  
    Get Docker CE for Debian:  
    https://docs.docker.com/v17.12/install/linux/docker-ce/debian/#extra-steps-for-wheezy-77
   
    a. 卸载老版本
    sudo apt-get remove docker docker-engine docker.io
    
    b. $ sudo apt-get update
    c. $ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
    
    d. Add Docker’s official GPG key:
       $ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
       $ sudo apt-key fingerprint 0EBFCD88
       
    e.  $ sudo add-apt-repository \
           "deb [arch=amd64] https://download.docker.com/linux/debian \
           $(lsb_release -cs) \
           stable"

# 5. Install Docker CE
     $ sudo apt-get update
     $ sudo apt-get install docker-ce  
   
# 6. Verify that Docker CE
    $ sudo docker run hello-world
    $ sudo docker info
    $ sudo docker version
   
# 7. 普通用户也能运行 docker
    $ sudo addgoup docker
    $ usermod -aG docker $USER
    $ docker info
   
   
