# phxsql安装配置

---

**机器准备 **

> `phxsql需求:PhxSQL需要在2台或以上的机器集群上运行（建议集群内机器数目n>=3 且 n为奇数）`

**初始化**

- `解压phxsql.tar.gz`

  - 命令：

    ```shell
    cd phxsql/tools
    ```

  - ```shell
    python2.7 install.py -i"your_inner_ip" -p 54321 -g 6000 -y 11111 -P 17000 -a 8001 -f/tmp/data/
    ```


- `加入集群`

  - 命令：

  - ```shell
    cd phxsql/sbin
    ```

  - ```shell
    ./phxbinlogsvr_tools_phxrpc -f InitBinlogSvrMaster -h"ip1,ip2,ip3" -p 17000
    ```

- `查看master初始化，进入phxsql执行读写确认phxsql正常运行`

  - 命令

    ```shell
    mysql -uroot -h"your_inner_ip" -P$phxsqlproxy_port
    ```

**简单测试**

- `cd phxsql/tools`
- `./test_phxsql.sh port ip1 ip2 ip3`，`port为phxsqlproxy_port`，`ip为集群集群ip`

**配置文件说明**

> phxsql一共有3个配置文件

###### 1. my.cnf： MySQL的配置,请根据你的业务需求进行修改（安装前请修改`tools/etc_template/my.cnf`，安装后请修改`etc/my.cnf`）

###### 2. phxbinlogsvr.conf

| Section name | Key name                   | comment                                                      |
| ------------ | -------------------------- | ------------------------------------------------------------ |
| AgentOption  | AgentPort                  | Phxbinlogsvr监听MySQL访问的端口，用于MySQL和binlogsvr之间的通信 |
|              | EventDataDir               | Phxbinlogsvr数据存放目录                                     |
|              | MaxFileSize                | Phxbinlogsvr每个数据文件的大小，数据文件过大会导致启动过慢，数据文件过小会导致文件数过多，单位为B |
|              | MasterLease                | Phxbinlogsv的master租约时间，单位为s                         |
|              | CheckPointTime             | Phxbinlogsvr会删除CheckPointTime时间前的数据，但如果被删数据中存在其他MySQL还没学到的，则不会删除该部分数据，单位为分钟 |
|              | MaxDeleteCheckPointFileNum | Phxbinlogsvr删数据时，每次删除的最大文件数                   |
|              | FollowIP                   | 机器为folloer机器，只负责拉取FollowIP机器上的数据，不参与集群的投票 |
| PaxosOption  | PaxosLogPath               | Phxbinlogsvr中paxos库的数据目录                              |
|              | PaxosPort                  | Phxbinlogsvr中paxos库的通信端口                              |
|              | PacketMode                 | Phxbinlogsvr在paxos协议中是否增大包的大小限制， 1为每个包的大小为100m，但超时限制变为1分钟，0 为每个包的大小为50m，超时限制2s起（动态变化） |
|              | UDPMaxSize                 | Phxpaxos默认使用udp和tcp混合的通讯方式，根据这个阈值觉得使用udp还是tcp。大小小于UDPMaxSize的消息会使用udp发送。 |
| Server       | IP                         | Phxbinlogsvr的监听ip                                         |
|              | Port                       | Phxbinlogsvr的监听端口                                       |
|              | LogFilePath                | Phxbinlogsvr的日志目录                                       |
|              | LogLevel                   | Phxbinlogsvr的日志级别                                       |

###### 3. phxsqlproxy.conf

| Section name | Key name               | comment                                                      |
| ------------ | ---------------------- | ------------------------------------------------------------ |
| Server       | IP                     | phxsqlproxy的监听ip                                          |
|              | Port                   | phxsqlproxy的监听端口                                        |
|              | QSLogFilePath          | phxsqlproxy的日志目录                                        |
|              | QSLogLevel             | phxsqlproxy的日志级别                                        |
|              | MasterEnableReadPort   | 是否打开master的只读端口。如果设为0，master会将只读端口的请求转发到一台slave上。 |
|              | TryBestIfBinlogsvrDead | 如果这个选项设为1，在本地phxbinlogsvr挂掉后，phxsqlproxy会重试从别的机器上的phxbinlogsvr获取master信息。 |

# **phxsql使用**

---

> `phxsqlproxy为PhxSQL的接入层，所有的请求均经过phxsqlproxy,再透传给MySQL`

### phxsqlproxy提供两个端口进行读写

##### 读写端口

该端口号为phxsqlproxy.conf配置中的端口号, 用户连接上proxyA的此端口，proxyA会自动把请求路由到Master机器，然后再对Master机器上的MySQL进行操作。

