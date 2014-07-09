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

