# RuiWen-IQTest

软件项目管理-瑞文标准智力测验系统



## 项目说明

1、本项目采用前后端分离，前端使用微信小程序，后端使用SpringBoot+MySQL+Redis。

2、后端项目打包成`jar`包部署到腾讯云服务器上，并在后台启动该项目，利用`Nginx`实现反向代理。

## 注意事项

1、由于小程序发布到线上后数据交互要用`https`进行通信，故本项目需要你买个便宜的`ECS`，并且配置好域名证书。

2、数据库表中某些字段值前缀含`http://127.0.0.1:8080`，把这些前缀名通通改成你的域名+端口（可选择），如：`https://hostip:8080/...`，其中端口非必须，可用`Nginx`反向代理转发请求内部路由，隐藏原先的请求路径。

3、因小程序上线是`https`通信，故还需要在微信开发者后台设置信任访问你前面设置的域名。

## 后端功能设计的一些思考

1、微信小程序中当用户授权之后，执行`wx.login`方法将`code`传送到后端，后端根据这个`code`和微信小程序开发者的`appid`、`secret`等几个重要参数请求微信小程序官方接口`https://api.weixin.qq.com/sns/jscode2session`获取登录用户的`openid`和`session_key`，随机生成一个`UUID`作为`key`，将这2个参数值绑定在一起作为`key`的`value`保存到`Reids`数据库中（键值对过期时间为1周）；为保证数据的安全性，只返回给前端这个随机生成的`UUID`，前端将其保存到本地缓存当中，作为客户端与服务端登录状态的记录标识，每次前端向服务端发起请求时都携带这个`UUID`，后台就根据这个`UUID`去`Redis`中获取当前登录用户的唯一标识`openid`，然后进行后续的业务处理。

2、由于设置了`UUID`的过期时间，在某时刻其过期了，当前端发起请求到服务端后，发现`Redis`中找不到这个`UUID`，那么用户测试的记录就白费了。改进的方法是：`SpringBoot`自定义一个拦截器，在`preHandle`方法中获取这个`UUID`，检查其过期时间是否不大于半个小时，若是则重新设置其过期时间为1周。

