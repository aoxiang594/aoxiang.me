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
			if string.find(body,"20900103") or string.find(body,"20.1.7") or string.find(body,"HLX") or string.find(body,"8.88") then
				ngx.exit(ngx.HTTP_NOT_FOUND)
			end
		end
	';
	# 接着往下继续执行php,正常请求
}

```

这里提一个点`access_by_lua`用来进行访问控制，之前一直用`content_by_lua_block`，匹配成功以后，就不会继续执行php了。一定要用`access_by_lua`

[https://github.com/openresty/lua-nginx-module](https://github.com/openresty/lua-nginx-module)