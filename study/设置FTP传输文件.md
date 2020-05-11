# 设置FTP传输文件

## ftp什么？



## 服务器端处理

vsftpd是Linux下比较好的FTP服务器。

```shell
// 检查是否安装vsftpd
rpm -qa | grep vsftpd
// 没有就安装vsftpd
yum -y install vsftpd

// 启动vsftpd服务
systemctl start vsftpd.service
// 停止vsftpd服务
systemctl stop vsftpd.service
// 卸载vsftp服务，配置出错是需要卸载vsftpd服务。
/sbin/service vsftpd stop
rpm -e vsftpd-3.0.2-21.el7.x86_64
```



## macOS处理

