---
title: MySQL 组复制
date: 2019-01-30 12:24:39
categories: ["database"]
tags:
  - mysql
  - mgr
---

MySQL 组复制
<!--more-->

## 1. 部署组复制实例
假定MySQL Server已下载并解压缩到名为的目录中mysql-8.0。以下过程使用一台物理计算机，因此每个MySQL服务器实例都需要该实例的特定数据目录。在名为的目录中创建数据目录data 并初始化每个目录。

```
mkdir data
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s1
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s2
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s3
```

里面data/s1，data/s2， data/s3是一个初始化的数据目录，包含了MySQL系统数据库和相关表等等

>--initialize-insecure在生产环境中使用，它仅用于简化教程。

## 2. 配置组复制实例

### 组Replication Server设置
要安装和使用组复制插件，必须正确配置MySQL Server实例。建议将配置存储在实例的配置文件中。以下是组中第一个实例的配置，在此过程中称为s1。以下部分显示了示例服务器配置。

```
[mysqld]

# server configuration
datadir=<full_path_to_data>/data/s1
basedir=<full_path_to_bin>/mysql-8.0/

port=24801
socket=<full_path_to_sock_dir>/s1.sock
```

这些设置将MySQL服务器配置为使用先前创建的数据目录以及服务器应打开的端口并开始侦听传入连接。

>使用非默认端口24801是因为在本教程中，三个服务器实例使用相同的主机名。在具有三台不同机器的设置中，这不是必需的。

组复制需要成员之间的网络连接，这意味着每个成员必须能够解析所有其他成员的网络地址。例如，在本教程中，所有三个实例都在一台机器上运行，因此为了确保成员可以相互联系，您可以在选项文件中添加一行，例如 report_host=127.0.0.1。

### 复制框架
以下设置根据MySQL组复制要求配置复制。

```
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE
```

这些设置将服务器配置为使用唯一标识符编号1，以启用全局事务标识符，以允许仅执行可使用GTID安全记录的语句，以及禁用写入写入二进制日志的事件的校验和。

>如果您使用的是早于8.0.3的MySQL版本，其中默认值已针对复制进行了改进，则需要将这些行添加到成员的选项文件中。

```
log_bin=binlog
log_slave_updates=ON
binlog_format=ROW
master_info_repository=TABLE
relay_log_info_repository=TABLE
```

这些设置指示服务器打开二进制日志记录，使用基于行的格式，将复制元数据存储在系统表而不是文件中，并禁用二进制日志事件校验和。有关更多详细信息，请参见 第18.8.1节“组复制要求”。

### 组复制设置
此时，该my.cnf文件确保配置服务器并指示在给定配置下实例化复制基础结构。以下部分配置服务器的“组复制”设置。

```
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24901"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group=off
```

- 配置 transaction_write_set_extraction 指示服务器对于每个事务，它必须收集写集并使用XXHASH64散列算法将其编码为散列。从MySQL 8.0.2开始，此设置是默认设置，因此可以省略此行。
- 配置 group_replication_group_name 告诉插件它正在加入或创建的组被命名为"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"，group_replication_group_name的值 必须是有效的UUID。在二进制日志中为组复制事件设置GTID时，将在内部使用此UUID。使用SELECT UUID()生成一个UUID。
- 配置 group_replication_start_on_boot 指示插件在服务器启动时不自动启动操作。这在设置组复制时很重要，因为它确保您可以在手动启动插件之前配置服务器。配置成员后，您可以设置 group_replication_start_on_boot 为on，以便在服务器引导时自动启动Group Replication。
- 配置 group_replication_local_address 告诉插件使用网络地址127.0.0.1和端口24901与组中的其他成员进行内部通信。

>组复制将此地址用于涉及组通信引擎（XCom，Paxos变体）的远程实例的内部成员到成员连接。此地址必须与用于SQL的主机名和端口不同，并且不得用于客户端应用程序。在运行组复制时，必须为组成员之间的内部通信保留它。

