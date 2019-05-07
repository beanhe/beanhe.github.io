---
layout: post
title: OpenResty防恶意攻击举例
category: web
tags: [__beanhe,openresty,nginx,lua,web]
---
    
#### OpenResty(nginx+lua+redis)防止恶意访问入门举例

##### 环境
- Web服务器：OpenResty1.11.2.1

#### 需求
- 需求背景：用户注册获取验证码的`API`：`/user/signup/otp?cellphone=<CELLPHONE>&token=<TOKEN>`，其中`<CELLPHONE>`代表用户输入的手机号码，`<TOKEN>`代表页面显示的图片验证码，正常情况下两者必须都非空切满足条件才能获取到手机验证码，但是由于后端代码`BUG`导致只对手机号码进行了判断，而`<TOKEN>`参数可为空，此时有攻击者发现此`BUG`就使用合法的手机号码疯狂调用此`API`导致后端接受请求后频繁发送手机验证码，消耗短信接口费用
- 需求：只要发现`API`中`TOKEN`为空就拒绝请求（403），并将请求中的手机号码和用户的访问IP存储到`Redis`服务器，给开发充足时间解决`BUG`，并在解决后检查是否有无效的注册予以清除

#### 方法
- 编写`Lua`脚本，内容如下：

```
$ cat /usr/local/openresty/lualib/ngx/userapi.lua
local function close_redis(red)
     if not red then
          return
     end
     --释放连接
     local pool_max_idle_time = 10000 --毫秒
     local pool_size = 100 --连接池大小
     local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)

     if not ok then
          ngx_log(ngx_ERR, "set redis keepalive error : ", err)
     end
end

local redis = require "resty.redis"
local red = redis:new()
red:set_timeout(1000)
local ip = "172.16.0.1" -- REDIS服务器IP
local port = 6379 -- REDIS端口
local ok, err = red:connect(ip,port)
if not ok then
     return close_redis(red)
end
-- clientIP 为请求的客户端IP
local clientIP = ngx.req.get_headers()["X-Real-IP"]
if clientIP == nil then
    clientIP = ngx.req.get_headers()["x_forwarded_for"]
end
if clientIP == nil then
    clientIP = ngx.var.remote_addr
end


local uri = ngx.var.uri
if uri == "/user/signup/otp" then
    local cellphone = ngx.var.arg_cellphone -- 获取请求中的手机号码
    local token = ngx.var.arg_token -- 获取请求中的token
    if (token == nil or token == "") then -- 如果token不存在或者为空
        if (cellphone ~= nil and cellphone ~= "") then 
            local blockIpListKey = "ngx:smember:blockiplist" -- 攻击者客户端IP列表
            local blockTelListKey = "sms:set:block" -- 攻击请求中的手机号码列表
            res, err = red:sadd(blockIpListKey, clientIP)
            res, err = red:sadd(blockTelListKey, cellphone)
        end
        ngx.exit(ngx.HTTP_FORBIDDEN) -- 丢弃请求
        return close_redis(red) -- 关闭连接
    end
end

close_redis(red)
```

- 配置`nginx`:

```
$ cat /etc/nginx/nginx.conf
http {
	### Lua path config
    lua_package_path "/usr/local/openresty/lualib/?.lua;;";
    lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
    ###
	upstream user_service {
		server 10.0.0.1:8888;
		server 10.0.0.2:8888;
	}
	server {
		server_name domain.com;
		location /user {
			access_by_lua_file "/usr/local/openresty/lualib/ngx/userapi.lua";
			proxy_pass http://user_service;
		}
	}
}
```

- 重启`nginx`: `service nginx reload`