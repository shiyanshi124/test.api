# test.api

# README #

Bibox 交易所 api 相关说明。

### 官网 ###

https://www.bibox.com/

# 如何使用apikey进行API请求？ #
### 1.申请apikey ###

* 登陆 https://www.bibox.com/ 进行操作
* 系统会生成 key 和 secret，请秘密保存

### 2.设置apikey ###

* 登陆 https://www.bibox.com/ 进行操作
* 可以设置每个 apikey 的权限

### 3.应用apikey ###

* 构建请求参数

```
PATH: apipath
METHOD: POST
PARAMS: 
{
	"cmds": "[{\"cmd\":\"user/userInfo\",\"body\":{}}]",
	"apikey":"52135958969bedca0809ac10b9caba758022b0a6",
	"sign":"6a21e39e3f68b6fc2227c4074c7e6a6c"
}
```

```
说明：
	apipath: api请求路径
	cmds: 请求的 api 参数信息
		cmd: 请求具体的api路径参数，参考下面的 cmd 列表
		body: cmd 对应的参数，参考下面的 cmd 列表
	apikey: 系统生成的 apikey
	sign: 通过 secret 对 cmds 进行 md5 转换后的结果，作为签名
```

* 生成 sign 方法

```
// node.js
	var CryptoJS = require("crypto-js");
	var secret = '7b58254791ada6c0194e6341953f862aff9a91b5';
	var cmds = '[{"cmd":"user/userInfo","body":{}}]';
	var sign = CryptoJS.HmacMD5(cmds, secret).toString();//6a21e39e3f68b6fc2227c4074c7e6a6c
```

# api列表 #

### 格式规范 ###

* 请求格式

```
{
	cmds: [
		{
			cmd: 'name1',//like user/userInfo
			body: {
				//params1
			}
		},
		{
			cmd: 'name2',//like other
			body: {
				//params2
			}
		}，
		......
	]，
	apikey:"**************",
    sign:"**************"
}
```

* 请求失败返回格式

```
{
	error: {
		code: '1000',
		msg: 'something error'
	}
}
```

```
说明：
	code: 错误码
	msg: 错误描述
```

* 请求成功返回格式

```
{
	result: [
		{
			cmd: 'name2',//like user/userInfo
			result: result2 //any type data
		},
		{
			cmd: 'name1',//like user/userInfo
			result: result1 //any type data
		},
		......
	]
}
```

```
说明：
	result 数组里的数据对应请求的数组 cmds，不保证顺序一致性
	数组中的每一项单独有对应的 result，数据类型是对应 cmd 请求的返回类型
```

### 查询相关api ###

#### /v1/user ####

* apipath: /v1/user

* cmd 列表

```
cmd: "user/userInfo"
body: {}
```

#### /v1/credit ####

* apipath: /v1/credit 

* cmd 列表

```
// 获取自己的放贷信息
cmd: "lendOrder/get"
body: {
    coin_id //币种 id, 可以通过 transfer/coinList 获取币种列表
    status //状态 0:进行中 1:已部分收回 2:已全部收回
    page //第几页，从1开始
    size //要几条
}
```

```
// 获取自己的借贷信息
cmd: "borrowOrder/get"
body: {
    coin_id //币种 id, 可以通过 transfer/coinList 获取币种列表
    status //状态 0:未成交 1:欠款中 2:已部分还款 3:已全部还款 4:取消借款
    page //第几页，从1开始
    size //要几条
}
```

```
// 获取自己的预定放贷信息
cmd: "lendOrderbook/get"
body: {
    coin_id //币种 id, 可以通过 transfer/coinList 获取币种列表
    status //状态 0:进行中 1:已部分完成 2:已全部完成
    page //第几页，从1开始
    size //要几条
}
```

```
// 获取信用账户资产
cmd: "borrowOrder/getAssets"
body: {}
```

#### /v1/transfer ####

* apipath: /v1/transfer 

* cmd 列表

```
// 币种列表
cmd: "transfer/coinList"
body: {}
```

```
// 获取充值地址
cmd: "transfer/transferIn"
body: {
    coin_symbol //币种符号，可从 transfer/coinList 中获得，如：BTC
}
```

```
// 查询可用提现地址，可提现金额，手续费
cmd: "transfer/transferOutInfo"
body: {
    coin_symbol //币种符号，可从 transfer/coinList 中获得，如：BTC
}
```

```
// 获取充值列表
cmd: "transfer/transferInList"
body: {
    search,  //币种符号
    filter_type,  //0：全部，1：进行中，2：已完成，3：失败
    page //第几页，从1开始
    size //要几条
}
```

```
// 获取提现列表
cmd: "transfer/transferOutList"
body: {
    search,  //币种符号
    filter_type,  //0：全部，1：进行中，2：已完成，3：失败
    page //第几页，从1开始
    size //要几条
}
```

```
// 我的资产
cmd: "transfer/assets"
body: {}
```

```
//用户账单
cmd: "transfer/bills"
body: {
    coin_id, //币种id，0:全部，否则从 transfer/coinList 中获得
    days, // 最近多少天的
    type,  // -1：'全部', 0：'充值', 2：'提現', 3：'賣出', 4：'手續費', 5：'買入', 6：'系统奖励', 7：'推荐返现'
    page //第几页，从1开始
    size //要几条
}
```

### 交易相关api ###

#### /v1/credit ####

* apipath: /v1/credit 

* cmd 列表

```
// 发起借款
cmd: "borrowOrder/book"
body: {
    coin_id,  //币种 id, 可以通过 transfer/coinList 获取币种列表
    amount,  // 借款数量
    interest_rate,  // 利率
    period  //借款天数
}
```

```
// 取消借款
cmd: "borrowOrder/cancel"
body: {
    order_id //借款单号
}
```

```
// 主动还款
cmd: "borrowOrder/refund"
body: {
    order_id, // 借款单号
    amount // 还款数量
}
```

```
// 发起放贷
cmd: "lendOrderbook/publish"
body: {
    coin_id,  //币种 id, 可以通过 transfer/coinList 获取币种列表
    amount,  // 数量
    interest_rate,  // 利率
    period,  // 放贷天数
    expire // 有效期
}
```

```
// 取消放贷
cmd: "lendOrderbook/cancel"
body: {
    orderbook_id //预定放贷单的id号
}
```

### 提现相关api ###

#### /v1/transfer ####

* apipath: /v1/transfer 

* cmd 列表

```
// 申请提现
cmd: "transfer/transferOut"
body: {
    totp_code,  // google 验证码
    trade_pwd,  // 交易密码
    coin_symbol,  // 币种符号，可从 transfer/coinList 中获得，如：BTC
    amount,  // 提币数量
    addr,  // 提现地址
    addr_remark // 地址别名
}
```
