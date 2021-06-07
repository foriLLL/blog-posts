* 关闭Selinux  
* 修改/etc/hostname  
* 修改/etc/hosts   
* 三台机器同步时间   ntpdate 每一分钟同步阿里时间  
* 三台机器添加普通用户  
* hadoop用户添加sudo权限？visudo -> hadoop ALL=(ALL) ALL  
* 定义三台linux服务器软件压缩包存放目录，以及解压后安装目录    
mkdir -p /foril/soft # 软件压缩包存放目录  
mkdir -p /foril/install # 软件解压后存放目录  
chown -R hadoop:hadoop /foril # 将文件夹权限更改为hadoop用户   
* 三台机器安装jdk   解压jdk.tar.gz  配置环境变量   
* 配置免密登录   
* 安装zookeeper并配置，配置集群id  
* zkServer.sh  
* 安装hadoop    **6个hadoop配置文件**   
* 创建文件存放目录  
* 配置环境变量
* 启动
* eclipse配置插件，远程hdfs

## 早上开机的步骤：
1. 打开虚拟机
2. 打开远程访问
3. JPS查询进程
4. 开启服务（HDFS,YARN,JOBHISTORYSERVER,ZOOKEEPER 目前是4个服务）