---
title: OpenLDAP 安装指南
date: 2019-12-27
categories:
  - linux
tags:
  - ldap
---

CentOS 7 OpenLDAP 安装配置指南
<!--more-->


## 1. 安装 openldap

```bash
[root@ldap ~]# yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools
```

### 查看版本
```bash
[root@ldap ~]# slapd -VV
@(#) $OpenLDAP: slapd 2.4.44 (Jan 29 2019 17:42:45) $
	mockbuild@x86-01.bsys.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
```

### 为默认用户设置密码
```bash
[root@ldap ~]# slappasswd
New password:
Re-enter new password:
{SSHA}P+NaHJ1IQAM/17rDCIK8i2Klm3rStDVo
```

## 2. 配置 openldap
### 修改 `olcDatabase={2}hdb.ldif` 文件

```bash
[root@ldap ~]# vim /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb.ldif
```

```bash
# AUTO-GENERATED FILE - DO NOT EDIT!! Use ldapmodify.
# CRC32 2df1c260
dn: olcDatabase={2}hdb
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: {2}hdb
olcDbDirectory: /var/lib/ldap
# olcSuffix: dc=my-domain,dc=com
olcSuffix: dc=example,dc=com
# olcRootDN: cn=Manager,dc=my-domain,dc=com
olcRootDN: cn=Manager,dc=example,dc=com
olcDbIndex: objectClass eq,pres
olcDbIndex: ou,cn,mail,surname,givenname eq,pres,sub
structuralObjectClass: olcHdbConfig
entryUUID: 7b00e3c0-bcc6-1039-927c-c144385599c7
creatorsName: cn=config
createTimestamp: 20191227073020Z
entryCSN: 20191227073020.070906Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20191227073020Z
# add
olcRootPW: {SSHA}P+NaHJ1IQAM/17rDCIK8i2Klm3rStDVo
```

### 修改 `olcDatabase={1}monitor.ldif` 文件

```bash
[root@ldap ~]# vim /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{1\}monitor.ldif
```

```bash
# AUTO-GENERATED FILE - DO NOT EDIT!! Use ldapmodify.
# CRC32 cdc0bae3
dn: olcDatabase={1}monitor
objectClass: olcDatabaseConfig
olcDatabase: {1}monitor
# olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=extern
#  al,cn=auth" read by dn.base="cn=Manager,dc=my-domain,dc=com" read by * none
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=extern
 al,cn=auth" read by dn.base="cn=Manager,dc=example,dc=com" read by * none
structuralObjectClass: olcDatabaseConfig
entryUUID: 7b00dc72-bcc6-1039-927b-c144385599c7
creatorsName: cn=config
createTimestamp: 20191227073020Z
entryCSN: 20191227073020.070787Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20191227073020Z
```

### 验证配置文件是否正确

```bash
[root@ldap ~]# slaptest -u
5e05b57d ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
5e05b57d ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif"
config file testing succeeded
```

### 启动服务并设置开机启动

```bash
[root@ldap ~]# systemctl start slapd && systemctl enable slapd
[root@ldap ~]# systemctl status slapd
● slapd.service - OpenLDAP Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/slapd.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2019-12-27 15:41:35 CST; 4s ago
     Docs: man:slapd
           man:slapd-config
           man:slapd-hdb
           man:slapd-mdb
           file:///usr/share/doc/openldap-servers/guide.html
  Process: 1783 ExecStart=/usr/sbin/slapd -u ldap -h ${SLAPD_URLS} $SLAPD_OPTIONS (code=exited, status=0/SUCCESS)
  Process: 1768 ExecStartPre=/usr/libexec/openldap/check-config.sh (code=exited, status=0/SUCCESS)
 Main PID: 1786 (slapd)
   CGroup: /system.slice/slapd.service
           └─1786 /usr/sbin/slapd -u ldap -h ldapi:/// ldap:///

12月 27 15:41:32 ldap.example.com systemd[1]: Starting OpenLDAP Server Daemon...
12月 27 15:41:32 ldap.example.com runuser[1771]: pam_unix(runuser:session): session opened for user ldap by (uid=0)
12月 27 15:41:35 ldap.example.com slapd[1783]: @(#) $OpenLDAP: slapd 2.4.44 (Jan 29 2019 17:42:45) $
                                                        mockbuild@x86-01.bsys.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
12月 27 15:41:35 ldap.example.com slapd[1783]: ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
12月 27 15:41:35 ldap.example.com slapd[1783]: ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif"
12月 27 15:41:35 ldap.example.com slapd[1783]: tlsmc_get_pin: INFO: Please note the extracted key file will not be protected with a PIN any more, however it will be still prot...permissions.
12月 27 15:41:35 ldap.example.com slapd[1786]: hdb_db_open: warning - no DB_CONFIG file found in directory /var/lib/ldap: (2).
                                                Expect poor performance for suffix "dc=example,dc=com".
12月 27 15:41:35 ldap.example.com slapd[1786]: slapd starting
12月 27 15:41:35 ldap.example.com systemd[1]: Started OpenLDAP Server Daemon.
Hint: Some lines were ellipsized, use -l to show in full.
```

