

# Linux下科学上网

**Clash**

  下载[Clash](https://github.com/Dreamacro/clash/releases)
  把提供的配置文件config.yaml复制到~/.config/clash 以及解压后的二进制文件中的根目录
  从终端运行，如果需要配置的，登陆clash.razord.top
  设置系统代理，端口按照config.yaml中的设置

**Samba**

  配置局域网共享

  ```sudo pacman -S samba```

  创建 samba 账号

  ```pdbedit -a root```

  设置开机启动

  ```systemctl enable smb.service```

  启动

  ```systemctl start smb.service```

  配置文件在 

  ```/etc/samba/smb.conf```

  配置文件模板

  ```
  [global]
  workgroup=WORKGROUP
  security=user
  [share]
  path=/path/to/your/share/dir
  public=yes
  writable=yes
  read only=no
  ```
  

