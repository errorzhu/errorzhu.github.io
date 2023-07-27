# mysql部署

1. 5.x
```
0 systemctl stop mysql

1 vi /etc/my.cnf
skip-grant-tables
plugin-load-add = auth_socket.so

systemctl start mysql

2 mysql -uroot
set global read_only=0
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privilege



debug
mysql> select user,host,plugin ,authentication_string from user;

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
ALTER USER 'root'@'%' IDENTIFIED  BY '123456';
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
 GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root123' WITH GRANT OPTION


ALTER USER 'root'@'%' IDENTIFIED BY 'root';

SHOW VARIABLES LIKE 'validate_password%';

grant all privileges on *.* to 'root'@'localhost' identified by 'root' with grant option;



maria
set PASSWORD FOR 'root'@'localhost' = PASSWORD('root');
set PASSWORD FOR 'root'@'%' = PASSWORD('123456');
set PASSWORD FOR 'root'@'2021-12-10-1.novalocal' = PASSWORD('123456');
flush privileges;


create user 'pdp'@'115.171.39.228' identified by 'pdp123?';
create user 'pdp'@'122.200.93.22' identified by 'pdp123?';
grant all privileges on *.* to 'pdp'@'115.171.39.228' identified by 'pdp123?' with grant option;


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ubuntu mysql5.7安装
	apt-get install mysql-server
	cat /etc/mysql/debian.cnf
	hmHkPF3VeHtxZobW
	mv /var/lib/mysql /opt/ddc
	vi /etc/mysql/mysql.conf.d/mysqld.cnf
	datadir = /opt/ddc/mysql
	bind-address = 0.0.0.0
	mysql -udebian-sys-maint -phmHkPF3VeHtxZobW
	use mysql;
	#登陆成功后执行如下指令修改密码为123456
	alter user 'root'@'localhost' identified with mysql_native_password by '123456';
	mysql -uroot -p
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456';
	alter user 'root'@'%' identified with mysql_native_password by '123456';
	flush privileges;
	systemtl restart mysql

```

2. 8.x

```
安装mysql8
releasever=$(cat /etc/redhat-release |awk '{print $(NF-1)}'|awk -F. '{print$1}')

yum install http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el${releasever}/mysql80-community-release-el${releasever}-3.noarch.rpm

sed -i 's#repo.mysql.com/yum#mirrors.tuna.tsinghua.edu.cn/mysql/yum#; s/mysql-\([0-9]\)\.\([0-9]\)/mysql\1\2/; s#/el/\([0-9]\)/#-el\1/#; s#$basearch/##' /etc/yum.repos.d/mysql-community.repo 

yum repolist all | grep mysql
yum-config-manager --enable mysql80-community
 vi /etc/yum.repos.d/mysql-community.repo


yum install mysql-community-server


systemctl start mysqld

grep "A temporary password" /var/log/mysqld.log

mysql -uroot -p
ALTER USER USER() IDENTIFIED BY 'A%Xh/g<xk383';
set global validate_password.length=6;
set global validate_password.policy=0;
set global validate_password.check_user_name=off;
update user set host='%' where user='root';
flush privileges;

退出，重新登录
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;

```
