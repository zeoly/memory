## 使用证书登录ssh

### 本地生成证书

命令 `ssh-keygen`，连续回车即可，在~/.ssh/目录下会生成 **id_ed25519** 和 **id_ed25519.pub** 两个文件

### 在远端配置配置证书

- 上传公钥至服务端
- 使用命令 `cat id_ed25519.pub > ~/.ssh/authorized_keys` 将公钥写入authorized_keys文件

### 本地增加快捷连接

- 在本地 ~/.ssh/config 文件中增加配置，包括远端服务器ip、端口、登录用户

```
Host home
    HostName 192.168.0.1
    Port 10022
    User zeoly
```

- 可在本地通过 `ssh home`命令快捷登录

### 关闭服务端密码登录

- 修改 `/etc/ssh/sshd_config`，将`PasswordAuthentication no` 开放注释