### 查看监听端口

```bash
[root@ldap ~]# netstat -ntlp | grep slapd
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      1786/slapd
tcp6       0      0 :::389                  :::*                    LISTEN      1786/slapd
```

### 配置 openldap 数据库

```bash
[root@ldap ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@ldap ~]# chown ldap:ldap -R /var/lib/ldap
[root@ldap ~]# chmod 700 -R /var/lib/ldap
[root@ldap ~]# ll /var/lib/ldap/
总用量 324
-rwx------. 1 ldap ldap     2048 12月 27 15:41 alock
-rwx------. 1 ldap ldap   262144 12月 27 15:41 __db.001
-rwx------. 1 ldap ldap    32768 12月 27 15:41 __db.002
-rwx------. 1 ldap ldap    49152 12月 27 15:41 __db.003
-rwx------. 1 ldap ldap      845 12月 27 15:44 DB_CONFIG
-rwx------. 1 ldap ldap     8192 12月 27 15:41 dn2id.bdb
-rwx------. 1 ldap ldap    32768 12月 27 15:41 id2entry.bdb
-rwx------. 1 ldap ldap 10485760 12月 27 15:41 log.0000000001
```

### 导入基本 schema

```bash
[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"
```

## 3. 开启 openldap 日志访问功能
> 默认情况下 openldap 没有启用日志记录功能，在实际使用过程中，为了定位问题，需要使用到 openldap 日志。

```bash
[root@ldap ~]# cat > /root/loglevel.ldif << "EOF"
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats
EOF
```

```bash
[root@ldap ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/loglevel.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"

[root@ldap ~]# systemctl restart slapd
```

```bash
[root@ldap ~]# cat >> /etc/rsyslog.conf << "EOF"
local4.* /var/log/slapd.log
EOF
[root@ldap ~]# systemctl restart rsyslog
```

## 4. 安装和配置 phpldapadmin

```bash
[root@ldap ~]# yum -y install httpd php php-ldap php-gd php-mbstring php-pear php-bcmath php-xml
```

```bash
[root@ldap ~]# yum -y install epel-release
[root@ldap ~]# yum -y install phpldapadmin
```

- 修改配置文件

```bash
[root@ldap ~]# vim /etc/phpldapadmin/config.php
```

```bash
397 $servers->setValue('login','attr','dn');
398 // $servers->setValue('login','attr','uid');
```

```bash
[root@ldap ~]# vim /etc/httpd/conf.d/phpldapadmin.conf
```

```bash
#
#  Web-based tool for managing LDAP servers
#

Alias /phpldapadmin /usr/share/phpldapadmin/htdocs
Alias /ldapadmin /usr/share/phpldapadmin/htdocs

<Directory /usr/share/phpldapadmin/htdocs>
  <IfModule mod_authz_core.c>
    # Apache 2.4
    Require local
    Require ip 192.168.33.1
  </IfModule>
  <IfModule !mod_authz_core.c>
    # Apache 2.2
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
  </IfModule>
</Directory>
```

- 启动服务并设置开机启动

```bash
[root@ldap ~]# systemctl enable httpd && systemctl start httpd
```

- 浏览器访问
  - 主页
![](/resources/openldap/openldap01.png)
  - 登陆
![](/resources/openldap/openldap02.png)
  - 导航
![](/resources/openldap/openldap03.png)

## 5. 添加用户
### 修改 `/usr/share/migrate_common.ph` 文件

```bash
[root@ldap ~]# vim /usr/share/migrationtools/migrate_common.ph
```

```bash
# Default DNS domain
# $DEFAULT_MAIL_DOMAIN = "padl.com";
$DEFAULT_MAIL_DOMAIN = "example.com";

# Default base
# $DEFAULT_BASE = "dc=padl,dc=com";
$DEFAULT_BASE = "dc=example,dc=com";

# Turn this on for inetLocalMailReceipient
# sendmail support; add the following to
# sendmail.mc (thanks to Petr@Kristof.CZ):
##### CUT HERE #####
#define(`confLDAP_DEFAULT_SPEC',`-h "ldap.padl.com"')dnl
#LDAPROUTE_DOMAIN_FILE(`/etc/mail/ldapdomains')dnl
#FEATURE(ldap_routing)dnl
##### CUT HERE #####
# where /etc/mail/ldapdomains contains names of ldap_routed
# domains (similiar to MASQUERADE_DOMAIN_FILE).
# $DEFAULT_MAIL_HOST = "mail.padl.com";

# turn this on to support more general object clases
# such as person.
# $EXTENDED_SCHEMA = 0;
$EXTENDED_SCHEMA = 1;
```

### 添加用户及用户组

```bash
[root@ldap ~]# groupadd ldapgroup1
[root@ldap ~]# groupadd ldapgroup2
[root@ldap ~]# useradd -g ldapgroup1 ldapuser1
[root@ldap ~]# useradd -g ldapgroup2 ldapuser2
```

```bash
[root@ldap ~]# echo "ldap" | passwd --stdin ldapuser1
[root@ldap ~]# echo "ldap" | passwd --stdin ldapuser2
```

### 提取用户和组

```bash
[root@ldap ~]# cat /etc/passwd | grep ldapuser > users
[root@ldap ~]# cat users

