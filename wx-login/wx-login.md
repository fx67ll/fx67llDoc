# 微信登录前端开发指南

### 使用场景
1. 微信公众号内嵌H5网页调用微信登录  
2. 在微信浏览器中的网页唤起微信登录界面  
3. 详情可以查阅微信登录 [官方文档地址](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)

### 功能思路
1. 后台先在微信给开发者提供的测试账号平台上创建应用，并把前台开发同学的微信添加到这个平台中，用于提供测试公众号，[平台地址](https://mp.weixin.qq.com/debug/cgi-bin/sandboxinfo?action=showinfo&t=sandbox/index)  
2. 前台拼装各种微信登录必须的参数，并将页面跳转到微信指定的连接获取微信登录`code`参数  
3. 前台解析返回的链接，获取链接中的参数，并将`code`参数传给后台  
4. 由于后续获取的参数安全等级较高，所以后续的操作均由后台完成，并将数据存储在服务端，后台通过`code`参数换取`access_token`参数和`openid`参数并存储在服务端  
5. 后台再通过`access_token`参数和`openid`参数换取用户详细信息，并将这4/5两步包装成接口，让前台调用，返回前端登录状态完成微信登录操作  
6. 如果`access_token`参数失效，使用`refresh_token`刷新即可，详细操作查阅文档，一般用不到这步  

### 前端代码

#### 1. 获取code
```
function getWXcode() {
	
	// 公众号的唯一标识，找公众号管理员提供
	var APPID = '公众号的appid';
	
	// 授权后重定向的回调链接地址，要使用encodeURIComponent()对其进行编码处理
	// !!!注意回调的域名必须先让公众号管理员添加到安全域名中，否则无法完成跳转
	var REDIRECT_URI = encodeURIComponent('授权后跳转回的页面');
	
	// 常量，返回类型，填写code即可
	var RESPONSE_TYPE = 'code';
	
	// 常量，应用授权作用域
	// snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且， 即使在未关注的情况下，只要用户授权，也能获取其信息 ）
	// snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid）
	var SCOPE = 'snsapi_userinfo' || 'snsapi_base';
	
	// 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
	// 除了此项是非必须参数，其他都是必须要带的参数
	var STATE = '可以自定义的返回参数';
	
	// 无论直接打开还是做页面302重定向时候，必须带'#wechat_redirect'参数
	// 由于授权操作安全等级较高，所以在发起授权请求时，微信会对授权链接做正则强匹配校验，如果链接的参数顺序不对，授权页面将无法正常访问
	location.href =
		`https://open.weixin.qq.com/connect/oauth2/authorize?appid=${APPID}&redirect_uri=${REDIRECT_URI}&response_type=${RESPONSE_TYPE}&scope=${SCOPE}&state=STATE#wechat_redirect `;

}
```

#### 2. 解析参数

```
function parseData() {
	
	// 截取地址栏?后面的内容
	var search = location.search.slice(1);
	
	// 用&标识分割成数组
	var arr = search.split('&');
	
	// 处理数组转换成对象，并返回
	var result = {};
	arr.forEach(function(item) {
		var itemArr = item.split('=');
		result[itemArr[0]] = itemArr[1];
	});
}
```

#### 3. code传给后台，后台对接微信登录后，返回登录状态提示前端登录成功还是失败，并完成后续业务逻辑