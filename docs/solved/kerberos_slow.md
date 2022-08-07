# 记kerberos认证慢的异常

# 背景
前一段上了一套新的hadoop集群，500多台节点，安全集群开启了Kerberos认证，ldap做票据存储。随着离线作业迁移到新集群，开始出现大量作业提交时，认证非常慢的问题。
# 分析
考虑可能是两方面的问题，一是并发认证过多，kdc压力瓶颈，二是kdc查询ldap太慢，ldap的瓶颈。
# 排查
首先top了一下，发现slapd的进程占用了2000%的cpu，估计大概率是ldap出了问题。
查看官方文档
```
OpenLDAP 2.3 and later have transitioned to using a dynamic runtime configuration engine, slapd-config(5). slapd-config(5)
```
说新版本的已经使用了动态运行时的配置，但实际发现同事安装时还是用的旧配置文件，因此其配置没有生效。

通过journalctl 查看日志，发现访问ldap没有走索引

查看现有配置

```
cd /etc/openldap/slapd.d
ls -l
drwxr-x---. 3 ldap ldap 4096 Dec 28 07:54 cn=config
-rw-------. 1 ldap ldap  589 Dec 28 07:54 cn=config.ldif
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:///  -b cn=config 'olcDatabase={1}hdb' olcDbIndex
```
增加两个索引
```
dn:olcDatabase={1}hdb,cn=config
changetype:modify
add:olcDbIndex
olcDbIndex:krbPrincipalName eq
```

```
dn:olcDatabase={1}hdb,cn=config
changetype:modify
add:olcDbIndex
olcDbIndex:objectClass eq
```
```
ldapmodify -Q  -Y EXTERNAL -H ldapi:///  -f  hdb-index.idf

```
再次使用Kerberos认证，很快