---
layout: post
title:  "OpenLDAP"
categories: linux openldap
---

1. 安装

```bash
# lsb_release -a # 查看ubuntu版本号
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 19.04
Release:	19.04
Codename:	disco
# apt-get update
# apt-get install -y slapd ldap-utils
```

2. 配置监听端口

```bash
# sed -i -e 's!^SLAPD_SERVICES=.*!SLAPD_SERVICES="ldap://172.16.13.81:389/ ldapi:///"!g' /etc/default/slapd
# systemctl daemon-reload
# service slapd restart
```

3. 创建db

```bash
#slappasswd -s ldap #i use 'ldap' as admin password
{SSHA}478rUAqn11CwbDd8tVI+eIsXft0fEO3U
# printf "{SSHA}478rUAqn11CwbDd8tVI+eIsXft0fEO3U" | base64
e1NTSEF9NDc4clVBcW4xMUN3YkRkOHRWSStlSXNYZnQwZkVPM1U=
# cat newDb.ldif
dn: olcDatabase=mdb,cn=config
changetype: add
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
olcDbDirectory: /var/lib/ldap
olcAccess: {0}to attrs=userPassword by self write by anonymous auth by * none
olcAccess: {1}to attrs=shadowLastChange by self write by * read
olcAccess: {2}to * by * read
olcLastMod: TRUE
olcSuffix: dc=wdan,dc=ltd
olcRootDN: cn=admin,dc=wdan,dc=ltd
olcRootPW:: e1NTSEF9NDc4clVBcW4xMUN3YkRkOHRWSStlSXNYZnQwZkVPM1U=
olcDbIndex: objectClass eq
olcDbIndex: cn,uid eq
olcDbIndex: uidNumber,gidNumber eq
olcDbIndex: member,memberUid eq
olcDbMaxSize: 1073741824
# ldapmodify -Y EXTERNAL -H ldapi:/// -f newDb.ldif
# ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```

4. 创建条目
```bash
# cat base.ldif
dn: dc=wdan,dc=ltd
dc: wdan
objectClass: top
objectClass: domain

dn: ou=People,dc=wdan,dc=ltd
ou: People
objectClass: organizationalUnit
# ldapadd -x -D "cn=admin,dc=wdan,dc=ltd" -w ldap -f base.ldif
# ldapsearch -x -b 'dc=wdan,dc=ltd'  '(objectclass=*)' #验证是否创建成功
```

5. 添加用户

```bash
# cat item.ldif
dn: cn=user01,ou=People,dc=wdan,dc=ltd
objectclass: inetOrgPerson
cn: user01
sn: wan
mail: user01@outlook.com
mobile: 13701010202
displayName: user01
# ldapadd -x -D "cn=admin,dc=wdan,dc=ltd" -w ldap  -f item.ldif
# ldapsearch -x -D "cn=admin,dc=wdan,dc=ltd" -b "cn=user01,ou=People,dc=wdan,dc=ltd" -w ldap
```

6. 更新并验证用户密码

```bash
# ldappasswd -x -D "cn=admin,dc=wdan,dc=ltd" -w ldap "cn=user01,ou=People,dc=wdan,dc=ltd"
New password: OOmEkFjE
# ldapwhoami -x -w OOmEkFjE -D cn=user01,ou=People,dc=wdan,dc=ltd -H ldapi:///  #验证用户名和密码
```

7. 查看数据
```bash
# ldapsearch -x -D "cn=admin,dc=wdan,dc=ltd" -b "cn=user01,ou=People,dc=wdan,dc=ltd" -w ldap
```

8. 修改数据

```bash
# ldapsearch -x -b "cn=user01,ou=People,dc=wdan,dc=ltd" -D "cn=admin,dc=wdan,dc=ltd" -w ldap
# cat update.ldif
dn: cn=user01,ou=People,dc=wdan,dc=ltd
changetype: modify
replace: mail
mail: user01@example.com
-
add: preferredLanguage
preferredLanguage: zh-cn
-
delete: mobile
-
# ldapmodify -x -D "cn=admin,dc=wdan,dc=ltd" -w ldap -f update.ldif
# ldapsearch -x -D "cn=admin,dc=wdan,dc=ltd" -b "cn=user01,ou=People,dc=wdan,dc=ltd" -w ldap
```

9. 删除数据

```bash
# ldapsearch -x -D "cn=admin,dc=wdan,dc=ltd" -b "cn=user01,ou=People,dc=wdan,dc=ltd" -w ldap
# cat del.ldif
dn: cn=user01,ou=People,dc=wdan,dc=ltd
changetype: delete
# ldapmodify -x -D "cn=admin,dc=wdan,dc=ltd" -w ldap -f del.ldif
# ldapsearch -x  -D "cn=admin,dc=wdan,dc=ltd" -b "cn=user01,ou=People,dc=wdan,dc=ltd" -w ldap
```

10. 启用新模块

```bash
# ldapadd -Q -Y EXTERNAL -H ldapi:/// -W -f /etc/ldap/schema/ppolicy.ldif
```

11. 查看加载的模块
```bash
# ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config nsa*
```

### 参考资料
http://www.openldap.org/
https://www.ibm.com/developerworks/cn/linux/l-openldap/index.html
https://www.digitalocean.com/community/tutorials/understanding-the-ldap-protocol-data-hierarchy-and-entry-components
https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf
https://stackoverflow.com/questions/22248382/how-to-configure-ppolicy-overlays-in-openldap-2-4-28-ubuntu-12-04
https://www.cnblogs.com/jifeng/p/3470797.html
http://www.zytrax.com/books/ldap/ch3/
https://openldap.org/doc/admin24/schema.html
http://www.rfc-editor.org/rfc/rfc2798.txt
https://www.alvestrand.no/objectid/top.html
