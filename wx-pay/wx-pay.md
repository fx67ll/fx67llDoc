# 微信支付前端开发指南

### 使用场景
1. 微信公众号内嵌H5网页调用微信支付  
2. 在微信浏览器中的网页唤起微信支付界面  

#### 详情可以查阅微信支付官方文档 [链接地址](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)  

### 功能思路
1. 后台整合微信官方的相关下单接口，编写预下单接口提供前台调用，并返回订单相关参数  
2. 前台按照下方示例代码，传递相关订单参数后唤起微信支付界面  
3. 后台整合微信官方订单状态查询接口，编写订单状态查询接口提供前台调用，并返回订单支付状态
4. 用户支付完后，通过微信返回的参数以及步骤3的订单状态接口来判断订单是否支付成功，完成后续业务逻辑操作  

### 前端代码

#### 1. 设置判断是否可以调用微信支付的变量
```
let can_pay;
```

#### 2. 判断WeixinJSBridge对象是否存在当前浏览器中，该对象只在微信浏览器中有效，并在回调函数中通过设置判断变量来确认微信支付可用，调用微信相关API
```
if (typeof WeixinJSBridge === 'undefined') {
	if (document.addEventListener) {
		document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
	} else if (document.attachEvent) {
		document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
		document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
	}
} else {
	onBridgeReady();
}

onBridgeReady() {
	can_pay = true;
	WeixinJSBridge.call('hideOptionMenu'); // 微信API，用于隐藏右上角按钮，与之对应的是显示右上角按钮（showOptionMenu），还有显示隐藏底部导航栏的API（showToolbar/hideToolbar）
}
```

#### 3. 如果判断变量为真则开始通过对象唤起支付界面，并在回调函数中完成其他业务逻辑操作，注意查看注释内容不要直接复制全部代码不做修改
```
if (can_pay) {
	WeixinJSBridge.invoke(
		'getBrandWCPayRequest',
		{
			// res.data对象为后台返回参数对象
			appId: res.data.appId, // 公众号id，为当前服务商号绑定的appid
			timeStamp: res.data.timeStamp, // 时间戳，自1970年以来的秒数
			nonceStr: res.data.nonceStr, // 随机字符串，不长于32位
			package: res.data.payInfo, // 订单详情扩展字符串，统一下单接口返回的prepay_id参数值
			signType: res.data.signType, // 签名方式，默认为MD5，支持HMAC-SHA256和MD5
			paySign: res.data.sign // 签名
		},
		// 微信支付官方接口的返回函数
		function(res) {
			if (res.err_msg === 'get_brand_wcpay_request:ok') {
				console.log('这里调用后台的订单状态查询函数，如果后台查询成功则微信支付成功，反之失败')
			} else {
				console.log('支付异常')
			}
		}
	);
}
```

我是[fx67ll.com](https://fx67ll.com)，如果您发现本文有什么错误，欢迎在评论区讨论指正，感谢您的阅读！  
如果您喜欢这篇文章，欢迎访问我的 [本文github仓库地址](https://github.com/fx67ll/fx67llDoc/blob/main/wx-pay/wx-pay.md)，为我点一颗Star，Thanks~ :)  
***转发请注明参考文章地址，非常感谢！！！***