# rpm制作和yum仓库设置
---

---
## rpm打包工具fpm ##

 - 安装相关软件包-ruby
 
```
yum -y install ruby-devel ruby-rdoc  
```
 - 安装rubygem
 
```
wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.25.zip  
unzip rubygems-1.8.25.zip
cd rubygems-1.8.25  && ruby  setup.rb
```

- 安装rpmbuild

```
yum -y install rpm-build
```

- 安装打包工具fpm

```
gem install fpm
```
---
## fpm使用命令(以htop为例）##
   

- 下载htop源码软件包 htop-1.0.3.tar.gz 
    
```
        tar xvzf htop-1.0.3.tar.gz 
        cd htop-1.0.3
        ./configure --prefix=/opt/app/htop
        make
        make install DESTDIR=/tmp/htop #先安装到临时安装目录，以便下一步生成rpm
```

- 使用fpm编译生成rpm包

```
    fpm -s dir -t rpm -n "htop-lieo" -v 1.0.3 /tmp/htop
```
    
note: 如果是要增加软件依赖则加入-d "soft", -e 参数是先预览更改spec配置，然后再编译，-s指定数据来源于目录，-t rpm指生成rpm包，-n 生成的软件包名字，-v 指定软件版本，如下：

```
    fpm -s dir -t rpm -n "htop-lieo" -v 1.0.3 -d "http" -e /tmp/htop
```
note:   当前目录会生成一个htop-1.0.3-1.x86_64.rpm

---
- yum仓库搭建

```
yum -y install createrep
```

- 创建目录

```
mkdir -p /yum/centos/5/{i386,x86_64}
```

- 初始化repodata

```
createrepo -d  /yum/centos/6/x86_64
```

- 配置nginx

```
server {

   listen 80;
   server_name yum.lieo.com;

   location / {
         root /yum;
         autoindex  on;
    }

}
```

- 配置yum的源

```
[lieo_repo]
name=lieo repo
baseurl=http://yum.lieo.com/centos/6/$basearch/
gpgcheck=0
enabled=1
```
note:由于没有配置gpg，所以要将gpgcheck设置为0.

- 打包好的rpm，放入仓库并更新

```
createrepo --update /yum/centos/6/x86_64/
```

