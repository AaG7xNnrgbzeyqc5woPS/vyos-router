# 参看:
    教程,源码都在这里:  
    https://github.com/vyos/vyos-build

----
# 一. 准备
----
## 1. 准备合适的虚拟机, 
    - linode上创建, 在北美,下载速度会比国内快很多. 
    - 首先在已有的vps上构建, git下载没有问题, 
    - 但是 docker build 出现问题,第一次搞这个,没有太大信心.
    - 此vps 是 centos 8, 看提示,有的组件说 不建议用 root用户. 
    - 重复几次还是 不行,换方案,新建一个 干净,更合适的 vps
    - 新建的vps 是 debain 8, Debian 8 "Jessie" 

## 2. 准备 vps Debian 8 "Jessie" 
    - linode 1 CPU, 1G内存, 20G SSD硬盘,
    - Debian 8, 使用  ssh key 登录
    - adduser user1
    - 授予 user1 sudo权限
    - $ usermod -aG sudo user1
    - 使用 user1 用户登录OK
    - 让 user1 使用 ssh key登录 ok
    
## 3. 安装一般工具
     - tmux, git,lsb_release
    
## 4. Set up the repository  
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

## 5. Install Docker CE
     $ sudo apt-get update
     $ sudo apt-get install docker-ce  
   
## 6. Verify that Docker CE
    $ sudo docker run hello-world
    $ sudo docker info
    $ sudo docker version
   
## 7. 普通用户也能运行 docker
    $ sudo addgoup docker
    $ usermod -aG docker $USER
    $ docker info
    
 
----- 
# 二. 建立vyos-builder容器
-----



## 8. 下载vyos-build源码
    $git clone https://github.com/vyos/vyos-build.git
    $cd vyos-build/
   
## 9. 建立vyos-builder images文件
    $tmux new -s vyos
    
    $pwd
    $/vyos-build/
    #确认下当前目录在 /vyos-build/目录下,没错的!
    
    $docker build -t vyos-builder docker
    # -- docker build 命令,使用dockerfile 建立一个容器的镜像文件(images)
    # -t vyos-builder -t参数指定生成后的images使用的tag, tag=vyos-builder, 
    #    就是 docker images 命令下看到的 REPOSITORY 名称
    # docker, 这里是 Dockerfile 的路径. Dockerfile名字永远不能修改,大小写也不能变.
    
    $ ....
    #等待超长时间,20分钟到几个小时,看机器速度
    ctrl+b d 退出当前 tmux会话.
    想看,登录到服务器后,用 
    $ tmux attach -t vyos
    $tmux ls #查看有多少个会话
    # ......
    
## 10. 查看 vyos-builder images 文件 
      $docker images
      REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
      vyos-builder        latest              bbc06f2b9618        39 minutes ago      4.33GB
      debian              buster              67e34c1c9477        3 weeks ago         114MB
      hello-world         latest              fce289e99eb9        11 months ago       1.84kB
      $
      可见vyos-builder的images已经建立好了!!!
      #第一个可观的成果!!!
      
 
## 11. 庆祝下第一个成果:
       $docker images
       vyos-builder 建立成功!
       
       
-----       
# 三.建VyOS的ISO文件
----

## 12. 运行新建的容器(vyos-builder)
       先确认下当前目录:
       $pwd
       $/vyos-build/
       #确认下当前目录在 /vyos-build/目录下,没错的!
    
       Run newly built container:
       $ docker run --rm -it --privileged -v $(pwd):/vyos -w /vyos vyos-builder bash
       # --rm 退出容器,自动删除容器
       # --it 交互方式进入容器内部,可以输入命令,cli
       # --privileged 必须使用此参数,  Give extended privileges to this container, 此容器需要扩展权限
       # -v $(pwd):/vyos, 当前的工作目录映射到 容器内部的 /vyos, 便于外部世界获取编译后的结果
       # -w /vyos, 指定容器内部用 /vyos做工作目录
       # vyos-builder  启动vyos-builder容器
       # bash 进入容器后自动执行 bash命令,进入命令行.
       