##### 只读端口

该端口号为读写端口号+1, 用户连接上此端口时，会对本机的MySQL进行操作(但若本机为master且phxsqlproxy.conf中设置了MasterEnableReadPort=0，则phxsqlproxy会把请求转发到其他phxsqlproxy的只读端口）。

### PhxSQL的使用

1. 通过执行命令`mysql -uroot -h$phxsqlproxyip -P$phxsqlproxyport -ppwd` 连接`phxsqlproxy`，
2. 进行sql命令操作

> phxsqlproxyip为集群内的任意一台phxsqlproxy的ip
>
> phxsqlproxyport为phxsqlproxy端口(读写/只读)

# PhxSQL管理

---

PhxSQL提供一个工具`phxbinlogsvr_tools_phxrpc`来方便管理者对PhxSQL的运营管理。

PhxSQL集群中对MySQL的管理使用两个账号管理员帐号和数据同步账号。管理员账号默认账号密码为（"`root`"，""），数据同步账号默认账号密码为（"`replica`","`replica123`"）。账号密码的更改通过工具才执行（工具会直接操作MySQL数据，不需要人工进行MySQL操作）。

### `phxbinlogsvr_tools -f GetMasterInfoFromGlobal -h <host> -p <port>`

**功能：**集群的master机器ip和超时时间

**参数：**

- **Host:** 集群中的其中一台机器ip
- **Port:** phxbinlogsvr的监听port

### `phxbinlogsvr_tools -f SetMySqlAdminInfo -h <host> -p <port> -u <admin username> -d <admin pwd> -U <new admin username> -D <new admin pwd>`

**功能：** 设置 PhxSQL管理员账号密码

**参数：**

- **Host:** 集群中的其中一台机器ip
- **Port:** phxbinlogsvr的监听port
- **Admin username:** 当前的管理员账号（默认为root）
- **Admin pwd:** 当前的管理员密码（默认为空）
- **New admin username:** 新的管理员账号
- **New admin pwd:** 新的管理员密码

### `phxbinlogsvr_tools -f SetMySqlReplicaInfo -h <host> -p <port> -u <admin username> -d <admin pwd> -U <new replica username> -D <new replica pwd>`

**功能：** 设置PhxSQL同步数据账号密码

**参数：**

- **Host:** 集群中的其中一台机器ip
- **Port:** phxbinlogsvr的监听port
- **Admin username:** 当前的管理员账号（默认为root）
- **Admin pwd:** 当前的管理员密码（默认为空）
- **New replica username:** 新的同步数据账号
- **New replica pwd:** 新的同步数据密码

### `phxbinlogsvr_tools_phxrpc -f GetMemberList -h <host> -p <port>`

**功能：**集群的所有机器信息

**参数：**

- **Host:** 集群中的其中一台机器ip
- **Port:** phxbinlogsvr的监听port

# phxbinlogsvr成员管理

### 移除成员

执行工具命令将机器A移除集群`phxbinlogsvr_tools_phxrpc -f RemoveMember -h<host> -p<port> -m <机器A的ip>`

执行成功后，机器A将会在一段时间后不在接收数据

### 添加成员

1. 执行工具将机器A加入到集群`phxbinlogsvr_tools -f AddMember -h<host> -p<port> -m <机器A的ip>`

2. 在新机器A上安装好PhxSQL

   > 根据 "`初始化PhxSQL`" 步骤安装PhxSQL

3. 执行成功后，机器A的phxbinlogsvr将会在一段时间后开始接收数据

4. 从集群内任意一台机器的percona导出一份镜像数据

5. kill掉机器A上的phxbinlogsvr, 并通过本地的MySQL端口进入MySQL，执行`set global super_read_only = 0; set global read_only = 0`;

6. 将镜像数据导入到机器A上的percona，并kill掉机器A上的phxbinlogsvr

7. 一段时间（~1分钟）后，机器A开始正常工作

# phxbinlogsvr故障处理

###### 当机器出现问题，PhxSQL无法正常启动时，可以选择在该机器上重装PhxSQL，重装过程可参考4.2

在重装过程中，`phxbinlogsvr`可能会拉取集群内其他机器`checkpoint`来启动，待`checkpoint`拉取结束后，`phxbinlogsvr`会自杀（为了确保数据安全），日志中会出现`"All sm load state ok, start to exit process"`，此时须重新启动`phxbinlogsvr`，启动后会正常运行。

###### 当MySQL数据出现问题时，phxbinlogsvr会停止工作，此时需要检查MySQL的binlog数据是否正常。

###### 当出现问题时，可观察日志中带有err标志的红色字体的日志，来确认是否有异常。