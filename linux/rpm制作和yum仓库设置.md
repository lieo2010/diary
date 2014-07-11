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

## python 打包实例##

1. 首先，下载 Python-2.7.6 的源码包并解压：
```python
curl --progress-bar -LO http://mirrors.sohu.com/python/2.7.6/Python-2.7.6.tgz
tar xf Python-2.7.6.tgz
```

2. 第二步，编译 Python-2.7.6
```python
cd Python-2.7.6.tgz

# Python2.7编译安装后会安装到这个目录，方便打包
export INTERMEDIATE_INSTALL_DIR=/tmp/installdir-Python-2.7.6
# RPM包安装后Python2.7的目录
export INSTALL_DIR=/usr/local

LDFLAGS="-Wl,-rpath=${INSTALL_DIR}/lib ${LDFLAGS}" \
            ./configure --prefix=${INSTALL_DIR} --enable-unicode=ucs4 \
                --enable-shared --enable-ipv6
make
make install DESTDIR=${INTERMEDIATE_INSTALL_DIR}
```

3. 第三步，使用 FPM 创建 RPM 包：
```python
# 注意之前导出 INTERMEDIATE_INSTALL_DIR 和 INSTALL_DIR 这两个环境变量，这里还要使用
fpm -s dir -t -f rpm -n python27 -v '2.7.6' \
    -d 'openssl' \
    -d 'bzip2' \
    -d 'zlib' \
    -d 'expat' \
    -d 'db4' \
    -d 'sqlite' \
    -d 'ncurses' \
    -d 'readline' \
    --directories=${INSTALL_DIR}/lib/python2.7/ \
    --directories=${INSTALL_DIR}/include/python2.7/ \
    -C ${INTERMEDIATE_INSTALL_DIR} .
```


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

[updates]
name=lieo - Updates
baseurl=http://repo.kingnet.com/centos/6/$basearch/
gpgcheck=0
enabled=1

```
note:由于没有配置gpg，所以要将gpgcheck设置为0.

- 打包好的rpm，放入仓库并更新

```
createrepo --update /yum/centos/6/x86_64/
```

- 增加EPEL安装源
  - 下载并安装GPG key
  ```ruby
  $ sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 https://www.fedoraproject.org/static/0608B895.txt
  $ sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
  ```

  - 检验下是否安装成功
  ```ruby
  $ sudo rpm -qa gpg*
  ```

  - 安装epel-release-6-8.noarch包
  ```ruby
  $ sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
  ```

- 增加PUIAS安装源
  - 创建/etc/yum.repos.d/PUIAS_6_computational.repo，并添加如下内容：
  ```ruby
  [PUIAS_6_computational]
  name=PUIAS computational Base $releasever - $basearch
  mirrorlist=http://puias.math.ias.edu/data/puias/computational/$releasever/$basearch/mirrorlist
  #baseurl=http://puias.math.ias.edu/data/puias/computational/$releasever/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-puias
  ```
  
  - 下载并安装GPG key
  ```ruby
  $ sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-puias http://springdale.math.ias.edu/data/puias/6/x86_64/os/RPM-GPG-KEY-puias
  $ sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-puias
  ```

  - 检验下是否安装成功
  ```ruby
  $ sudo rpm -qa gpg*
  ```

- 使用阿里云镜像服务器
  - 官方镜像源

  ```ruby
  # 禁用 fastestmirror 插件
  sed -i.backup 's/^enabled=1/enabled=0/' /etc/yum/pluginconf.d/fastestmirror.conf
  # 备份
  mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
  # 使用阿里云镜像
  wget -O /etc/yum.repos.d/CentOS-Base-aliyun.repo http://mirrors.aliyun.com/repo/Centos-6.repo
  ```

  - EPEL 镜像

  ```ruby
  # 安装 EPEL 源
  wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
  wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
  rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
  # 使用阿里云镜像
  if [[ ! -f /etc/yum.repos.d/epel.repo.backup ]]; then
    mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup 2>/dev/null || :
  fi

  if [[ ! -f /etc/yum.repos.d/epel-testing.repo.backup ]]; then
    mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup 2>/dev/null || :
  fi

  wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
  ```

  - PyPi 镜像
  ```ruby
  mkdir -p ~/.pip
  touch ~/.pip/pip.conf

  sed -i.backup -r \
    's/^index-url\s*=\s*.+$/index-url = http:\/\/mirrors.aliyun.com\/pypi\/simple\//' \
    ~/.pip/pip.conf

  # If file not changed, write contents back to pip.conf
  diff "~/.pip/pip.conf" "~/.pip/pip.conf.backup" &> /dev/null

  if [ $? -eq 0 ]; then
  cat > ~/.pip/pip.conf <<EOF
  [global]
  index-url = http://mirrors.aliyun.com/pypi/simple/
  EOF
  fi
  ```
  - RubyGems 镜像
  ```ruby
  gem source -r https://rubygems.org/
  gem source -a http://mirrors.aliyun.com/rubygems/
  
  ```

- Tips：安装完EPEL和PUIAS两个源后，可以检测下：
```ruby
$ sudo yum repolist
```