## 13. Building the ISO image
      环境确认
      $who
      $whoami
      $uname -r
      $uanme 
      $pwd   #应该是 /vyos
      $ls -l
      
      $./configure --help
      # Ok
      $./configure --custom-package vim --build-by jrandom@hacker.com
      # 这个命令居然开始出错了.说不认识这个命令, 从网页上拷贝的居然不对.是不是有非法字符呢,比如中文字符?
      #后来从 ./configure --help 这行拷贝了命令,再拼接起来,OK!
      
      $sudo make iso
      # Great Work is beginning! 
      # .....  wait long time.....
      # if ok then
  ### After some time you will find the resulting ISO image in the build folder.
  
  
## 14. 查看编译结果

       #编译结束, 非常好!
       $ls -l
       drwxr-xr-x 9 vyos_bld vyos_bld  4096 Dec 19 15:30 build
       可以看到多了一个build子目录 
       $ls build -al  
       lrwxrwxrwx  1 root     root            27 Dec 19 15:30 vyos-1.3-rolling-20191219
       这里可以发现 iso文件,这是是最新的滚动发布版.
       #vyos-1.3-rolling-20191219
       #时间是: 12月19日 15:30 UTC,换成北京时间,+8个小时,地球东方时间要早,所以时间跑得快些,数字大,用加法.
               12月19日 23:30 CST.北京时间
       #记得昨天也是11左右睡觉,所以构造ISO包也没有用多久.快!!!
       #服务器的速度还是很快很快的!
  
----
# 四.建立用户虚拟平台上的images文件
----

## 15. 为虚拟平台建立images
        Building images for virtualization platforms
        1. QEMU
        Run following command after building the ISO image.
        $ make qemu

        2. VMware
        Run following command after building the QEMU image.
        $ make vmware
       

## 16. 建立 QEMU images
       $ make qemu
        .....
        ==> Builds finished. The artifacts of successful builds are:
        --> qemu-image: VM files in directory: packer_build/qemu
       $ ls
        build      data    Jenkinsfile  Makefile      packer_cache  scripts
        configure  docker  LICENSE      packer_build  README.md     tools
        #可以发现多了一个 packer_build目录
        $ls packer_build
        build.log  qemu
        $ls packer_build/qemu -lh
        total 196K
        -rw-r--r-- 1 vyos_bld vyos_bld 193K Dec 20 00:46 vyos_qemu_image.img
        #vyos_qemu_image.img已经生成, OK
        #可见文件非常小,只有 193K,惊人的小,正确吗?
        
        
## 17. 建立 VMware images
      $pwd
      /yvos
      
      $ sudo make vmware #需要root权限
      ....
      Your system doesn't have vmdk-convert. Please install it from https://github.com/vmware/open-vmdk.
      make: *** [Makefile:61: vmware] Error 1
      
## 18. 安装 Open VMDK
     参见官方说明:
     https://github.com/vmware/open-vmdk
     
     Installation
        Firstly, you need to download and extract open-vmdk-master.zip and extract it:
        $ wget https://github.com/vmware/open-vmdk/archive/master.zip
        $ unzip master.zip

        Then, run below commands to build and install it:
        $ cd open-vmdk-master
        $ make
        $ sudo make install
        #注意最后一步要用 root权限, 不用sudo 执行会错误.
        
## 19. 再次建立 VMware images
      $cd ..        
      $pwd
      /yvos  #重要,必须是 /yvos目录下
      
      $sudo make vmware   #需要root权限
        Your system has vmdk-convert.
        Your system doesn't have ovftool. Please install it from https://www.vmware.com/support/developer/ovf/.
        make: *** [Makefile:61: vmware] Error 1
     # 注释:  还缺工具,下载页面去看了,需要注册才能下载,太麻烦.
     # 再考虑到 我也不用 VMware images,直接放弃吧!
     
## 20. 建立 VMware images失败,放弃!
      #建立失败!
      #苦海乌鸦,回头是岸!
        
-----     
# 五. 结束 
-----
       
       
## 15. The End!
   
   
