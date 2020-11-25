
## Clash
把提供的配置文件config.yaml复制到~/.config/clash 以及解压后的二进制文件中的根目录
从终端运行，如果需要配置的，登陆clash.razord.top
设置系统代理，端口按照config.yaml中的设置

## Samba
1. 配置局域网共享
```shell
  sudo pacman -S samba
```

2. 创建 samba 账号
```shell
pdbedit -a root
```

3. 设置开机启动
```shell
systemctl enable smb.service
```

4. 启动
```shell
systemctl start smb.service
```

5. 配置文件在 
```shell
/etc/samba/smb.conf
```

6. 配置文件模板
```ini
  [global]
  workgroup=WORKGROUP
  security=user # share (dangerous!), user ,server,domain
  [share]
  path=/path/to/your/share/dir
  public=yes
  writable=yes
  read only=no
```

7. X Forwarding to windows
当（从Windows）ssh到服务器上时，有时候需要临时运行一下Linux下面的X11程序。我们需要设置X11转发。

### X-Client配置：
对于OpenSSH
把sshd-config文件下的如下几项的注释去掉并重启ssh server

```
AllowTcpForwarding yes
X11Forwarding yes
X11DisplayOffset 10
```

并且导入环境变量
```
export DISPLAY=ip:0.0   # ip 是ssh客户端的ip
```

### X-Server配置：
在Windows下，推荐使用VcXsrv 作为 X-Server
链接时带上-X参数