ldapuser1:x:1000:1000::/home/ldapuser1:/bin/bash
ldapuser2:x:1001:1001::/home/ldapuser2:/bin/bash

[root@ldap ~]# cat /etc/group | grep ldapgroup > groups
[root@ldap ~]# cat groups

ldapgroup1:x:1000:
ldapgroup2:x:1001:
```

### 生成用户和组的 `ldif` 文件

```bash
[root@ldap ~]# /usr/share/migrationtools/migrate_passwd.pl /root/users > /root/users.ldif
[root@ldap ~]# /usr/share/migrationtools/migrate_group.pl /root/groups > /root/groups.ldif
```

```bash
[root@ldap ~]# cat users.ldif
dn: uid=ldapuser1,ou=People,dc=example,dc=com
uid: ldapuser1
cn: ldapuser1
sn: ldapuser1
mail: ldapuser1@example.com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$6$ab3EWxCv$w/1qc6F3.z4PT0CLHQeqCQdxEUxbKHQBboBFprgz3HdaoJ8flCSmSjQ7yhTzGCoB3wUhJYT6bSVkpJENqPNtq/
shadowLastChange: 18257
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/ldapuser1

dn: uid=ldapuser2,ou=People,dc=example,dc=com
uid: ldapuser2
cn: ldapuser2
sn: ldapuser2
mail: ldapuser2@example.com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$6$Lh9DKEZ0$qHpMrkzDYLuGhYmq4OLY2uuxv1Oj8TgiwsYs9CKPTMw0kzwGs0EekygLca27hHozJ04AMxE3cONNovcCrwti2/
shadowLastChange: 18257
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/ldapuser2
```

```bash
[root@ldap ~]# cat groups.ldif
dn: cn=ldapgroup1,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup1
userPassword: {crypt}x
gidNumber: 1000

dn: cn=ldapgroup2,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup2
userPassword: {crypt}x
gidNumber: 1001
```

### 导入用户及用户组到 `openldap` 数据库

> 尽管已经把用户和用户组信息导入到 openldap 数据库中。但目前 openldap 用户和组之间是没有任何关联的。要把 openldap 数据库中用户和组关联起来需要做单独配置

- 导入基础数据库

```bash
[root@ldap ~]# cat > /root/base.ldif << EOF
dn: dc=example,dc=com
o: example com
dc: example
objectClass: top
objectClass: dcObject
objectclass: organization

dn: cn=Manager,dc=example,dc=com
cn: Manager
objectClass: organizationalRole
description: Directory Manager

dn: ou=People,dc=example,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=example,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit
EOF
```

```bash
[root@ldap ~]# ldapadd -x -w "ldap" -D "cn=Manager,dc=example,dc=com" -f /root/base.ldif
adding new entry "dc=example,dc=com"

adding new entry "cn=Manager,dc=example,dc=com"

adding new entry "ou=People,dc=example,dc=com"

adding new entry "ou=Group,dc=example,dc=com"
```

- 导入用户

```bash
[root@ldap ~]# ldapadd -x -w "ldap" -D "cn=Manager,dc=example,dc=com" -f /root/users.ldif
adding new entry "uid=ldapuser1,ou=People,dc=example,dc=com"

adding new entry "uid=ldapuser2,ou=People,dc=example,dc=com"
```

- 导入组

```bash
[root@ldap ~]# ldapadd -x -w "ldap" -D "cn=Manager,dc=example,dc=com" -f /root/groups.ldif
adding new entry "cn=ldapgroup1,ou=Group,dc=example,dc=com"

adding new entry "cn=ldapgroup2,ou=Group,dc=example,dc=com"
```


- 将用户加入到用户组

```bash
[root@ldap ~]# cat > add_user_to_groups.ldif << "EOF"
dn: cn=ldapgroup1,ou=Group,dc=example,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser1

dn: cn=ldapgroup2,ou=Group,dc=example,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser2
EOF
```

```bash
[root@ldap ~]# ldapadd -x -w "ldap" -D "cn=Manager,dc=example,dc=com" -f /root/add_user_to_groups.ldif
modifying entry "cn=ldapgroup1,ou=Group,dc=example,dc=com"
modifying entry "cn=ldapgroup2,ou=Group,dc=example,dc=com"
```

```bash
[root@ldap ~]# ldapsearch -LLL -x -D 'cn=Manager,dc=example,dc=com' -w "ldap" -b 'dc=example,dc=com' 'cn=ldapgroup1'
dn: cn=ldapgroup1,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup1
userPassword:: e2NyeXB0fXg=
gidNumber: 1000
memberUid: ldapuser1

[root@ldap ~]# ldapsearch -LLL -x -D 'cn=Manager,dc=example,dc=com' -w "ldap" -b 'dc=example,dc=com' 'cn=ldapgroup2'
dn: cn=ldapgroup2,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup2
userPassword:: e2NyeXB0fXg=
gidNumber: 1001
memberUid: ldapuser2
```
- phpldapadmin 导航
![](/resources/openldap/openldap04.png)