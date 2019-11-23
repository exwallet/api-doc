# 商户 OPEN API 文档

|版本号|作者|日期|备注|
|---|---|---|---|
|v1.0 | kidd | 2019.7.12|     |
|v1.1 | kidd | 2019.9.10| 优化接口字段   |


[TOC]


## OverView


#### 冷热钱包关系图

![冷热钱包关系图](https://s2.ax1x.com/2019/11/22/MoYuAs.png)

#### 充值/提币帐单说明

<b>status状态说明</b>

``` 
	 0   // 已创建
	 1   // 等待提交
	 2   // 提交成功 (提交钱包节点成功)
	 3   // 提交失败 (提交钱包节点失败)
	 301 // 提交失败待确认
	 4   // 广播成功 
	 5   // 等待确认 (等待区块确认高度)
	 10  // 已确认  (入帐标志)
	 20  // 已确认2 (针对充值的区块高度达到确认数1基础上, 再达确认数2, 如果省略10直接推送20,直接入帐)
	 -1  // 已回滚
    
```
<b>充值/提币帐单通知字段</b>
``` 
symbol
contractId
contractAddress
status
fromAddress
toAddress
amount
isMemo
memo
usid
txid
fee
confirm
blockHash
blockHeight
fork

```

<b>通知充值/提币确认流程</b>
  - status = 2 更新txid,可自行查询链上状态. (txid如果上链失败会重新广播,status=4时需要再更新txid)
  - status = 3 流程暂停, 等待人工处理 
  - status = 4  更新txid, blockHash, blockHeight, confirm
  - status = 5, confirm=n ,  更新confirm
  - status = 5, confirm=n+?  更新confirm, 此过程不断推送, 确认数不断增加, 交易所可更新确认数
  - status = 10  已达入帐高度, 确认入帐.
  - status = 20  已达可提币的高度 (大于入帐高度) (预留,暂时不推) 
  
  <b>*通知状态可能略过4, 5状态,通知10,可直接入帐</b>
  <b>*如果在接收到状态10前通知-1, 这笔帐单忽略入帐</b>
  <b>*如果接收到状态-1后, 又接收10, 正常入帐</b>   




#### 全局说明

1. 系统支持最大精度`14`位
2. 系度支持最大金额`1000000000000` 



#### API说明
商户申请开放API功能后, 得到`appKey`和`appSecret`

- 请求说明:

    请求的参数通过`GET`方式提交, querystring中带上需要的参数, 并加入当前时间戳`timestamp` (精确到秒).
    最后,在HTTP请求头部加上`appKey`与`sign`签名


- 头部说明: 

    `appKey` : 提供的appKey.
    `sign`   : 通过`HmacSHA256`,用`appSecret`与各URI参数(经Ascii升序排序后)生成的签名.
    例如: 
    `HmacSha256("a=123&b=456")` 



## API 列表

    回调的头部带上`sign`签名, 商户接收使用`appSecret`对请求的参数做签名, 将结果与`sign`校验是否一致

拼接URL `GetPayOutAddress[方法名]`


<b>返回数据</b>

```json
{
  "code": 1000,
  "message": "error message",
  "data": []
}
```


<b>响应Code</b>

|code |  说明 | 
|---|---|
|1000  | 成功    |
|1001  | 服务出错    |
|1002  | 请求出错    |
|1003  | 非法访问    |


#### VerifyAddress 地址校验

请求参数: 


|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| symbol            | string | * | 主链名称 |
| address           | string | * | 地址    |


  

#### GetBaseCoins 取得基础币种

请求参数 : 没有

```
    Id              int64    //      
    Symbol          string   // 币种名称
    ContractId      string   // 合约哈希
    IsContract      int64    // 是否合约 1是,0不是
    ContractAddress string   // 合约地址
    Token           string   // 合约名称
    Name            string   // 长名称
    Confirm         int64    // 确认次数
    Confirm2        int64    // 提币确认次数
    Decimals        int64    // 货币小数精确度
    FeeFixed        string   // 单笔提现固定费率
    FeeRate         string   // 单笔提现浮动费率
    MinPayIn        string   // 单笔最少充值金额
    MinPayOut       string   // 单次允许最小提币 0不限制
    MaxPayOut       string   // 单次允许最大提币 0不限制
    IsPayIn         int64    // 是否开启充币 (1是,0否)
    IsPayOut        int64    // 是否开启提币 (1是,0否)
    Status          int64    // 是否有效 0不可用, 1可用
    EnableMemo      int64    // 是否启用memo 1启用,0不启用
    BalanceMode     uint64   // 余额模型: 0地址, 1帐户(通过alias作为地址作用)
    SupportMemo     uint64   // 链是否支持memo, 0: false, 1: true
    OnlyContract    uint64   // 只支持合约, 0: false, 1: true
```

#### GetUserCoins 取得开通币种

请求参数 : 没有

```
    BaseCoinId      int64   // 币种ID
    Symbol          string  // 币种名称
    ContractId      string  // 合约哈希
    ContractAddress string  // 合约地址
    IsContract      int64   // 是否合约 0不是, 1是
    IsPayIn         int64   // 是否开启充币 (1是,0否)
    IsPayOut        int64   // 是否开启提币 (1是,0否)
    CreateTime      int64   // 创建时间
```


#### ApplyAddress 生成币种地址

请求参数 

|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| symbol            | string | * | 主链名称 |
| count             | string | * | 请求生成地址数数 |



```
    Symbol     string   // 币种名称
    Address    string   // 钱包地址
    IsMemo     int64    // 1是, 0否
    Memo       int64    // MEMO信息
    CreateTime int64    // 创建时间
```

#### GetBill 获取帐单
请求参数 

|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| usid            | string | * | 唯一业务流水号,不超过128位 |

返回单个实体
     
```
	BaseCoinId      int64  `json:"baseCoinId"`      // 币种ID
	Symbol          string `json:"symbol"`          // 主链币
	IsContract      int64  `json:"isContract"`      // 是否哈希 0不是,1是
	ContractAddress string `json:"contractAddress"` // 合约地址
	BillType        int64  `json:"billType"`        // 业务类型
	Status          int64  `json:"status"`          // 帐单状态 看枚举
	From            string `json:"from"`            // 从哪充进来
	To              string `json:"to"`              // 到哪个地址
	Amount          string `json:"amount"`          // 变更金额
	Fee             string `json:"fee"`             // 记帐手续费
	IsMemo          int64  `json:"isMemo"`          // 1是,0否
	Memo            string `json:"memo"`            // Memo
	Confirm         int64  `json:"confirm"`         // 确认数
	BlockHeight     int64  `json:"blockHeight"`     // 区块高度
	BlockHash       string `json:"blockHash"`       // 区块哈希
	Usid            string `json:"usid"`            // 商户业务ID
	Txid            string `json:"txid"`            // 链上交易单号
	CreateTime      int64  `json:"createTime"`      // 记帐时间
	UpdateTime      int64  `json:"updateTime"`      // 更新时间
	NotifyStatus    int64  `json:"notifyStatus"`    // 通知状态 具体看status状态说明
	Fork            int64  `json:"fork"`            // 分叉标志
	
``` 


#### GetBillList 获取帐单<旧>

请求参数 

|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| usid            | string | * | 唯一业务流水号,不超过128位 |

返回列表
   
```
	BaseCoinId      int64  `json:"baseCoinId"`      // 币种ID
	Symbol          string `json:"symbol"`          // 主链币
	IsContract      int64  `json:"isContract"`      // 是否哈希 0不是,1是
	ContractAddress string `json:"contractAddress"` // 合约地址
	BillType        int64  `json:"billType"`        // 业务类型
	Status          int64  `json:"status"`          // 帐单状态 看枚举
	From            string `json:"from"`            // 从哪充进来
	To              string `json:"to"`              // 到哪个地址
	Amount          string `json:"amount"`          // 变更金额
	Fee             string `json:"fee"`             // 记帐手续费
	IsMemo          int64  `json:"isMemo"`          // 1是,0否
	Memo            string `json:"memo"`            // Memo
	Confirm         int64  `json:"confirm"`         // 确认数
	BlockHeight     int64  `json:"blockHeight"`     // 区块高度
	BlockHash       string `json:"blockHash"`       // 区块哈希
	Usid            string `json:"usid"`            // 商户业务ID
	Txid            string `json:"txid"`            // 链上交易单号
	CreateTime      int64  `json:"createTime"`      // 记帐时间
	UpdateTime      int64  `json:"updateTime"`      // 更新时间
	NotifyStatus    int64  `json:"notifyStatus"`    // 通知状态 具体看status状态说明
	Fork            int64  `json:"fork"`            // 分叉标志
	
``` 




#### GetFunds 获取资金

请求参数 :没有

```
    BaseCoinId      int64   // 币种ID
	Symbol          string  // 主链币
	ContractId      string  // 合约哈希
	ContractAddress string  // 合约地址
	IsContract      int64   // 是否合约(0否,1是)
	BalanceA        string  // 充值钱包余额
	BalanceB        string  // 提币钱包余额

```

#### ApplyTransaction 提币申请

请求参数 

|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| usid            | string | * | 唯一业务流水号,不超过128位 |
| symbol          | string | * | 主链名称 |
| contractId      | string | * | 合约ID   |   
| contractAddress | string | * | 合约地址,  contractAddress,contractId两个只需要填一个 |
| amount          | string | * | 金额     |
| to              | string | * | 提币地址 |
| memo            | string |   | 使用memo时选填     |
    

#### RePushBill 重新推送帐单

请求参数 

|参数名 | 类型 | 必选项 | 说明 | 
|---|---|---|---|
| usid            | string | * | 唯一业务流水号,不超过128位 |



## 商户回调

> 通知事件到交易所提供的URL , 走HTTP协议, 商户推送内容json经Base64 encode. 使用Api密钥作HmacSHA256生成sgin 将sign写入头部.
交易所接收到内容, 取data对应的值,  使用Api密钥作HmacSHA256生成sgin, 与Http头部Sgin对比一致性, 通过一致性后, 再对data作Base64Decode, 取得实际数据


交易所接收推送内容, 如
```
{
    "data": "eyJtZXRob2QiOiJQYXlJbiIsInRpbWVzdGFtcCI6MTU2Mjk0MTA2MTQ5MSwiZGF0YSI6W3sic3ltYm9sIjoiVFJYIiwiY29udHJhY3RJZCI..."
}
```

取`data`的值, 经`Base64`解码后, 内容如
```json
{
    "method": "PayIn",
    "timestamp": 1562941061491,
    "data": [{
        "symbol": "TRX",
        "contractId": "7hLlcYzn5DH6Fkdo8n11pGpBKrb2JMcdEzKfdUUM9cg=",
        "status": 10,
        "fromAddress": "TF84rDKSaVXpFyzMvP8SPPzEChQ4QAP75N",
        "toAddress": "TQro3rA6xPGoNiPBKNDupsbAHVKXVEUgH8",
        "amount": "1.0000000000000000",
        "usid": "MT_26969f0456e3c0f656a796b5819e9303f9dbd4ffd442c9056c4a76fd9fd5b0bc",
        "txid": "bfcf22d7dee815d658899ba1fe63058a2df5de629a2012502d128c758b238a11",
        "fee": "0.0000000000000000",
        "confirm": 13,
        "blockHash": "0000000000a60fea66794a40345a70e2e16fc0efb78b9f67e34e1de8bf100100",
        "blockHeight": 10883050,
        "fork": 0
    }]
}
``` 




