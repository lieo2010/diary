# deb软件包制作和deb仓库设置
---

---
## deb打包工具fpm ##

 - 安装相关软件包-ruby
 
```
apt-get install ruby 
```

- 安装打包工具fpm

```
gem install fpm
```
---
## fpm使用命令(以mysql为例）##
   

- 下载htop源码软件包 mysql-5.6.10.tar.gz 
    
```
        tar xvzf mysql-5.6.10.tar.gz
        cd mysql-5.6.10
        cmake . -DCMAKE_INSTALL_PREFIX=/opt/app/mysql-5.6.10 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DEXTRA_CHARSETS=all

        make
        make install DESTDIR=/tmp/mysql-root #先安装到临时安装目录，以便下一步生成rpm
```

- 使用fpm编译生成rpm包

```
    fpm -s dir -t deb -n "mysql-kingnet" -v 5.6.10 -C /tmp/mysql-root -p /root/mysql-lieo-VERSION_ARCH.deb --after-install=after-mysql -e .
```
    
note: 如果是要增加软件依赖则加入-d "soft", -e 参数是先预览更改spec配置，然后再编译，-s指定数据来源于目录，-t deb指生成deb包，-n 生成的软件包名字，-v 指定软件版本，-p 指定deb包生成再哪个目录下面，这里是生成到/root/下面,--after-install是指包安装完毕后要执行的命令写到after-mysql这个文件里面，另外，最后要有一个 "."不然会报错，这个是fpm的一个bug


---
- deb仓库搭建

```
apt-get install reprepro gnupg
```

- 创建仓库

我们现在要开始创建仓库，首先你需要创建一些文件夹，我们的仓库会放在/opt/apt目录，让我们先创建这些目录。

```
cd /opt
mkdir apt/{incoming,conf,key}
```
我们需要在/var/www/apt/conf创建一个文件“distributions”。

```
touch /var/www/apt/conf/distributions
```

加入下面这几行到distributions这个文件中并保存。

```
Origin: (你的名字)
Label: (库的名字)
Suite: (stable 或 unstable)
Codename: (发布的代码名，比如  trusty)
Version: (发布的版本，比如 14.04)
Architectures: (软件包所支持的架构， 比如 i386 或 amd64)
Components: (包含的部件，比如 main restricted universe multiverse)
Description: (描述)
SignWith: yes #不用key的话，这个可以不要写进去
```

- 创建仓库树

```
reprepro --ask-passphrase -Vb /opt/apt export
```

- 在新创建的仓库中加入包

```
cd /opt/apt(必须进入这个目录执行)

reprepro --ask-passphrase -Vb . includedeb distributions文件里面的Codename /home/packages.deb
```

- 如果要移除包

```
 reprepro --ask-passphrase -Vb /opt/apt remove distributions文件里面的Codename package.deb
```

- 配置nginx

```
server {

   listen 80;
   server_name apt.lieo.com;

   location / {
         root /opt/apt;
         autoindex  on;
    }

}
```

- 配置apt的源

```
/etc/apt/sources.list 里添加自己的apt源

deb http://apt.lieo.com/ distributions文件里面的Codename main

```


- 更新apt就可以用了

```
apt-get update
```

