# docker_cluster_mysql
docker实现mysql数据库集群
Mysql一主多从配置





环境：
centos 7.0
tomcat 5.7

1.安装docker
安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
更新并安装 Docker-CE
sudo yum makecache fast
sudo yum install epel-release
sudo yum -y install docker-ce
开启Docker服务
sudo service docker start
设置docker存放路径
vi /usr/lib/systemd/system/docker.service  
ExecStart=/usr/bin/dockerd --graph /new-path/docker   //添加新存放路径（挂盘）
systemctl daemon-reload   // reload配置文件
systemctl restart docker.service // 重启docker
docker ps //测试 是否启动

2.	主从库配置：
解压文件docker_mysql.zip到主从路径/usr/local/docker 
路径格式：/usr/local/docker/slave2_docker_config
生成主mysql服务器：
docker run --name tp_mysql_master  -e MYSQL_ROOT_PASSWORD=111  -v /etc/localtime:/etc/localtime:ro   -v /usr/local/docker/master_docker_config:/etc/mysql/mysql.conf.d -p 3320:3306   mysql:5.7
*报错执行 docker rm tp_mysql_master;
   主库复制权限授权：

CREATE USER 'slave1'@'ip' IDENTIFIED BY 'slave1';//此处需要换成内网ip
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave1'@'ip';
FLUSH PRIVILEGES;
      查询主库的file和size
SHOW MASTER STATUS;//后面需要根据此来做同步

生成多从mysql服务器
slave2启动   docker run --name tp_mysql_slave_2  -e MYSQL_ROOT_PASSWORD=xxx -v /etc/localtime:/etc/localtime:ro   -v /usr/local/docker/slave2_docker_config:/etc/mysql/mysql.conf.d  -p 3323:3306   mysql:5.7
slave1启动   docker run --name tp_mysql_slave_1  -e MYSQL_ROOT_PASSWORD=XXX -v /etc/localtime:/etc/localtime:ro   -v /usr/local/docker/slave_docker_config:/etc/mysql/mysql.conf.d  -p 3322:3306   mysql:5.7  //端口不能重复


1
