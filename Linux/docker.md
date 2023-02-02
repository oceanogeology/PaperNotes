# Docker骚操作集锦

Docker学习教程： https://www.runoob.com/docker/docker-hello-world.html ；https://docs.docker.com/engine/reference/commandline/images/；http://doc.hz.netease.com/pages/viewpage.action?pageId=261792775

### 1. Docker 常用命令

```
# 容器里面查看容器id
cat /proc/self/cgroup | head -1
```



```
docker run --name='dk_debug' --gpus='"device=0,1,2,3"' --shm-size 50g \
    -v /nas:/nas -v /tmp:/tmp -v /root:/root --network=host \
    --rm -it hobby-registry-vpc.cn-hangzhou.cr.aliyuncs.com/prod/ai_label_platform_v2_multimodal:latest bash

```



```
docker run --name=duankui-time0 --cpus=20 --cpuset-cpus="1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20" --shm-size=32g --network=host -v /home/duankui/wy-cv-duankui/:/app -v /mnt:/mnt -v /data:/data --rm -it duankui:cuda10.0-cudnn7-mxnet1.5-torch1.8-py3.8.5-cv2-ssh-req-gjiajia-htop bash
```

```
docker run --name=duankui-train1 --cpus=20 --cpuset-cpus="1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20" --shm-size=32g --network=host -v /home/duankui/wy-cv-duankui/:/app -v /mnt:/mnt -v /data:/data --rm -it duankui:cuda10.0-cudnn7-mxnet1.5-torch1.8-py3.8.5-cv2-ssh-req-gjiajia-htop bash
```

### 2.建立镜像步骤

```
step1：编写Dockerfile
    FROM cuda10.0-cudnn7-mxnet1.5-torch1.2-trt5-torch2trt-tvm-py3-cv2:latest
    COPY requirements.txt requirements.txt
    RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
    RUN for req in $(cat requirements.txt); do pip install --no-cache-dir --upgrade -i https://pypi.tuna.tsinghua.edu.cn/simple $req ; done
    RUN rm requirements.txt
    WORKDIR "/app"
step2：编写sh建立镜像脚本
    （方法1）docker build --network=host -t duankui:cuda10.0-cudnn7-mxnet1.5-torch1.2-trt5-torch2trt-tvm-py3-cv2 .
    （方法2）repo=duankui/base
        tag="cuda100-cudnn7-tensorrt7-devel-ubuntu1604-mxnet180-torch160-py38-cv2"
        full=$repo:$tag
        echo "building $full"
        docker build --network=host -t $full .
        echo "build $full completed"
        # docker login hub.c.163.com/neteaseis/
        # docker push $full
        echo "done."
```

### 3.链接镜像

```
（方法1）docker run -v /mnt:/mnt --rm --name duankui-test --network=host -it cuda10.0-cudnn7-mxnet1.5-torch1.2-trt5-torch2trt-tvm-py3-cv2:latest bash
（方法1）docker run --name=duankui-train1 --cpus=20 --cpuset-cpus="1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20" --shm-size=32g --network=host -v /home/duankui/wy-cv-duankui/:/app -v /mnt:/mnt -v /data:/data --rm -it duankui:cuda10.0-cudnn7-mxnet1.5-torch1.8-py3.8.5-cv2-ssh-req-gjiajia-htop bash
（方法2）
先编写docker-compose.yml
    version: '2'
    services:
      duankui_bighead:
        build: .
        # image: efficient-head-model:latest
        image: duankui:cuda10.0-cudnn7-mxnet1.5-torch1.2-trt5-torch2trt-tvm-py3-cv2
        network_mode: host
        volumes:
        - .:/app
        - /mnt/:/mnt
再运行run.sh
    docker-compose run --rm duankui_bighead bash 
```

### 4.docker 与vscode链接

 https://blog.csdn.net/u010099080/article/details/104801858
修改ubuntu镜像源： https://blog.csdn.net/zgljl2012/article/details/79065174；https://mirrors.ustc.edu.cn/help/ubuntu.html

```
step1：passwd root
step2：更换ubuntu镜像源
    1, 将下面所有放到一个sources.list中
        # 默认注释了源码仓库，如有需要可自行取消注释
        deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
        # 预发布软件源，不建议启用
        # deb https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
    2，然后进入docker里面之后【docker中不要把/etc进行映射】 cp sources.list  /etc/apt/ 
step3：apt-get update
step4：apt-get install -y --no-install-recommends openssh-server
step5：sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
step6：启动ssh服务
    1，修改/etc/ssh/sshd_config，copy下面的附件到容器中的sshd_config
    2，service ssh restart
    3，将C:\Users\duankui\.ssh\config 拖入vscode中（保证容器中port【下图1】和本地port【下图2】一致）
step7：这时vscode中点击远程，会出现一个test为名字的远程端口，点击下面的文件，需要输入密码，密码就是step1设置的。

（***）每次断开终端的docker，要重新链接的时候，需要重新配置一下本地的known_hosts文件【下图3】，删除端口密钥。然后重新连接。

```

```
其他注意：
需要安装python 和 需要生成launch.json 才能有debug功能；
点击下面的环境，可以选择容器中python的执行环境
```

![image-20211208](..\images\20211208.png)

![image-202112081](..\images\202112081.png)

![image-202112082](..\images\202112082.png)

