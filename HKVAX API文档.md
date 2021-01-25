# API 概述

## 基本介绍

欢迎使用HKVAX 开发者文档。该文档概述了HKVAX的API介绍、接入方式和接口说明。

API分为三类：行情、交易和个人数据查询。您可以根据自身需求建立不同权限的API，并利用API进行自动交易。

行情API提供市场的行情数据，所有行情接口都是公开的。交易和个人数据查询API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。

## 交易规则

### 撮合规则

HKVAX按照“价格优先-用户优先-时间优先”的规则进行撮合。所有订单类型的规则相同。

价格优先是指，价格较高的买入订单优先于价格较低的买入成交，价格较低的卖出订单优先于价格较高的卖出成交；

用户优先是指，若挂单价格相同，客户的订单将优先于HKVAX公司账户订单及HKVAX员工的订单成交；

时间优先是指，同价位、同用户类型的挂单，依照挂单时间的先后决定成交顺序，先挂单的先成交，后挂单的后成交。

### 订单生命周期

订单进入撮合引擎后是"未成交" `MISMATCH `状态；

订单开始成交，状态会变成“部分成交” `PARTIAL `状态；

直到一个订单被撮合而全部成交，那么它会变成"全部成交"`DEAL`状态；

如果一个未成交的订单订单撤单成功，那么它会变成"未成交已撤消"`CANCEL_MISMATCH`状态；

部分成交的订单订单撤单成功，那么它会变成"部分成交已撤消"`CANCEL_PARTIAL `状态；

另外，撤销订单时，若系统正在处理，撤单还没有结果响应，则订单为"正在撤销"`FIRSTTRIAL `状态。

### 禁止自成交

HKVAX不允许进行自成交：用户订单成交时，若交易对手方为自己，则系统会自动撤销挂单时间较晚的一个订单，以防止自成交的发生。

## 费用

HKVAX采用maker-taker收费规则，为鼓励挂单，maker挂单成交的手续费会比taker吃单成交的手续费低。Maker：0.2%，Maker：0.1%

## 服务器

HKVAX的服务器在香港。

## 模拟环境

模拟环境可用于测试API连接性和交易。模拟环境的功能是独立于正式环境的，您需要单独注册、登录、创建API密钥。

此外，一旦您完成注册，我们会提供充足的模拟资金供您测试。

### 模拟环境地址

在测试API连接性时，请确保使用以下URL:

**网站**: [https://simtrade.hkvax.com](https://simtrade.hkvax.com/)

**REST API**: https://api.simtrade.hkvax.com/v1 



# 接入API

## 使用流程

使用API需要先完成API Key的创建和权限配置。在API管理页面点击创建API Key，然后根据此文档详情进行开发和交易。

每个用户可创建10组Api Key，每个Api Key可对应设置行情、交易、个人信息查询三种权限。创建成功后请务必记住以下信息：

- `Access Key` API 访问密钥
- `Secret Key` 签名认证加密所使用的密钥（仅创建成功时可见）

## 接入URLs

[https://api.trade.hkvax.com/v1](https://api.trade.hkvax.com/v1)

## 请求格式

所有请求基于https协议。

### 请求参数

一个合法的请求由header部分和requestbody部分组成。

**header**


| key          | 类型   | 是否必须 | 备注                                   |
| ------------ | ------ | -------- | -------------------------------------- |
| APIKEY       | string | true     | 公钥                                   |
| SIGNATURE    | string | true     | **请求数据签名串**                     |
| CEXTOKEN     | string | false    | 获取apikey接口时需要,token需要登录获取 |
| DEVICEID     | string | true     | 随机字符串(可固定一值)                 |
| DEVICESOURCE | string | true     | 固定值web                              |
| Lang         | string | true     | zh-CN或en-US                           |
| Client       | string | true     | 固定值，由cex分配                      |

**requestbody**

| 字段名 | 字段类型 | 是否必须 | 备注                                                         |
| ------ | -------- | -------- | ------------------------------------------------------------ |
| lang   | string   | true     | 与header保持一致                                             |
| data   | 任意类型 | false    | string/int/array/jsonObject/jsonArray<br>其他类型都可，实际类型以接口说明为准 |

**data参数为string示例如下**

```json
{
    "lang" : "zh-CN",
    "data" : "123"
}
```

**data参数为json示例如下**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123,
        "currency" : "USD"
    }
}
```

### 响应参数

所有的接口返回都是JSON格式。在JSON最上层有两个表示请求状态和属性的字段: "code", "msg"，实际的接口返回内容在"data"字段里。

**responseBody**

| 字段名 | 字段类型 | 备注                                                         |
| ------ | -------- | ------------------------------------------------------------ |
| code   | string   | 000000表示成功，其他表示失败                                 |
| msg    | string   | 返回信息                                                     |
| data   | 任意类型 | string/int/array/jsonObject/jsonArray<br>等类型都可实际类型以接口说明为准 |

## 签名认证

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，所有接口均必须使用您的 API Key 做签名认证。每一个API Key需要有适当的权限才能访问相应的接口，在使用接口前请确保您的API Key有相应的权限。

### 签名步骤

1.对API Key、CEXTOKEN 、data参数按照文档给定的请求参数顺序进行排序；

2.对以上的string进行RSA运算，得到签名。

**签名实例**

以下实例使用SHA256withRSA签名：

   公钥：

```java
   MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCScP9BF63DY5z/WgCqAXpUSbWnY7wYLGT+5pn2NWjRuKmKF4aMxbSKmiGny3Sno+2yvqFqKWv/lz56yaylPKucN2WGdH0HKYnxhXWbaWLK23gBVfHu6YZvh8sBzJ7lOCXvLwMCRGNfDqFNj2TPmA0wMnop+0FXQOVe/q1USbFXPwIDAQAB
