# Docker Images Pusher

使用Github Action将国外的Docker镜像转存到阿里云私有仓库，供国内服务器使用，免费易用<br>
- 支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库<br>
- 支持最大40GB的大型镜像<br>
- 使用阿里云的官方线路，速度快<br>

视频教程：https://www.bilibili.com/video/BV1Zn4y19743/



## 使用方式
使用教程：https://blog.csdn.net/qq_45152962/article/details/140693432

### 配置阿里云
登录阿里云容器镜像服务<br>
https://cr.console.aliyun.com/<br>
启用个人实例，创建一个命名空间（**ALIYUN_NAME_SPACE**）
![](/doc/命名空间.png)

访问凭证–>获取环境变量<br>
用户名（**ALIYUN_REGISTRY_USER**)<br>
密码（**ALIYUN_REGISTRY_PASSWORD**)<br>
仓库地址（**ALIYUN_REGISTRY**）<br>

![](/doc/用户名密码.png)


### Fork本项目
Fork本项目<br>
#### 启动Action
进入您自己的项目，点击Action，启用Github Action功能<br>
#### 配置环境变量
进入Settings->Secret and variables->Actions->New Repository secret
![](doc/配置环境变量.png)
将上一步的**四个值**<br>
ALIYUN_NAME_SPACE,ALIYUN_REGISTRY_USER，ALIYUN_REGISTRY_PASSWORD，ALIYUN_REGISTRY<br>
配置成环境变量

### 添加镜像
打开images.txt文件，添加你想要的镜像 
可以加tag，也可以不用(默认latest)<br>
可添加 --platform=xxxxx 的参数指定镜像架构<br>
可使用 k8s.gcr.io/kube-state-metrics/kube-state-metrics 格式指定私库<br>
可使用 #开头作为注释<br>
![](doc/images.png)
文件提交后，自动进入Github Action构建

### 使用镜像
回到阿里云，镜像仓库，点击任意镜像，可查看镜像状态。(可以改成公开，拉取镜像免登录)
![](doc/开始使用.png)

在国内服务器pull镜像, 例如：<br>
```
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/alpine
```
registry.cn-hangzhou.aliyuncs.com 即 ALIYUN_REGISTRY(阿里云仓库地址)<br>
shrimp-images 即 ALIYUN_NAME_SPACE(阿里云命名空间)<br>
alpine 即 阿里云中显示的镜像名<br>

### 多架构
需要在images.txt中用 --platform=xxxxx手动指定镜像架构
指定后的架构会以前缀的形式放在镜像名字前面
![](doc/多架构.png)

### 镜像重名
程序自动判断是否存在名称相同, 但是属于不同命名空间的情况。
如果存在，会把命名空间作为前缀加在镜像名称前。
例如:
```
xhofe/alist
xiaoyaliu/alist
```
![](doc/镜像重名.png)

### 定时执行
修改/.github/workflows/docker.yaml文件
添加 schedule即可定时执行(此处cron使用UTC时区)
![](doc/定时执行.png)

# 下面是自动化工作流的执行流程
## 在项目中，有一个关键的文件 images.txt，它记录着所需的镜像信息，当修改项目中的images.txt文档，并且推送到main分支时,会自动触发githubaction工作流操作。环境变量有：阿里云镜像仓库地址、阿里云的命名空间、阿里云账号用户名、阿里云账号密码。拉取流程
### 1.环境清理：
#### 整个操作在 github 网站提供的 Ubuntu 服务器系统上执行。触发操作后，第一步便是对 Ubuntu 环境进行清理通过清理临时文件、删除不再使用的旧镜像和容器等操作，确保服务器有充足的资源来处理新的镜像拉取任务，避免因空间不足导致操作失败。
### 2.镜像清单获取：
#### 将存放在 Github 仓库中的 images.txt（镜像列表）复制到服务器上服务器通过读取这个文件，便能清晰地获取到需要处理的镜像清单
### 3.镜像处理与推送：
#### 根据获取到的镜像清单，服务器开始进行镜像的拉取。拉取完成后，为了符合阿里云镜像仓库的存储规范或便于管理，会对镜像进行改名操作 。接着，将改名后的镜像推送到阿里云镜像仓库，完成中转存储。当本地 K8s 集群需要这些镜像时，便可从阿里云仓库中快速拉取。最后，为了节省服务器空间，删除 ubuntu 的本地镜像文件，保持服务器的整洁与高效 。
