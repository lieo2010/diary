## ngx_lua_waf 安装配置##
---
ngx_lua_waf是一个基于lua-nginx-module(openresty)的web应用防火墙

- 安装luajit
```ruby
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz
cd LuaJIT-2.0.2
make
make install PREFIX=/opt/app/luajit
ln -s /opt/app/luajit/lib/libluajit-5.1.so.2 /lib64/
或者echo "/opt/app/luajit/lib" /etc/ld.so.conf; ldconfig
export LUAJIT_LIB=/opt/app/LuaJIT/lib
export LUAJIT_INC=/opt/app/LuaJIT/include/luajit-2.0
```

- 安装ngx_devel_kit、lua-nginx-module、pcre

	```ruby
    cd /opt/soft
    
	wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.17rc2.zip

	unzip v0.2.17rc2

	wget https://github.com/chaoslawful/lua-nginx-module/archive/v0.7.4.zip

	unzip v0.7.4

	wget http://blog.s135.com/soft/linux/nginx_php/pcre/pcre-8.10.tar.gz

	tar zxvf pcre-8.10.tar.gz

	cd pcre-8.10/

	./configure --prefix=/opt/app/pcre-8.10

	make && make install

	cd ..

	```

- 安装编译nginx
```ruby

	tar xvzf nginx-1.6.1.tar.gz

	cd nginx-1.6.1

	./configure --prefix=/opt/app/nginx-1.6.1 --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-http_image_filter_module --with-pcre --add-module=/opt/soft/lua-nginx-module --add-module=/opt/soft/ngx_devel_kit-0.2.19

	make
    
    make install
```

- 下载ngx_lua_waf
```ruby
	git clone https://github.com/loveshell/ngx_lua_waf.git
    
    mv ngx_lua_waf.git /opt/app/nginx/conf/waf    
```

- nginx.conf http段配置
```ruby
	lua_package_path "/usr/local/nginx/conf/waf/?.lua";
    lua_shared_dict limit 10m;
    init_by_lua_file  /usr/local/nginx/conf/waf/init.lua; 
    access_by_lua_file /usr/local/nginx/conf/waf/waf.lua;
```

- 配置config.lua里的waf规则目录(一般在waf/conf/目录下)
```ruby
 RulePath = "/opt/app/nginx/conf/waf/wafconf/"
```
- 配置文件详细说明
```ruby
	RulePath = "/usr/local/nginx/conf/waf/wafconf/"
    --规则存放目录
    attacklog = "off"
    --是否开启攻击信息记录，需要配置logdir
    logdir = "/usr/local/nginx/logs/hack/"
    --log存储目录，该目录需要用户自己新建，切需要nginx用户的可写权限
    UrlDeny="on"
    --是否拦截url访问
    Redirect="on"
    --是否拦截后重定向
    CookieMatch = "on"
    --是否拦截cookie攻击
    postMatch = "on" 
    --是否拦截post攻击
    whiteModule = "on" 
    --是否开启URL白名单
    ipWhitelist={"127.0.0.1"}
    --ip白名单，多个ip用逗号分隔
    ipBlocklist={"1.0.0.1"}
    --ip黑名单，多个ip用逗号分隔
    CCDeny="on"
    --是否开启拦截cc攻击(需要nginx.conf的http段增加lua_shared_dict limit 10m;)
    CCrate = "100/60"
    --设置cc攻击频率，单位为秒.
    --默认1分钟同一个IP只能请求同一个地址100次
    html=[[Please go away~~]]
    --警告内容,可在中括号内自定义
    备注:不要乱动双引号，区分大小写
```