```

   私钥：

```text
   MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBAJJw/0EXrcNjnP9aAKoBelRJtadjvBgsZP7mmfY1aNG4qYoXhozFtIqaIafLdKej7bK+oWopa/+XPnrJrKU8q5w3ZYZ0fQcpifGFdZtpYsrbeAFV8e7phm+HywHMnuU4Je8vAwJEY18OoU2PZM+YDTAyein7QVdA5V7+rVRJsVc/AgMBAAECgYB33wMytz1HqWzEIVpVzyvhfwyxXpSDfSOW/BCfV4zbzzsIjMVYyiVFJ3HRNlvhNfDG1gCvNATxjU5ZmGg4Qfd+hNqB2X4BeeK3pUNbesz5SNXPGhu/0oIWVN5oKZJjwzhB8aaSSxNyU73g36NyvkfSJMOMvQMOqtlnAPAwL3biIQJBAM1NtEwtn2IcogYsF315qgqfefLPtFr/szbczVSkOng/+uVNmcxDrfA3bwu0+IqV5Yw/+57rDjkuBuU/yyO6aekCQQC2mlA3GqkNIFIo/C+W7aRhpJS2DF+J6EznZf82hgD0C/3D/npC0k/Diy89gglNpaKuSx/IxJjPLUpJl0uNe9bnAkEAnT7q3X4EGY18u+WBiGVrS/+h08wqg5hdl6O+0RmIfxnh/UdWiRE9ZEPRFdJimyL8UlOfUbUPi9QpC+W0nYTmIQJBAIy218/O/Kz/1jB9PhMZqE4SbQLpAAqe9/xtrkEO/NcUEochqIer2AnBTTMh7Rdn57hWbfTiAzvME+4n5/Hsl8sCQQC4/K4Q9r2frQcMRUNEu42tMkA1w1XgC8fRC7/9K4Y4+XLJIx7YN9hoKkyvxbmM35yRiwYUoUJrRuLNaQ32trRu
