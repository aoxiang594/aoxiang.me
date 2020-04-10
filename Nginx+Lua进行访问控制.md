# Nginx+Lua进行访问控制
#### 背景
前阵子帮朋友做了个服务端，用来给他APP做一些基本的注册、付费等功能。但是他APP被盗版了，话说这盗版也是厉害，有时候注册量远远高于正版注册量，昨天盗版一天注册6000人，正版才2000人。于是乎服务器扛不住了，注册接口超时，客户端自动发起重连，又超时，又重连，如此往复，最后正常用户都没法访问了。
#### 处理方案
要保证现有客户的能正常使用，还要限制盗版，那必须得知道盗版跟正版有啥区别了，检查过后，发现盗版应用在接口中有一个参数跟正版不一致。那当然得从参数上做限制。
两个方案:
1. PHP接收到数据后，判断然后剔除。
2. 使用Lua在Nginx层直接剔除

不用看，当然能在Web服务剔除最好了。为啥呢？如果在应用层提出，一样会进入PHPFPM消耗资源。
#### 具体实施
安装Lua过程不表，可以参考[https://segmentfault.com/a/1190000018641801](https://segmentfault.com/a/1190000018641801)
```shell
lua_need_request_body on;
location ~ \.php {
	access_by_lua '
		local uri = ngx.var.request_uri
		if string.find(uri,"/api/register") ~= nil then
			local body = ngx.req.get_body_data()    
			if string.find(body,"20200101") or string.find(body,"20200102") or string.find(body,"9.9.9") or string.find(body,"8.8.8") then
				ngx.exit(ngx.HTTP_NOT_FOUND)
			end
		end
	';
	# 接着往下继续执行php,正常请求
}

```

这里提一个点`access_by_lua`用来进行访问控制，之前一直用`content_by_lua_block`，匹配成功以后，就不会继续执行php了。一定要用`access_by_lua`

[https://github.com/openresty/lua-nginx-module](https://github.com/openresty/lua-nginx-module)



---

2020年04月08日简单优化一下。随着盗版越来越多，盗版的关键字丢数组里

```shell
lua_need_request_body on;
location / {  
  access_by_lua '
	local uri = ngx.var.request_uri
	if string.find(uri,"/api/register") ~= nil then
		local body = ngx.req.get_body_data()
		local denyList = {"20200101","20200102","9.9.9","8.8.8"}
		for i = 1,#denyList do
			if string.find(body,denyList[i]) then
				ngx.header.content_type = "application/json";
                ngx.say(\'{"toast":"您正在使用盗版软件"}\');       
                ngx.exit(ngx.HTTP_OK);
			end
		end
	end
';
}  

```



细心的朋友会发现，上面的for循环中，i是从1开始的，没毛病，在很多语言中长度是从0开始，但是在lua里面，长度居然是从1开始。

---

2020年04月10日在优化一波，将所有处理失败的请求全部记录下来

在处理这个功能的时候，发现一个神奇的bug

在lua中，比如匹配`8.8.8`里面的`点`居然会匹配成任意字符，也就是说，`8d8c8`也会被匹配成功，导致有一部分用户被误判。`点`需要用`%p`来表示

将错误信息，写入文件中。

```shell
file = io.open("/tmp/lua.txt", "a")
io.output(file)
io.write(date)
io.write(body)
io.close(file)
```

以下是完整代码

```shell
lua_need_request_body on;
location / {  
  access_by_lua '
	local uri = ngx.var.request_uri
	if string.find(uri,"/api/register") ~= nil then
		local body = ngx.req.get_body_data()
		local denyList = {"20200101","20200102","9%p9%p9"}
		for i = 1,#denyList do
			if string.find(body,denyList[i]) then
					local date=os.date("%Y-%m-%d %H:%M:%S");
          file = io.open("/tmp/lua.txt", "a")
          io.output(file)
          io.write(date)
          io.write(body)
          io.close(file)
					ngx.header.content_type = "application/json";
          ngx.say(\'{"toast":"您正在使用盗版软件"}\');       
          ngx.exit(ngx.HTTP_OK);
			end
		end
	end
';
```