配置的网络地址 group_replication_local_address 必须可由所有组成员解析。例如，如果每个服务器实例位于具有固定网络地址的其他计算机上，则可以使用计算机的IP地址，例如10.0.0.1。如果使用主机名，则必须使用完全限定名称，并确保它可以通过DNS解析，并且配置正确 /etc/hosts文件或其他名称解析过程。从MySQL 8.0.14开始，可以使用IPv6地址（或解析它们的主机名）以及IPv4地址。组可以包含使用IPv6的成员和使用IPv4的成员的混合。

建议的端口 group_replication_local_address 是33061。在本教程中，我们使用在一台计算机上运行的三个服务器实例，因此端口24901到24903用于内部通信网络地址。 group_replication_local_address Group Replication使用它作为复制组中组成员的唯一标识符。只要主机名或IP地址都不同，您就可以为复制组的所有成员使用相同的端口，并且如本教程所示，只要具有相同的主机名或IP地址，就可以使用相同的主机名或IP地址。港口都不一样。

- 配置 group_replication_group_seeds 设置组成员的主机名和端口，新成员使用它们建立与组的连接。这些成员称为种子成员。建立连接后，将列出组成员身份信息 performance_schema.replication_group_members。通常， group_replication_group_seeds 列表包含hostname:port每个组成员的列表 group_replication_local_address，但这不是强制性的，可以选择组成员的子集作为种子。

>该hostname:port列在 group_replication_group_seeds 是种子构件的内部网络地址，由被配置 group_replication_local_address ，而不是SQL hostname:port用于客户端连接，并且例如在显示 performance_schema.replication_group_members 表中。

启动该组的服务器不使用此选项，因为它是初始服务器，因此，它负责引导组。换句话说，引导该组的服务器上的任何现有数据都是用作下一个加入成员的数据。第二个服务器连接要求组中唯一的成员加入，第二个服务器上的任何缺失数据都从引导成员上的施主数据中复制，然后组扩展。加入的第三个服务器可以要求这两个服务器中的任何一个加入，数据被同步到新成员，然后该组再次扩展。

>在同时加入多个服务器时，请确保它们指向已在该组中的种子成员。不要使用也加入该组的成员作为种子，因为他们在联系时可能尚未加入该组。

最好首先启动引导程序成员，然后让它创建组。然后使其成为正在加入的其余成员的种子成员。这确保了在连接其余成员时形成的组。

不支持创建组并同时加入多个成员。它可能有效，但可能是操作竞争，然后加入该组的行为最终会出错或超时。

加入成员必须使用种子成员在group_replication_group_seeds 选项中通告的相同协议（IPv4或IPv6）与种子成员通信 。出于组复制的IP地址白名单的目的，种子成员上的白名单必须包含种子成员提供的协议的加入成员的IP地址，或者解析为该协议的地址的主机名。除了加入成员之外，还必须设置此地址或主机名并列入白名单 group_replication_local_address 如果该地址的协议与种子成员的通告协议不匹配。如果加入成员没有适当协议的白名单地址，则拒绝其连接尝试。有关更多信息，请参见 第18.5.1节“IP地址白名单”。

- 配置 group_replication_bootstrap_group 指示插件是否引导组。

>此选项只能在任何时候在一个服务器实例上使用，通常是第一次引导组时（或者在整个组关闭并重新备份的情况下）。如果多次引导组，例如当多个服务器实例设置了此选项时，则可以创建一个人工分裂脑情景，其中存在两个具有相同名称的不同组。在第一个服务器实例联机后禁用此选项。组中所有服务器的配置非常相似。您需要更改有关每个服务器的细节（例如server_id， datadir， group_replication_local_address）。本教程稍后将对此进行说明。


## 3. 用户凭证
组复制使用异步复制协议来实现 分布式恢复过程依赖于名为的复制通道group_replication_recovery，该通道用于将来自供体成员的事务转移到加入该组的成员。因此，您需要设置具有正确权限的复制用户，以便组复制可以建立直接的成员到成员恢复复制通道。