```

   请求体：

```json
{"data":{"currency":"USD"},"lang":"zh-CN"}
```

  签名信息：

```text
KsxKzaDdcoufq8Nz3xHV0rz1t4ewswZNKrY3Utv3ltR/xcMztm0LUDcGo9B1YTz8lilzLIlknj/VcFsYzBEtonhA0nxcuRVkfcvkY5gHCcaGBvrSzjkDNSmr1tsPcoaDShy5MxEJeiLNn8qi6pFxtYWT8LHMvM+N1/jUSRVVpWI=
```

## 分页

分页后，data参数由分页对象和业务数组对象组成。

**分页对象**

字段名 | 字段说明 | 字段类型 | 备注
---|---|---|---
pageSize | 总共页数 | int | &nbsp;
currPage | 当前页 | int | 从1开始
totalRows | 总共行数 | int | &nbsp;
pageRows | 当前行数 | int | 分页大小(最大1000)

**成功示例一**

```json
{
    "code" : "000000",
    "msg"  : "success",
    "data" : 15
}
```

**成功示例二(分页示例)**

```json
{
    "code" : "000000",
    "msg"  : "success",
    "data" : {
        "pagination" : {
            "pageSize" : 22,
            "currPage" : 1,
            "totalRows" : 212,
            "pageRows" : 10
        },
        "list" : [
            {
                "orderNo" : 123,
                "currency" : "BTC",
                "amount" : 10,
                "userNo" : 11
            }, {
                "orderNo" : 222,
                "currency" : "BTC",
                "amount" : 3,
                "userNo" : 12
            }
        ]
    }
}
```


**失败示例**
```json
{
    "code" : "999006",
    "msg" : "订单号不存在",
    "data" : null
}
```

## 标准规范

### 时间戳

除非另外指定，API中的所有时间戳均为Unix时间戳。

### 精度

建议您在发起请求时，将数字转换为字符串以避免截断和精度错误。

| 交易对     | 挂单数量精度 | 挂单价格精度 | 挂单金额精度 |
| ---------- | ------------ | ------------ | ------------ |
| BTC/USD    | 4            | 2            | 2            |
| ETH/USD    | 4            | 2            | 2            |
| BCHABC/USD | 4            | 2            | 2            |
| LTC/USD    | 4            | 2            | 2            |
| XRP/USD    | 4            | 6            | 6            |
| USDT/USD   | 4            | 4            | 2            |

## 响应参数错误码

| 错误码 | 说明                | 备注                                 |
| ------ | ------------------- | ------------------------------------ |
| 100006 | 参数异常            | 请求参数异常或header参数异常         |
| 100037 | 用户与订单不匹配    | 用户请求撤单的单号不是自己的订单     |
| 100038 | 订单不存在          |                                      |
| 100039 | 订单已撤销          |                                      |
| 100040 | 订单已完成          |                                      |
| 100041 | 订单撤销失败        |                                      |
| 100043 | 不支持的交易对      | 该交易对已下线或者交易对不存在       |
| 100044 | 订单动作不支持      | 传递了买卖之外的动作                 |
| 100046 | 交易对不可下单      | 该交易对禁止下单                     |
| 100047 | 禁止交易            | 用户组权限限制                       |
| 100048 | 订单价格或数量无效  | 订单数量或价格过大                   |
| 100049 | 订单价格或数量无效  | 订单数量或价格小于交易对设置的最小值 |
| 100050 | 订单价格或数量无效  | 订单数量或价格大于交易对设置的最大值 |
| 100051 | 价格超过限制        |                                      |
| 100052 | 订单类型不支持      | 传递错误的订单类型                   |
| 100054 | 余额不足            |                                      |
| 100055 | 下单失败            |                                      |
| 100112 | 账户不存在          | 资金账户未生成                       |
| 100212 | 安全认证失败        | 参数或结果的安全校验不通过           |
| 100152 | 订单下单频繁        | 超过订单限流                         |
| 500100 | 签名错误            |                                      |
| 500202 | ApiKey 没有访问权限 |                                      |

## 接口分组

| 接口分组     | 包含接口                                                     |
| ------------ | ------------------------------------------------------------ |
| 行情数据     | 24小时行情(POST /quotation/quotation/query/symbolQuotationGroupBy)<br />深度数据 ( POST /quotation/quotation/query/depthData) <br />K线 (POST /quotation/kline/get/quotationHistory)<br />最新成交记录（POST /quotation/quotation/query/latestTradeQuotation)<br />订单簿数据(POST /quotation/order/query/orderBook)<br />查询最新的成交单信息(POST /quotation/order/query/dealList)<br />查询最优买卖(POST /quotation/order/query/marketBestPrice) |
| 交易         | 下单(POST /order/cex/user/order/create)<br />撤单(POST /order/cex/user/order/cancel) |
| 个人数据查询 | 查询指定账户余额(POST /user/cex/user/asset/query/currencyAsset)<br/>查询业务委托订单(POST /user/cex/user/order/query/orderList)<br/>查询订单明细(POST /user/cex/user/mm/query/orderDetail)<br/>查询成交单信息(POST /user/cex/user/order/query/dealOrderList)<br/>查询成交明细(POST /user/cex/user/order/query/dealDetail) |



# 行情

## 24小时行情接口

获取所有交易对的24小时成交量和价格。

**HTTP调用**

> POST /quotation/quotation/query/symbolQuotationGroupBy

**请求参数**

无

**请求示例**

```json
{
    "lang" : "zh-CN"
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/symbolQuotationGroupBy](https://api.trade.hkvax.com/v1/quotation/quotation/query/symbolQuotationGroupBy)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\"}"

**响应参数**

| 字段                | 字段说明      | 类型      | 是否必须 | 备注           |
| ------------------- | ------------- | --------- | -------- | -------------- |
| group               | 组名          | string    | true     | &nbsp;         |
| list                | 行情信息      | jsonArray | true     | &nbsp;         |
| list.symbol         | 交易对        | string    | true     | &nbsp;         |
| list.max            | k线最高       | string    | true     | &nbsp;         |
| list.min            | k线最低       | string    | true     | &nbsp;         |
| list.lastPrice      | 收盘价        | string    | true     | &nbsp;         |
| list.quantity       | 24H成交量     | string    | true     | &nbsp;         |
| list.amount         | 24H成交额     | string    | true     | &nbsp;         |
| list.lastUsdPrice   | 收盘USD折合价 | string    | true     | &nbsp;         |
| list.waveChange     | 涨跌比例      | string    | true     | &nbsp;         |
| list.wavePrice      | 涨跌值        | string    | true     | &nbsp;         |
| list.wavePercent    | 涨跌百分比    | string    | true     | &nbsp;         |
| list.direction      | 涨跌          | int       | true     | 1涨、0平、-1跌 |
| list.priceDirection | 当前价格涨跌  | int       | true     | 1涨、0平、-1跌 |
| list.maxUsdPrice    | 最高USD折合价 | string    | true     | &nbsp;         |
| list.minUsdPrice    | 最低USD折合价 | string    | true     | &nbsp;         |
| list.steps          | 可选小数位    | jsonArray | true     | &nbsp;         |
| steps.code          | 小数码        | string    | true     | &nbsp;         |
| steps.value         | 小数值        | string    | true     | &nbsp;         |
| steps.decimalValue  | 小数位数      | int       | true     | &nbsp;         |

**响应示例**

```json
{
	"code": "000000",
	"msg": "success",
	"data": [{
		"group": "BTC",
		"list": [{
		    "amount" : "0",
			"areaUsdPrice": "1.00000000",
			"direction": 0,
			"lastPrice": "1.5600",
			"lastUsdPrice": "1.56000000",
			"max": "1.5600",
			"maxUsdPrice": "1.56000000",
			"min": "1.5600",
			"minUsdPrice": "1.56000000",
			"quantity": "0",
			"steps": [{
				"code": "00001",
				"decimalValue": 4,
				"value": "0.0001"
			}, {
				"code": "001",
				"decimalValue": 2,
				"value": "0.01"
			}, {
				"code": "1",
				"decimalValue": 0,
				"value": "1.0"
			}],
			"symbol": "BTC/USD",
			"wavePercent": "0.00",
			"wavePrice": "0.0000"
		}]
	}]
}
```

## 深度数据接口

获取指定交易对的深度数据。

**HTTP调用**

> POST /quotation/quotation/query/depthData

**请求参数**

| 字段   | 字段说明 | 类型   | 是否必须 | 备注   |
| ------ | -------- | ------ | -------- | ------ |
| symbol | 交易对   | string | true     | &nbsp; |

**请求示例**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD"
    }
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/depthData](https://api.trade.hkvax.com/v1/quotation/quotation/query/depthData)" -H "accept: \*/*" -H "SIGNATURE:   your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\"}}"

**响应参数**

| 字段          | 字段说明      | 类型      | 是否必须 | 备注           |
| ------------- | ------------- | --------- | -------- | -------------- |
| bidsMaxPrice  | 最大买单价格  | string    | true     | &nbsp;         |
| asksMinPrice  | 最大卖单价格  | string    | true     | &nbsp;         |
| depthPercent  | 深度百分比    | string    | true     | &nbsp;         |
| xMinPrice     | x轴最小价格   | string    | true     | &nbsp;         |
| xMaxPrice     | x轴最大价格   | string    | true     | &nbsp;         |
| yQuantity     | y轴数量最大值 | string    | true     | &nbsp;         |
| symbol        | 交易对        | string    | true     | &nbsp;         |
| bids          | 买方深度数据  | jsonArray | true     | &nbsp;         |
| bids.price    | 价格          | string    | true     | &nbsp;         |
| bids.quantity | 数量          | string    | true     | &nbsp;         |
| asks          | 卖方深度数据  | jsonArray | true     | 结构和bids一致 |

**响应示例**

```json
{
	"code": "000000",
	"data": {
		"asksMinPrice": "0.70560000",
		"bidsMaxPrice": "0.23000000",
		"depthPercent": "206.79",
		"symbol": "BTC/USD",
		"xMaxPrice": "0.49119000",
		"xMinPrice": "0.44441000",
		"yQuantity": "0.0000",
		"bids": [{
			"price": "0.46780000",
			"quantity": "0.0000"
		}, {
			"price": "0.46733220",
			"quantity": "0.0000"
		}],
		"asks": [{
			"price": "0.46826780",
			"quantity": "0.0000"
		}, {
			"price": "0.46873560",
			"quantity": "0.0000"
		}]
	},
	"msg": "success"
}
```

## K线数据

获取指定交易对的历史K线数据。

**HTTP调用**

> POST /quotation/kline/get/quotationHistory

**请求参数**

| 字段      | 字段说明     | 类型   | 是否必须 | 备注                                                         |
| --------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| symbol    | 交易对       | string | true     | &nbsp;                                                       |
| startTime | 开始时间     | long   | true     | unix时间戳                                                   |
| endTime   | 结束时间     | long   | true     | unix时间戳                                                   |
| range     | 数据时间范围 | int    | true     | 对应的数值<br>1MIN(60 * 1000)<br>5MIN(5 * 60 * 1000)<br>15MIN(15 * 60 * 1000)<br>30MIN(30 * 60 * 1000)<br>1HOUR(60 * 60 * 1000)<br>2HOUR(2 * 60 * 60 * 1000)<br>4HOUR(4 * 60 * 60 * 1000)<br>6HOUR(6 * 60 * 60 * 1000)<br>12HOUR(12 * 60 * 60 * 1000)<br>1DAY(24 * 60 * 60 * 1000)<br>1WEEK(7 * 24 * 60 * 60 * 1000) |

**请求示例**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD",
        "startTime" : 1569405600000,
        "endTime" : 1569407400000,
        "range" : 300000
    }
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/kline/get/quotationHistory](https://api.trade.hkvax.com/v1/quotation/kline/get/quotationHistory)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\", \\"startTime\\" : 1569405600000, \\"endTime\\" : 1569407400000, \\"range\\" : 300000}}"

**响应参数**

data下是个字段为quotationHistory的jsonArray,内部详细字段如下:

| 字段     | 字段说明 | 类型   | 是否必须 | 备注       |
| -------- | -------- | ------ | -------- | ---------- |
| first    | 开盘价   | string | true     | &nbsp;     |
| last     | 收盘价   | string | true     | &nbsp;     |
| max      | 最高价   | string | true     | &nbsp;     |
| min      | 最低价   | string | true     | &nbsp;     |
| quantity | 成交量   | string | true     | &nbsp;     |
| time     | 时间     | long   | true     | unix时间戳 |

**响应示例**

```json
{
	"code": "000000",
	"msg": "success",
	"data": {
		"quotationHistory": [{
			"first": "0.23000000",
			"last": "0.23000000",
			"max": "0.23000000",
			"min": "0.23000000",
			"quantity": "0.000000"
			"time": 1569405600000
		}, {
			"first": "0.23000000",
			"last": "0.23000000",
			"max": "0.23000000",
			"min": "0.23000000",
			"quantity": "0.000000",
			"time": 1569405900000
		}]
	}
}
```

## 最新成交记录

获取指定交易对的最新一条成交数据。

**HTTP调用**

> POST /quotation/quotation/query/latestTradeQuotation

**请求参数**

| 字段   | 字段说明 | 类型   | 是否必须 | 备注   |
| ------ | -------- | ------ | -------- | ------ |
| symbol | 交易对   | string | true     | &nbsp; |

**请求示例**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "BTC/USD"
    }
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/quotation/query/latestTradeQuotation](https://api.trade.hkvax.com/v1/quotation/quotation/query/latestTradeQuotation)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\"}}"

**响应参数**

| 字段     | 字段说明 | 类型   | 是否必须 | 备注              |
| -------- | -------- | ------ | -------- | ----------------- |
| dealNo   | 成交单号 | long   | true     | &nbsp;            |
| symbol   | 交易对   | string | true     | &nbsp;            |
| price    | 成交价格 | string | true     | &nbsp;            |
| action   | 买卖     | string | true     | 买:BUY<br>卖:SELL |
| quantity | 成交量   | string | true     | &nbsp;            |
| utcDeal  | 时间     | long   | true     | unix时间戳        |

**响应示例**

```json
{
	"code": "000000",
	"msg": "success",
	"data": {
		"action": "SELL",
		"dealNo": 1645640033529819137,
		"price": "0.230000000000000000000000000000",
		"quantity": "1000.000000000000000000000000000000",
		"symbol": "BTC/USD",
		"utcDeal": 1569404634172
	}
}
```

## 订单簿数据

获取指定交易对当前的订单簿数据。

**HTTP调用**

> POST /quotation/order/query/orderBook

**请求参数**

| 字段      | 字段说明 | 类型   | 是否必须 | 备注                                                         |
| --------- | -------- | ------ | -------- | ------------------------------------------------------------ |
| symbol    | 交易对   | string | true     | &nbsp;                                                       |
| depthStep | 深度值   | string | true     | 对应值<br>深度十位:10<br>整数位:1<br>一位小数:01<br>二位小数:001<br>三位小数:0001<br>以此类推 |

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/quotation/order/query/orderBook" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\", \\"depthStep\\":\\"0000001\\"}}"

**响应参数**

| 字段                   | 字段说明       | 类型      | 是否必须 | 备注                                         |
| ---------------------- | -------------- | --------- | -------- | -------------------------------------------- |
| symbol                 | 交易对         | string    | true     | &nbsp;                                       |
| limitSize              | 订单簿行数     | int       | true     | &nbsp;                                       |
| step                   | 返回的深度     | string    | true     | &nbsp;                                       |
| askQuantity            | 卖方最大挂单量 | string    | true     | 订单簿在当前聚合深度下，卖单挂单量最大的数值 |
| bidQuantity            | 买方最大挂单量 | string    | true     | 订单簿中当前聚合深度下，买单挂单量最大的数值 |
| askList                | 卖单           | jsonArray | true     | &nbsp;                                       |
| askList.amount         | 价格           | string    | true     | &nbsp;                                       |
| askList.quantity       | 数量           | string    | true     | &nbsp;                                       |
| askList.limitPrice     | 限价           | string    | true     | &nbsp;                                       |
| askList.amountAccuracy | 精确金额       | string    | true     | &nbsp;                                       |
| bidList                | 买单           | jsonArray | true     | 结构与askList一致                            |


**响应示例**

```json
{
	"code": "000000",
	"data": {
		"askList": [{
			"amount": "0.549000",
			"amountAccuracy": "0.549000",
			"limitPrice": "0.001830",
			"quantity": "300.0000"
		}],
		"askQuantity": "300.0000",
		"bidList": [{
			"amount": "0.182000",
			"amountAccuracy": "0.182000",
			"limitPrice": "0.001820",
			"quantity": "100.0000"
		}],
		"bidQuantity": "100.0000",
		"latestDeal": {
			"direction": 0,
			"price": "0.001813",
			"usdPrice": "0.00181300"
		},
		"limitSize": 15,
		"step": "0000001",
		"symbol": "BTC/USD"
	},
	"msg": "success",
	"success": true,
	"traceId": "6c5be4dde02e4a8f"
}
```

## 查询最新的成交单信息

获取指定交易对的最新12条成交数据。

**HTTP调用**

> POST /quotation/order/query/dealList

**请求参数**

| 字段   | 字段说明 | 类型   | 是否必须 | 备注   |
| ------ | -------- | ------ | -------- | ------ |
| symbol | 交易对   | string | true     | &nbsp; |

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/quotation/order/query/dealList](https://api.trade.hkvax.com/v1/quotation/order/query/dealList)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\"}}"

**响应参数**

| 字段     | 字段说明 | 类型   | 是否必须 | 备注           |
| -------- | -------- | ------ | -------- | -------------- |
| action   | 买卖     | string | true     | 买:BUY 卖:SELL |
| dealNo   | 成交单号 | long   | true     | &nbsp;         |
| price    | 成交价格 | string | true     | &nbsp;         |
| quantity | 数量     | string | true     | &nbsp;         |
| symbol   | 交易对   | string | true     | &nbsp;         |
| utcDeal  | 成交时间 | long   | true     | unix时间戳     |

**响应示例**

```json
{
	"code": "000000",
	"data": [{
		"action": "BUY",
		"dealNo": 1647517554993250305,
		"price": "0.001813",
		"quantity": "200.0000",
		"symbol": "BTC/USD",
		"utcDeal": 1571195178105
	}, {
		"action": "SELL",
		"dealNo": 1647517531935064065,
		"price": "0.001812",
		"quantity": "100.0000",
		"symbol": "BTC/USD",
		"utcDeal": 1571195156310
	}],
	"msg": "success",
	"success": true,
	"traceId": "b701341fe31412ad"
}
```

## 查询最优的买卖

获取指定交易对的最优买价和最优卖价。

**HTTP调用**

> POST /quotation/order/query/marketBestPrice

**请求参数**

| 字段   | 字段说明 | 类型   | 是否必须 | 备注              |
| ------ | -------- | ------ | -------- | ----------------- |
| symbol | 交易对   | string | true     | &nbsp;            |
| action | 买卖类型 | string | true     | 买:BUY<br>卖:SELL |

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/quotation/order/query/marketBestPrice" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\",\\"action\\":\\"BUY\\"}}"

**响应参数**

| 字段      | 字段说明 | 类型   | 是否必须 | 备注           |
| --------- | -------- | ------ | -------- | -------------- |
| action    | 买卖     | string | true     | 买:BUY 卖:SELL |
| bestPrice | 成交价格 | string | true     | &nbsp;         |
| symbol    | 交易对   | string | true     | &nbsp;         |

**响应示例**

```json
{
	"code": "000000",
	"data": {
		"action": "BUY",
		"bestPrice": "0.046912",
		"symbol": "BTC/USD"
	}
}
```



# 交易

## 下单

发送一个新订单进行撮合。

**频率限制**

每2秒钟最多下单1次

**HTTP调用**

> POST /order/cex/user/order/create

**请求参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
symbol | 交易对 | string | true | &nbsp;
action | 买卖 | string | true | 买:BUY<br>卖:SELL
orderType | 订单类型 | string | true | 限价单:LMT<br>市价单:MKT
limitPrice | 下单价格 | decimal | false | 当为市价单时可不传
quantity | 下单数量 | decimal | false | 市价买入时可不传
amount | 金额 | decimal | false | 市价买入需要传入该字段

**请求示例**

> 市价买入示例:下面的amount表示冻结的USD数量为10000

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "BUY",
        "orderType" : "MKT",
        "amount" : 10000
    }
}
```

> 市价卖出:下面的quantity表示要冻结的LTC数量为10000个

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "SELL",
        "orderType" : "MKT",
        "quantity" : 10000
    }
}
```

