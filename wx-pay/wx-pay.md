# 微信支付

### 使用场景
在微信浏览器中唤起微信支付界面，[官方文档地址](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)  

### 前端代码
```
// 判断是否可以调用微信支付的变量
let can_pay;

// 判断WeixinJSBridge对象是否存在当前浏览器中，该对象只在微信浏览器中有效
if (typeof WeixinJSBridge == 'undefined') {
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
	WeixinJSBridge.call('hideOptionMenu');
}

// 微信支付的准备工作完成之后，从后台拿到相关支付参数，唤起手机界面上的微信支付界面
if (can_pay) {
	WeixinJSBridge.invoke(
		'getBrandWCPayRequest',
		{
			// res.data对象为后台返回参数对象
			appId: res.data.appId, //公众号名称，由商户传入
			timeStamp: res.data.timeStamp, //时间戳，自1970年以来的秒数
			nonceStr: res.data.nonceStr, //随机串
			package: res.data.payInfo,
			signType: res.data.signType, //微信签名方式
			paySign: res.data.sign //微信签名
		},
		// 微信支付官方接口的返回函数
		function(res) {
			if (res.err_msg == 'get_brand_wcpay_request:ok') {
				console.log('这里调用后台的订单状态查询函数，如果后台查询成功则微信支付成功，反之失败')
			} else {
				console.log('支付异常')
			}
		}
	);
}

```