使用选项文件启动服务器：

```
mysql-8.0/bin/mysqld --defaults-file=data/s1/s1.cnf
```

使用该REPLICATION-SLAVE权限创建MySQL用户 。可以在二进制日志中捕获此过程，然后您可以依靠分布式恢复来复制用于创建用户的语句。或者，您可以禁用二进制日志记录，然后在每个成员上手动创建用户，例如，如果要避免将更改传播到其他服务器实例。要禁用二进制日志记录，请连接到服务器s1并发出以下语句：

```
mysql> SET SQL_LOG_BIN=0;
```

在以下示例中rpl_user，将password显示具有密码 的用户 。配置服务器时，请使用合适的用户名和密码。

```
mysql> CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
mysql> FLUSH PRIVILEGES;
```

如果禁用了二进制日志记录，则在创建用户后再次启用它。

```
mysql> SET SQL_LOG_BIN=1;
```

配置用户后，使用该 CHANGE MASTER TO语句将服务器配置为group_replication_recovery在下次需要从另一个成员恢复其状态时使用复制通道的给定凭据 。发出以下，替换 rpl_user和 password与创建用户时使用的值。

```
mysql> CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' \\
		      FOR CHANNEL 'group_replication_recovery';
```

分布式恢复是加入组的服务器采取的第一步，并且没有与组成员相同的事务集。如果没有为group_replication_recovery 复制通道正确设置这些凭据，并且rpl_user如图所示，则服务器无法连接到供体成员并运行分布式恢复过程以与其他组成员同步，因此最终无法加入该组。

同样，如果服务器无法通过服务器正确识别其他成员，hostname则恢复过程可能会失败。建议运行MySQL的操作系统具有正确配置的唯一 hostname，使用DNS或本地设置。这hostname可以Member_host在performance_schema.replication_group_members 表格的列中 进行验证 。如果多个组成员外部化hostname操作系统的默认 设置，则成员可能无法解析为正确的成员地址而无法加入该组。在这种情况下，使用report_host配置hostname由每个服务器外部化的唯一。

### 使用组复制和缓存SHA-2用户凭据插件
默认情况下，在MySQL 8中创建的用户使用 缓存SHA-2可插入身份验证。如果rpl_user您配置分布式恢复使用缓存SHA-2认证插件并没有使用安全套接字层支持（SSL） 的group_replication_recovery 复制通道，RSA密钥用于密码交换，创建SSL和RSA证书和密钥，您可以将rpl_user应该从组中恢复其状态的成员的公钥复制 到该组，也可以将捐赠者配置为在请求时提供公钥。

更安全的方法是将公钥复制 rpl_user到应该从捐赠者恢复组状态的成员。然后，您需要group_replication_recovery_public_key_path 在加入组的成员上配置 系统变量，并为其提供公钥的路径rpl_user。

可选地，不太安全的方法是设置 group_replication_recovery_get_public_key=ON 捐赠者，以便他们rpl_user在加入组时提供成员的公钥 。无法验证服务器的身份，因此只有group_replication_recovery_get_public_key=ON 在您确定没有服务器身份被泄露的风险时才会设置 ，例如通过中间人攻击。



## 4. 启动组复制
配置并启动服务器s1后，安装组复制插件。连接到服务器并发出以下命令：

```
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```

>mysql.session在加载组复制之前，用户必须存在。 MySQL 8.0.2版中添加了mysql.session。如果使用早期版本初始化数据字典，则必须运行 mysql_upgrade过程。如果未运行升级，则组复制无法以错误消息启动尝试使用用户访问服务器时出现错误：mysql.session@localhost。确保用户在服务器中，并且在服务器更新后运行了mysql_upgrade。

要检查插件是否已成功安装，请发出 SHOW PLUGINS;并检查输出。它应该显示如下：

