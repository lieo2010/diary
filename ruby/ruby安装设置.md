# RUBY 安装设置 
---

## ruby下载安装 ##

```ruby
wget https://ruby.taobao.org/mirrors/ruby/2.1/ruby-2.1.2.tar.gz

tar xvzf ruby-2.1.2.tar.gz

cd ruby-2.1.2

./configure

make

make install
```

## rubygem下载安装 ##

```ruby
git clone https://github.com/rubygems/rubygems.git OR wget http://production.cf.rubygems.org/rubygems/rubygems-2.3.0.tgz

cd rubygems

ruby setup.rb 
```

## 更新gem ##

```ruby
gem update --system
```

## gem使用taobao源加速 ##

```ruby
gem sources --remove http://rubygems.org/

gem sources -a https://ruby.taobao.org/
```

## bundle用户使用淘宝源 ##

```ruby
source 'https://ruby.taobao.org/'
gem 'rails', '4.1.0'
...
```

## rvm使用者使用淘宝源 ##

- for mac

```ruby
sed -i .bak 's!cache.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
```

- for linux

```ruby
sed -i 's!cache.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
```
---
##使用rvm安装ruby（推荐）

- 安装rvm

```ruby
curl -L https://get.rvm.io | bash -s
source /etc/profile.d/rvm.sh
```

- 确保rvm是最新稳定版

```ruby
rvm get stable #如果报错，按提示执行下面一条
gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
```

- 修改 RVM 的 Ruby 安装源到国内的 淘宝镜像服务器，这样能提高安装速度

```ruby
sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db
```
---
##ruby的安装与切换
- 检查安装ruby的环境

```ruby
rvm requirements
```
- 列出已知版本

```ruby
rvm list known
```

- 安装一个ruby版本

```ruby
rvm install 1.9.3
```

- 使用一个版本

```ruby
rvm use 1.9.3
```

- 查询已安装的ruby

```ruby
rvm list
```

- 卸载一个已安装版本

```ruby
rvm remove 1.9.2
```

- 预设ruby版本

```ruby
rvm use 1.9.3 --default
```

- 更新rubygems版本搭配你预设的版本

```ruby
rvm rubygems current
```

- 安装rails

```ruby
gem install rails --no-rdoc --no-ri
```

