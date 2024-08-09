# 安装
```shell
sudo apt install mysql-server mysql-client
```

# 修改密码
```shell
sudo mysql

#更改mysql的默认root用户密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码'; 

#更新权限
FLUSH PRIVILEGES;

# 退出mysql
quit;
# 重启服务即可
sudo service mysql restart
```

# 开启远程连接
```shell
# 登录mysql数据库
mysql -u root -p
# 选择mysql数据库
use mysql;
# 修改root的host为%即无论在哪台主机都能够登录
update user set host='%' where user='root' and host='localhost';
# 刷新权限
flush privileges
# 退出
quit;
# 接下来修改 /etc/mysql/mysql.conf.d/mysqld.cnf
# 注释 bind-address=127.0.0.1 然后重启服务即可
```

# 好用的IDE介绍
[HeidiSQL](https://www.heidisql.com/)