> 限价买入示例:冻结的USD数量为(0.2 * 10000 = 2000)

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "BUY",
        "orderType" : "LMT",
        "limitPrice" : 0.2,
        "quantity" : 10000
    }
}
```

> 限价卖出:下面的quantity表示要冻结的LTC数量为10000个

```json
{
    "lang" : "zh-CN",
    "data" : {
        "symbol" : "LTC/USD",
        "action" : "SELL",
        "orderType" : "LMT",
        "limitPrice" : 0.3,
        "quantity" : 10000
    }
}
```

**响应参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单号 | long | true | 创建订单成功返回

**成功响应示例**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "orderNo" : 123
    }
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/order/cex/user/order/create](https://api.trade.hkvax.com/v1/order/cex/user/order/create)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"symbol\\": \\"BTC/USD\\", \\"action\\": \\"BUY\\", \\"orderType\\":\\"MKT\\", \\"amount\\": 10000}}"

## 撤单

发送一个撤单请求。

**HTTP调用**

> POST /order/cex/user/order/cancel

**请求参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单编号 | long | true | &nbsp;

**请求示例**
```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123
    }
}
```

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/order/cex/user/order/cancel" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"orderNo\\": 123}}"

**响应参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单编号 | long | true | &nbsp;

**成功响应示例**
```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "orderNo" : 123
    }
}
```



# 个人数据查询

## 查询指定账户余额

查询指定币种的可用余额。

**HTTP调用**

> POST /user/cex/user/asset/query/currencyAsset

**请求参数**

| 字段     | 字段说明 | 类型   | 是否必须 | 备注   |
| -------- | -------- | ------ | -------- | ------ |
| currency | 币种     | string | true     | &nbsp; |

**请求示例**

```json
{
    "lang" : "zh-CN",
    "data" : {
        "currency" : "USD"
    }
}
```

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/asset/query/currencyAsset" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"currency\\": \\"USD\\"}}"

**响应参数**

| 字段            | 字段说明     | 类型   | 是否必须 | 备注   |
| --------------- | ------------ | ------ | -------- | ------ |
| availableAmount | 当前可用余额 | string | true     | &nbsp; |

**响应示例**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "availableAmount" : "1372.624"
    }
}
```

