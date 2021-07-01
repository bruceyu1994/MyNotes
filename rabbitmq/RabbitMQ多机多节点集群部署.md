# RabbitMQ多机多节点集群部署

## 一、环境准备

准备三台安装好RabbitMQ 的机器
服务器分别为：

* 192.168.31.100
* 192.168.31.101
* 192.168.31.102



## 二、 修改配置文件

1. 改 10.10.1.41 机器上的 /etc/hosts 文件

   ```shell
   sudo vim /etc/hosts
   ```

2. 添加IP和节点名

   ```shell
   192.168.31.100 centos01
   192.168.31.101 centos02
   192.168.31.102 centos03
   ```

3. 修改对应主机的hostname

   ```shell
   hostname centos01
   hostname centos02
   hostname centos03
   ```

4. 将192.168.31.100上的hosts文件复制到另外两台机器上

   ```shell
   sudo scp /etc/hosts root@node2:/etc/
   sudo scp /etc/hosts root@node3:/etc/
   ```

   > 说明：命令中的root是目标机器的用户名，命令执行后，可能会提示需要输入密码，输入对应用 户的密码就行了

5. 将 192.168.31.100 上的 /var/lib/rabbitmq/.erlang.cookie 文件复制到另外两台机器上

   ```shell
   scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/
   scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/
   ```

   

## 三、防火墙添加端口

给每台机器的防火墙添加端口

1. 添加端口

   ```shell
   sudo firewall-cmd --zone=public --add-port=4369/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=5672/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=25672/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=15672/tcp --permanent
   ```

2. 重启防火墙

   ```shell
   sudo firewall-cmd --reload
   ```



## 四、启动RabbitMQ

1. 启动每台机器的RabbitMQ

   > PS: 每台机器都要重新启动

   ```shell
   sudo systemctl start rabbitmq-server
   ```

2. 将 192.168.31.101 加入到集群

   ```shell
   # 停止RabbitMQ 应用
   rabbitmqctl stop_app
   # 重置RabbitMQ 设置
   rabbitmqctl reset
   # 加入到集群
   rabbitmqctl join_cluster rabbit@centos01 --ram
   # 启动RabbitMQ 应用
   rabbitmqctl start_app
   ```

3. 查看集群状态，看到 running_nodes,[rabbit@node1,rabbit@node2] 表示节点启动成功

   ```shell
   rabbitmqctl cluster_status
   ```

   > 提示：在管理界面可以更直观的看到集群信息

4. 将 192.168.31.102 加入到集群

   ```shell
   # 停止 RabbitMQ 应用
   rabbitmqctl stop_app
   # 重置 RabbitMQ 设置
   rabbitmqctl reset
   # 节点加入到集群
   rabbitmqctl join_cluster rabbit@centos01 --ram
   # 启动 RabbitMQ 应用
   rabbitmqctl start_app
   ```

5. 重复地3步，查看集群状态



## 常见问题

1. 在将节点加入集群时，执行命令 `rabbitmqctl join_cluster rabbit@centos01 --ram` 报如下错误


![](https://gitee.com/brucefish/image-storage/raw/main/2021/06/30/RabbitMQ%E5%A4%9A%E6%9C%BA%E5%A4%9A%E8%8A%82%E7%82%B9%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%9801.png)



* 解决方案

  1. 确认每个主机的rabbitmq都已经重启并重置（`rabbitmqctl reset`）过

  2. 上一步骤执行后，若还是不行，将.erlang.cookie文件复制到每个服务器的用户目录下面（例如：若当前是root用户，就复制到/root目录下），再次重启每个主机上的rabbitmq，然后再确认加入集群是否正常

     

