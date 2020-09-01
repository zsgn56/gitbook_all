## 1、安装OpenLDAP

```
 参考http://blog.51cto.com/11555417/2065747
    https://www.ilanni.com/?p=14127
    https://cloud.tencent.com/developer/article/1349459
```

```
yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools
slapd -VV

slappasswd -s ******
#加密后的字段保存下，等会我们在配置文件中会使用到。

Vim /etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif
修改olcDatabase={2}hdb.ldif文件,对于该文件增加一行
olcRootPW: {SSHA}o1bqtofUr95dkEDdXbAMAVPFSnNDU3+2，
然后修改域信息：
olcSuffix: dc=homeking365,dc=com
olcRootDN: cn=root,dc=homeking365,dc=com

slaptest -u进行验证

启动命令
systemctl enable slapd
systemctl start slapd
systemctl status slapd
```

## 2、配置OpenLDAP数据库

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap:ldap -R /var/lib/ldap
chmod 700 -R /var/lib/ldap
ll /var/lib/ldap/


ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif



vim /usr/share/migrationtools/migrate_common.ph +71
$DEFAULT_MAIL_DOMAIN = "homeking365.com";
$DEFAULT_BASE = "dc=homeking365,dc=com";
$EXTENDED_SCHEMA = 1;
```

## 3、创建用户导入信息

```
groupadd ldapgroup1
useradd -g ldapgroup1 ldapuser1
echo '123456' | passwd --stdin ldapuser1
grep ":10[0-9][0-9]" /etc/passwd > /root/users
grep ":10[0-9][0-9]" /etc/group > /root/groups

/usr/share/migrationtools/migrate_passwd.pl /root/users > /root/users.ldif
/usr/share/migrationtools/migrate_group.pl /root/groups > /root/groups.ldif
cat users.ldif
cat groups.ldif


cat > /root/base.ldif << EOF
dn: dc=hbgd,dc=com
o: hbgd com
dc: hbgd
objectClass: top
objectClass: dcObject
objectclass: organization
dn: cn=root,dc=hbgd,dc=com
cn: root
objectClass: organizationalRole
description: Directory Manager
dn: ou=People,dc=hbgd,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit
dn: ou=Group,dc=hbgd,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit
EOF

ldapadd -x -w "xxxxxx" -D "cn=root,dc=homeking365,dc=com" -f /root/base.ldif
ldapadd -x -w "xxxxxx" -D "cn=root,dc=homeking365,dc=com" -f /root/users.ldif
ldapadd -x -w "xxxxxx" -D "cn=root,dc=homeking365,dc=com" -f /root/groups.ldif


cat > add_user_to_groups.ldif << "EOF"
dn: cn=ldapgroup1,ou=Group,dc=homeking365,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser1
EOF
ldapadd -x -w "xxxxx" -D "cn=root,dc=homeking365,dc=com" -f /root/add_user_to_groups.ldif 

ldapsearch -LLL -x -D 'cn=root,dc=homeking365,dc=com' -w "xxxx" -b 'dc=homeking365,dc=com' 'cn=ldapgroup1'
```

## 4、开启OpenLDAP日志访问功能

```
cat > /root/loglevel.ldif << "EOF"
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats
EOF

ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/loglevel.ldif
systemctl restart slapd


cat >> /etc/rsyslog.conf << “EOF”
local4.* /var/log/slapd.log
EOF
systemctl restart rsyslog
tail -f /var/log/slapd.log
```

## 5、开启memberof

```
##启用MemberOf时需要注意配置文件的通配符{0}/{2},这个数字是根据当前的/etc/openldap/slapd.d/cn\=config/生成的内容得出

ldapsearch -H ldapi:// -LLL -Q -Y EXTERNAL -b "cn=config" "(olcRootDN=*)" dn olcRootDN olcRootPW

cat > /root/memberof_conf.ldif << "EOF"
dn: cn=module{0},cn=config
cn: modulle{0}
objectClass: olcModuleList
objectclass: top
olcModuleload: memberof.la
olcModulePath: /usr/lib64/openldap

dn: olcOverlay={0}memberof,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfUniqueNames
olcMemberOfMemberAD: uniqueMember
olcMemberOfMemberOfAD: memberOf
EOF

cat > /root/refint1.ldif << "EOF"
dn: cn=module{0},cn=config
add: olcmoduleload
olcmoduleload: refint
EOF


cat > /root/refint2.ldif << "EOF"
dn: olcOverlay=refint,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: refint
olcRefintAttribute: memberof uniqueMember  manager owner
EOF

