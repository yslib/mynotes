
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