## 查询业务委托订单(当前委托，历史委托)

基于搜索条件查询个人委托单列表。

**HTTP调用**

> POST /user/cex/user/order/query/orderList

**请求参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
gmtStart | 订单查询开始时间 | long | true | unix时间戳
gmtEnd | 订单查询结束时间 | long | true | unix时间戳
symbol | 交易对 | string | false | &nbsp;
orderAction | 买卖 | string | false | 买:BUY<br>卖:SELL
orderType | 订单类型 | string | true | 限价单:LMT<br>市价单:MKT<br>全部:ALL
orderStatus | 订单状态 | string | true | 全部成交:DEAL
未成交：MISMATCH
部分成交：PARTIAL
正在撤消：FIRSTTRIAL
部分成交已撤消：CANCEL_PARTIAL
未成交已撤消：CANCEL_MISMATCH 
pageRows | 页容 | int | true | &nbsp;
currPage | 当前页数 | int | true | 从1开始

**请求示例**
```json
{
    "lang" : "zh-CN",
    "data" : {
        "gmtStart" : 1569291600000,
        "gmtEnd" : 1569377997876,
        "orderStatus" : 0,
        "pageRows" : 10,
        "currPage" : 1
    }
}
```

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/order/query/orderList" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"gmtStart\\": 1569291600000, \\"gmtEnd\\": 1569377997876, \\"orderStatus\\": 0,\\"pageRows\\": 10,\\"currPage\\": 1}}"

