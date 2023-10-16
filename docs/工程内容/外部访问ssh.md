---
sort: 2
---

# 外部访问ssh

> 持续更新中

[🔨算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/工程内容/外部访问ssh.html)
[🔨个人知乎](https://www.zhihu.com/people/zhangyj-n)

[TOC]

> 在容器化开发环境配置中,直接替换ssh配置文件

```
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem    sftp    /usr/lib/openssh/sftp-server
PubkeyAuthentication yes
RSAAuthentication yes
PermitRootLogin yes
```

https://www.cnblogs.com/caixiaohua/p/9683591.html

## **1.安装ssh**

```bash
apt-get install openssh-server
```

## **2.备份ssh的配置文件**

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```



## **3.新装的ssh需要修改配置文件**

```bash
vi /etc/ssh/sshd_config
```

配置文件修改这几处地方

```bash
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem    sftp    /usr/lib/openssh/sftp-server
PubkeyAuthentication yes
RSAAuthentication yes
PermitRootLogin yes
```

**## 4.启动ssh**

```
service ssh start            # * Starting OpenBSD Secure Shell server sshd<br># 或者<br>
/etc/init.d/ssh start        # * Starting OpenBSD Secure Shell server sshd
```

如果提示错误信息中包含**could not load host key** 则需要重新生成 key

```
sudo rm /etc/ssh/ssh*key # 先移除旧的key 

dpkg-reconfigure openssh-server
```

生成之后需要重启SSH服务使新的密钥生效：   

```
service ssh restart          # * Restarting OpenBSD Secure Shell server sshd<br># 或者
/etc/init.d/ssh restart       # * Restarting OpenBSD Secure Shell server sshd
```

启动、停止和重启ssh的命令如下

```
/etc/init.d/ssh start         # * Starting OpenBSD Secure Shell server sshd
/etc/init.d/ssh stop          # * Stopping OpenBSD Secure Shell server sshd
/etc/init.d/ssh restart       # * Restarting OpenBSD Secure Shell server sshd
```



## **5.查看服务状态**

```
service ssh status # * sshd is running  显示此内容则表示启动正常
```



## **6.查看ssh是否启动**

```
ps -e | grep ssh
```

如果ssh已经启动则会提示

```
469 ?        00:00:00 sshd
```



## **7.设置ssh开机自启动**

```
sudo systemctl enable ssh    #说明：sudo是提升权限，systemctl是服务管理器，enable是systemctl的参数，表示启用开机自动运行，ssh是要设置的服务名称。
```

注：如果要设置开机禁止启动则

```
sudo systemctl disable ssh   #说明：sudo是提升权限，systemctl是服务管理器，disable是systemctl的参数，表示禁止开机运行，ssh是要设置的服务名称。
```

容器内设置

```
# 找到并打开文件/root/.bashrc
$ vim /root/.bashrc
# 在.bashrc末尾添加如下代码
$ service ssh start
```





更多内容查看

[环境框架安装](https://kg-nlp.github.io/Algorithm-Project-Manual/算法框架/框架环境.html)
