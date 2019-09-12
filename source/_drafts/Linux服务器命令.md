title:Linux服务器命令

---

######linux下非root用户安装软件
切换到root用户:
sudo su -
提示输入密码,输入当前登陆账户得密码即可.

查看用户权限:
ls -l

添加用户:
> useradd -d /usr/username -m username

为用户增加密码：
> passwd username

新建工作组：
> groupadd groupname

将用户添加进工作组：
> usermod -G groupname username

删除用户：
> userdel username


更改/usr/local下的nginx-web文件夹及其包含的文件夹的所有用户为nginx:
> chown -R nginx /usr/local/nginx-web

更改/usr/local下的nginx-web文件夹及其包含的文件夹的所有用户组为nginx:
> chgrp -R nginx /usr/local/nginx-web

更改/usr/local下的nginx-web文件夹及其包含的文件夹的操作权限为755:
> chmod -R 755 /usr/local/nginx-web