**响应参数（分页响应）**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单编号 | long | true | &nbsp;
symbol | 交易对 | string | true | &nbsp;
action | 买卖 | string | true | 买:BUY<br>卖:SELL
quantity | 数量 | string | false | 买入市价<br>为0或空
quantityDeal | 成交量 | string | false | &nbsp;
quantityRemain | 订单剩余数量 | string | false | &nbsp;
fee | 手续费 | string | false | string类型的数字
feeCurrency | 手续费币种 | string | false | &nbsp;
orderType | 订单类型 | string | false | 限价单:LMT<br>市价单:MKT
priceAverage | 成交均价 | string | false | &nbsp;
priceLimit | 限价单限价 | string | false | &nbsp;
status | 订单状态 | string | true | 全部成交:DEAL<br/>未成交：MISMATCH<br/>部份成交：PARTIAL<br/>正在撤消：FIRSTTRIAL<br/>部份成交已撤消：CANCEL_PARTIAL<br/>未成交已撤消：CANCEL_MISMATCH 
gmtCreate | 创建时间 | string | true | unix时间戳
amount | 订单金额 | string | false | &nbsp;
amountRemain | 订单剩余金额 | string | false | &nbsp;

**响应示例**
```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "pagination" : {
            "pageSize" : 22,
            "currPage" : 1,
            "totalRows" : 212,
            "pageRows" : 10
        },
        "list" : [
            {
                "orderNo" : 123,
                "symbol" : "BTC/USD",
                "action" : "BUY",
                "quantity" : "0",
                "quantityDeal" : "10",
                "quantityRemain" : "30",
                "fee" : "3.3",
                "feeCurrency" : "BTC",
                "orderType" : "MKT",
                "priceAverage" : "300",
                "status" : "DEAL",
                "gmtCreate" : "1569291600000",
                "amount" : "0",
                "amountRemain" : "0"
            }
        ]
    }
}
```