```
mysql> SHOW PLUGINS;
+----------------------------+----------+--------------------+----------------------+-------------+
| Name                       | Status   | Type               | Library              | License     |
+----------------------------+----------+--------------------+----------------------+-------------+
| binlog                     | ACTIVE   | STORAGE ENGINE     | NULL                 | PROPRIETARY |

(...)

| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | PROPRIETARY |
+----------------------------+----------+--------------------+----------------------+-------------+
```

要启动该组，请指示服务器s1引导该组，然后启动组复制。此引导程序应仅由单个服务器完成，该服务器启动组并且只执行一次。这就是为什么bootstrap配置选项的值未保存在配置文件中的原因。如果它保存在配置文件中，则在重新启动时，服务器会自动引导具有相同名称的第二个组。这将导致两个具有相同名称的不同组。同样的推理适用于在此选项设置为的情况下停止并重新启动插件ON。

```
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;
```

一旦START GROUP_REPLICATION 语句返回，该集团已启动。您可以检查该组现在是否已创建，并且其中包含一个成员：

```
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| group_replication_applier | ce9be252-2b71-11e6-b8f4-00212844f856 | myhost      |       24801 | ONLINE        |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
```

此表中的信息确认组中有成员具有唯一标识符 ce9be252-2b71-11e6-b8f4-00212844f856，它正在ONLINE并且正在myhost 侦听端口上的客户端连接 24801。

为了证明服务器确实在一个组中并且它能够处理负载，创建一个表并向其添加一些内容。

```
mysql> CREATE DATABASE test;
mysql> USE test;
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL);
mysql> INSERT INTO t1 VALUES (1, 'Luis');
```

检查表的内容t1和二进制日志。

```
mysql> SELECT * FROM t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |   4 | Format_desc    |         1 |         123 | Server ver: 8.0.2-gr080-log, Binlog ver: 4                        |
| binlog.000001 | 123 | Previous_gtids |         1 |         150 |                                                                    |
| binlog.000001 | 150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 | 211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 | 270 | View_change    |         1 |         369 | view_id=14724817264259180:1                                        |
| binlog.000001 | 369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 | 434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 | 495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 | 585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 | 646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 | 770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 | 831 | Query          |         1 |         899 | BEGIN                                                              |
| binlog.000001 | 899 | Table_map      |         1 |         942 | table_id: 108 (test.t1)                                            |
| binlog.000001 | 942 | Write_rows     |         1 |         984 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 | 984 | Xid            |         1 |        1011 | COMMIT /* xid=38 */                                                |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
```

如上所示，创建了数据库和表对象，并将相应的DDL语句写入二进制日志。此外，数据已插入表中并写入二进制日志。当组成长并且新成员尝试赶上并联机时执行分布式恢复时，下一节将说明二进制日志条目的重要性。

## 5. 向组添加实例
此时，该组中有一个成员服务器s1，其中包含一些数据。现在是时候通过添加先前配置的其他两个服务器来扩展组。

### 添加第二个实例
为了添加第二个实例，服务器s2，首先为它创建配置文件。配置类似于用于服务器s1的配置，除了诸如数据目录的位置，s2将要监听的端口或其之外的配置 server_id。这些不同的行在下面的列表中突出显示。

```
[mysqld]

# server configuration
datadir=<full_path_to_data>/data/s2
basedir=<full_path_to_bin>/mysql-8.0/

port=24802
socket=<full_path_to_sock_dir>/s2.sock

#
# Replication configuration parameters
#
server_id=2
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE

#
# Group Replication configuration
#
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24902"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group= off
```

与服务器s1的过程类似，使用选项文件启动服务器。

```
mysql-8.0/bin/mysqld --defaults-file=data/s2/s2.cnf
```

然后按如下方式配置恢复凭据。这些命令与设置服务器s1时使用的命令相同，因为用户在组内共享。在s2上发布以下语句。

```
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password' \\
	FOR CHANNEL 'group_replication_recovery';
```

