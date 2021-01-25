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

| 字段名    | 字段说明 | 字段类型 | 备注               |
| --------- | -------- | -------- | ------------------ |
| pageSize  | 总共页数 | int      | &nbsp;             |
| currPage  | 当前页   | int      | 从1开始            |
| totalRows | 总共行数 | int      | &nbsp;             |
| pageRows  | 当前行数 | int      | 分页大小(最大1000) |

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