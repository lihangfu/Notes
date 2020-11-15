# HTTP协议

HTTP 协议【超文本传输协议】，协议详细规定了浏览器和万维网服务器直接互相通信的规则。

约定，规则

## 请求报文

重点是格式与参数

```
行		POST	/id=1	HTTP/1.1
头		Host：lhf223.cn
		 Cookie：name=lhf
		 ...
空行
体		username=admin&password=admin
```



## 响应报文

```
行		HTTP/1.1	200		OK
头		Content-Type：text/html;charset=utf-8
		 Content-length：2048
		 ...
空行
体		<html>...</html>
```