## 查询订单明细

查询指定订单的状态和明细。

**HTTP调用**

> POST /user/cex/user/mm/query/orderDetail

**请求参数**

要查询的委托单号列表，委托单号之间用英文逗号分隔开

**请求示例**

```json
{
    "lang" : "zh-CN",
    "data" : [
        123,
        456,
        789
    ]
}
```

**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/user/cex/user/mm/query/orderDetail](https://api.trade.hkvax.com/v1/user/cex/user/mm/query/orderDetail)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":[123, 456, 789]}"

**响应参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单编号 | long | true | &nbsp;
symbol | 交易对 | string | true | &nbsp;
action | 买卖 | int | true | 买:2<br>卖:1
orderType | 订单类型 | int | true | 限价单:1<br>市价单:2
priceLimit | 价格（仅有限价单） | decimal | false | &nbsp;
quantity | 数量 | decimal | false | 买入市价<br>为0或空
quantityRemain | 剩余数量 | decimal | false | &nbsp;
status | 订单状态 | int | true | 正常:0<br>已撤销: 1
gmtCreate | 创建时间 | long | true | unix时间戳

**响应示例**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : [
        {
            "orderNo" : 123,
            "symbol" : "BTC/USD",
            "action" : 2,
            "quantity" : 0,
            "quantityRemain" : 30,
            "orderType" : 2,
            "status" : 0,
            "gmtCreate" : 1569291600000,
            "priceLimit" : 0
        }
    ]
}
```

## 查询成交单信息

基于搜索条件查询个人历史成交列表。

**HTTP调用**

> POST /user/cex/user/order/query/dealOrderList

**请求参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
gmtStart | 成交单查询开始时间 | long | false | unix时间戳
gmtEnd | 成交单查询结束时间 | long | false | unix时间戳
symbol | 交易对 | string | false | &nbsp;
orderAction | 买卖 | string | false | 买:BUY 卖:SELL 为空表示所有
orderType | 订单类型 | string | false | 限价单:LMT 市价单:MKT 为空表示所有
pageRows | 页容 | int | true | &nbsp;
currPage | 当前页数 | int | true | 从1开始

**请求示例**
```json
{
    "lang" : "zh-CN",
    "data" : {
        "gmtStart" : 1569291600000,
        "pageRows" : 10,
        "currPage" : 1
    }
}
```

**curl调用示例**

> curl -X POST "https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealOrderList" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"gmtStart\\": 1569291600000, \\"pageRows\\": 10,\\"currPage\\": 1}}"

**响应参数（分页响应)**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
symbol | 交易对 | string | true | &nbsp;
action | 买卖 | string | true | 买:BUY<br>卖:SELL
price | 成交价格 | decimal | true | &nbsp;
quantity | 成交量 | decimal | true | &nbsp;
amount | 成交额 | decimal | true | &nbsp;
utcDeal | 成交时间 | long | true | unix时间戳
dealNo | 成交编号 | long | true | &nbsp;
orderType | 订单类型 | string | true | 限价单:LMT<br>市价单:MKT
fee | 手续费 | decimal | true | &nbsp;
feeCurrency | 手续费币种 | string | false | &nbsp;

**响应示例**
```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : {
        "pagination" : {
            "pageSize" : 22,
            "currPage" : 1,
            "totalRows" : 212,
            "pageRows" : 10
        },
        "list" : [
            {
                "symbol" : "BTC/USD",
                "action" : "BUY",
                "price" : 319.22,
                "quantity" : 2.54,
                "amount" : 12.3,
                "utcDeal" : 1569291600000,
                "dealNo" : 1232,
                "fee" : "3.3",
                "feeCurrency" : "BTC",
                "orderType" : "MKT"
            }
        ]
    }
}
```

## 查询成交明细

查询指定订单号的成交明细。

**HTTP调用**

> POST /user/cex/user/order/query/dealDetail

**请求参数**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
orderNo | 订单编号 | long | true | &nbsp;

**请求示例**
```json
{
    "lang" : "zh-CN",
    "data" : {
        "orderNo" : 123
    }
}
```
**curl调用示例**

> curl -X POST "[https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealDetail](https://api.trade.hkvax.com/v1/user/cex/user/order/query/dealDetail)" -H "accept: \*/*" -H "SIGNATURE: your-signature-info" -H "APIKEY: your-public-key" -H "DEVICEID: 123" -H "DEVICESOURCE: web" -H "Lang: zh-CN" -H "Client: market002" -H "Content-Type: application/json" -d "{ \\"lang\\": \\"zh-CN\\", \\"data\\":{\\"orderNo\\": 123}}"

**响应参数(数组)**

字段 | 字段说明 | 类型 | 是否必须 | 备注
---|---|---|---|---
dealTime | 成交时间 | string | true | unix时间戳
dealPrice | 成交价格 | string | true | &nbsp;
dealQuantity | 成交数量 | string | true | &nbsp;

**响应示例**

```json
{
    "code" : "000000",
    "msg" : "success",
    "data" : [
        {
            "dealTime" : "1569291600000",
            "dealPrice" : "3.3",
            "dealQuantity" : "300"
        }
    ]
}
```
