# 阿里云 SSH 配置

前不久购买了阿里云，但是突然 SSH 登录不上，查阅一番，解决了这个问题，遂记录下俩。

## 几个概念
### 本地和服务器
本地：你正在使用的操作环境
服务器：你需要远程登录的环境

### 公钥和私钥
SSH 的建立需要公钥和私钥，一般而言，本地产生公钥和私钥，产生公钥和私钥的方式可以通过`ssh-keygen`或者第三方客户端工具如**puttygen**等。公钥需要上传到服务器，私钥用于本地 SSH 登录服务器时用于提供凭证。

> 阿里云是通过在服务器端生成公钥和私钥，私钥为`aliyun.pem`文件，公钥阿里云会自动放到你买的服务器里面

### 修改默认端口

```bash
vi /etc/ssh/sshd_config
# 找到 #Port 22，将 22 修改为其他不使用的端口
service sshd restart # 重启 sshd 服务
```

### 允许密码登录
```bash
su root #切换root账号
vim /etc/ssh/sshd_config #按 i 进入编辑

确保(去掉注释#)
RSAAuthentication      yes  #允许 RSA 认证
PubkeyAuthentication   yes  #允许公钥认证
PermitRootLogin        no  #禁止 ROOT 登陆
PasswordAuthentication  no  #禁止密码登陆
#修改端口, 将 Port 22 改为  Port 端口号数字 #改掉默认端口
# 按 esc 退出编辑，:wq  #保存并退出 sshd_config

service sshd restart #重启 SSH
```

## 参考链接
1. [本地SSH连接阿里云服务器新建的用户（root用户和非root用户）
](https://blog.csdn.net/m0_38128647/article/details/80204739)
2. [【转】服务器添加新用户用ssh-key 登录，并禁用root用户 密码登录](https://www.cnblogs.com/djh-create/p/6897077.html)
   
