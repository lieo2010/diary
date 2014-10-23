# PERL下面Gearman使用
---

- CPANMinus 的安装

```ruby
wget  http://xrl.us/cpanm  --no-check-certificate -O /sbin/cpanm 
chmod +x  /sbin/cpanm 
```

完整版本安装 使用了上面,没有必要运行下面的方法

```ruby
wget -O- http://xrl.us/cpanm  --no-check-certificate | perl - --sudo --self-upgrade \
 --mirror http://mirrors.163.com/cpan --mirror-only
```

- 安装必要包
	
```ruby
yum -y install boost libdrizzle libdrizzle-devel gcc44 g++44
```

- 安装Gearman

```ruby
/sbin/cpanm Gearman::Client 
/sbin/cpanm Gearman::Server 
/sbin/cpanm Gearman::Worker 
```

- 启动job server

```ruby
gearmand -d L 127.0.0.1 -p 7003 
```

- PHP扩展

```ruby
wget http://pecl.php.net/get/gearman-1.1.2.tgz
	
tar xvzf gearman-1.1.2.tgz 
cd gearman-1.1.2
/opt/app/php-5.4.24/bin/phpize 
./configure --with-php-config=/opt/app/php-5.4.24/bin/php-config 
make
make install
```