>如果您使用的是缓存SHA-2身份验证插件（MySQL 8中的默认设置），请参阅 使用组复制和缓存SHA-2用户凭据插件。

安装组复制插件并开始将服务器加入组的过程。以下示例以与部署服务器s1时相同的方式安装插件。

```
mysql> INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```

将服务器s2添加到组中。

```
mysql> START GROUP_REPLICATION;
```

与之前的步骤（与s1上执行的步骤相同）不同，此处的区别在于， 在启动组复制之前不会发出SET GLOBAL group_replication_bootstrap_group=ON;，因为该组已由服务器s1创建并引导。此时，只需将服务器s2添加到现有组中。

>当组复制成功启动并且服务器加入组时，它会检查 super_read_only变量。通过super_read_only 在成员的配置文件中设置为ON，可以确保因任何原因启动组复制时出现故障的服务器不接受事务。如果服务器应将该组作为读写实例加入，例如作为单主组中的主要组或多主组的成员，则当该 super_read_only变量设置为ON时，则在加入时将其设置为OFF群组。

performance_schema.replication_group_members 再次 检查 表显示该组中现在有两个 ONLINE服务器。

```
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 | myhost      |       24801 | ONLINE        |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 | myhost      |       24802 | ONLINE        |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
```

由于服务器s2也标记为ONLINE，它必须已经自动赶上服务器s1。验证它确实已与服务器s1同步，如下所示。

```
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+

mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         2 |         123 | Server ver: 8.0.3-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         2 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.000001 |  369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 |  434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=30 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT                                                             |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```

如上所示，第二台服务器已添加到组中，并自动从服务器s1复制了更改。根据分布式恢复过程，这意味着在加入组之后并且在被声明在线之前，服务器s2已经自动连接到服务器s1并从中获取丢失的数据。换句话说，它将事务从缺少的s1的二进制日志复制到它加入组的时间点。

### 添加其他实例
向组添加其他实例与添加第二个服务器的步骤顺序基本相同，只是必须更改配置，因为必须更改服务器s2。总结所需的命令：

1. 创建配置文件

```
[mysqld]

# server configuration
datadir=<full_path_to_data>/data/s3
basedir=<full_path_to_bin>/mysql-8.0/

port=24803
socket=<full_path_to_sock_dir>/s3.sock

#
# Replication configuration parameters
#
server_id=3
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE

#
# Group Replication configuration
#
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "127.0.0.1:24903"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
group_replication_bootstrap_group= off
```

2. 启动服务器

```
mysql-8.0/bin/mysqld --defaults-file=data/s3/s3.cnf
```

3. 配置group_replication_recovery通道的恢复凭据。

```
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='password'  \\
FOR CHANNEL 'group_replication_recovery';
```

4. 安装Group Replication插件并启动它。

```
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
START GROUP_REPLICATION;
```

此时，服务器s3已启动并正在运行，已加入该组并赶上该组中的其他服务器。performance_schema.replication_group_members 再次咨询 表证实了这种情况。

```
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 | myhost      |       24801 | ONLINE        |
| group_replication_applier | 7eb217ff-6df3-11e6-966c-00212844f856 | myhost      |       24803 | ONLINE        |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 | myhost      |       24802 | ONLINE        |
+---------------------------+--------------------------------------+-------------+-------------+---------------+
```

在服务器s2或服务器s1上发出相同的查询会产生相同的结果。此外，您可以验证服务器s3是否也赶上了：

```
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+

mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         3 |         123 | Server ver: 8.0.3-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         3 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.000001 |  369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 |  434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=29 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT                                                             |
| binlog.000001 | 1326 | Gtid           |         1 |        1387 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:6'  |
| binlog.000001 | 1387 | Query          |         1 |        1446 | BEGIN                                                              |
| binlog.000001 | 1446 | View_change    |         1 |        1585 | view_id=14724832985483517:3                                        |
| binlog.000001 | 1585 | Query          |         1 |        1650 | COMMIT                                                             |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```