ldapadd -Q -Y EXTERNAL -H ldapi:/// -f memberof_conf.ldif 
ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f refint1.ldif 
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f refint2.ldif
```

## 6、创建memberof组

```
cat > /root/new_memberof_group.ldif << "EOF"
dn: cn=zentao_groups,ou=Group,dc=homeking365,dc=com
objectClass: groupOfUniqueNames
cn: zentao_groups
description: 运维部
uniqueMember: uid=hkomd,ou=People,dc=homeking365,dc=com
EOF
ldapadd -x -w "xxxx" -D "cn=root,dc=homeking365,dc=com" -f new_memberof_group.ldif


###添加其它用户到这个组
cat > /root/adduser_to_memberof_group.ldif << "EOF"
dn: cn=zentao_groups,ou=Group,dc=homeking365,dc=com
changetype: modify
add: uniqueMember
uniqueMember: uid=wantt,ou=People,dc=homeking365,dc=com
EOF
ldapadd -x -w "xxx" -D "cn=root,dc=homeking365,dc=com" -f adduser_to_memberof_group.ldif

ldapsearch -x -LLL -H ldap:/// -b 'ou=group,dc=homeking365,dc=com'  memberof
```

## 7、confluence使用ldap

用户管理---用户目录中设置基本配置![](/assets/20181108.png)高级--用户模式中，过滤需要的用户组

![](/assets/201811083.png)

各按顺序执行脚本

    创建用户脚本
    #!/bin/sh

    DATE="$(date +%Y%m%d%H%M)"
    echo $DATE >>/root/create_ldapuser.log
    while read LINE
    do
    USER=`echo $LINE|awk {'print $2'}`
    SN=`echo $LINE |awk {'print $1'}`
    useradd -g ldapgroup ${USER}
    echo 'r5P2222' | passwd --stdin ${USER}
    grep ${USER} /etc/passwd > /root/users

    /usr/share/migrationtools/migrate_passwd.pl /root/users > /root/users.ldif

    sed -i "s/sn: ${USER}/sn: ${SN}/g" /root/users.ldif

    ldapadd -x -w "xxx" -D "cn=root,dc=homeking365,dc=com" -f /root/users.ldif


    cat > add_user_to_groups.ldif << EOF
    dn: cn=ldapgroup,ou=Group,dc=homeking365,dc=com
    changetype: modify
    add: memberuid
    memberuid: ${USER}
    EOF

    ldapadd -x -w "xxx" -D "cn=root,dc=homeking365,dc=com" -f /root/add_user_to_groups.ldif

    cat > /root/adduser_to_memberof_group.ldif << EOF
    dn: cn=docs_groups,ou=Group,dc=homeking365,dc=com
    changetype: modify
    add: uniqueMember
    uniqueMember: uid=${USER},ou=People,dc=homeking365,dc=com
    EOF
    ldapadd -x -w "xxx" -D "cn=root,dc=homeking365,dc=com" -f /root/adduser_to_memberof_group.ldif

    done < /root/ldapuser.txt

    ###
    bash createuser.sh  >>/root/create_ldapuser.log


    2、生成用户列表到confluence中加入到指定组
    awk '{print $2}' /root/ldapuser.txt > /root/tempuser.txt
    awk BEGIN{RS=EOF}'{gsub(/\n/," ");print}' /root/tempuser.txt |sed 's/[ ][ ]*/,/g' >/root/add_to_group.txt

## 8、gitlab使用ldap

```
     vim gitlab.rb

     label: 'LDAP'
     host: '192.168.100.21'
     port: 389
     uid: 'cn'
     bind_dn: 'cn=root,dc=xxx,dc=com'
     password: 'xxxxx'

     base: 'dc=homeking365,dc=com'
     user_filter: '(memberOf=gitlab-ee_groups,ou=Group,dc=xxxx,dc=com)'
```

## 9、nginx使用ldap

```
参考：https://github.com/kvspb/nginx-auth-ldap/



yum -y install  zlib zlib-devel openldap openldap-devel git
git clone https://github.com/kvspb/nginx-auth-ldap.git
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.37 --with-openssl=/usr/local/src/openssl-1.0.2d  --add-module=/usr/local/src/nginx-auth-ldap-master
make && make install
systemctl restart nginxd
```

nginx.conf

```
    ldap_server ldap {
        url "ldap://192.168.100.21:389/dc=homeking365,dc=com?uid?sub?(memberOf=cn=deploy_groups,ou=Group,dc=xxxx,dc=com)";
        binddn "cn=root,dc=xxxx,dc=com";
        binddn_passwd "xxxxxx";
        #group_attribute memberUid;
        group_attribute_is_dn on;
        #require group “cn=deploy,ou=Group,dc=xxxxx,dc=com”;
        require valid_user;
    }
```

vhost.conf

```
            auth_ldap “Forbidden”;
            auth_ldap_servers ldap;
```



