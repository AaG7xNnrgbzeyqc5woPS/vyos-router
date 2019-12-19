
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
    
# 8. 下载vyos-build源码
    $git clone https://github.com/vyos/vyos-build.git
    $cd vyos-build/
   
# 9. 建立vyos-builder images文件
    $tmux new -s vyos
    
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
    
# 10. 查看 vyos-builder images 文件 
      $docker images
      REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
      vyos-builder        latest              bbc06f2b9618        39 minutes ago      4.33GB
      debian              buster              67e34c1c9477        3 weeks ago         114MB
      hello-world         latest              fce289e99eb9        11 months ago       1.84kB
      $
      可见vyos-builder的images已经建立好了!!!
      #第一个可观的成果!!!
      
 # 11. 庆祝下第一个成果:
       $docker images
       vyos-builder 建立成功!
       
 # 12. 运行新建的容器(vyos-builder)
       Run newly built container:
       $ docker run --rm -it --privileged -v $(pwd):/vyos -w /vyos vyos-builder bash
       # --rm 退出容器,自动删除容器
       # --it 交互方式进入容器内部,可以输入命令,cli
       # --privileged 必须使用此参数,  Give extended privileges to this container, 此容器需要扩展权限
       # -v $(pwd):/vyos, 当前的工作目录映射到 容器内部的 /vyos, 便于外部世界获取编译后的结果
       # -w /vyos, 指定容器内部用 /vyos做工作目录
       # vyos-builder  启动vyos-builder容器
       # bash 进入容器后自动执行 bash命令,进入命令行.
       
# 13. Building the ISO image
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
       
     The End!
   
   
