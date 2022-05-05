# referral-system-design
## 推荐流程
1. 玩家需要消费至少X美元，即可以生成邀请码。
2. 用户生成邀请码后，可以自行分享给其他用户。
3. 其他用户收到邀请码，登陆网站后可以输入邀请码，建立联系。
4. 邀请人根据被邀请人消费的数额进行分红，领取手续费。


### 1. 查询状态
**HTTP GET**: `{{URL}}/refer/status?player=0x063fEE1bF351b05C264d4234ea993A22A18D6081`
* 参数 player: 用户钱包地址

返回值：

通过API查询用户是否可以生成邀请码。不满足要求的用户。**[STATUS CODE 400]**
```javascript
{
    "canRefer": false, // 不可以生成邀请码
    "credit": 0        // 用户当前消费总额
}
```

满足要求的用户，则返回邀请码。**[STATUS CODE 200]**
```javascript
{
    "stats": [],
    "totalCredit": "0.0",
    "totalClaimed": "0.0",
    "credit": 579, // 用户当前消费总额
    "code": ""     // 邀请码
}
```

已有分红数据的用户，则返回下线人，下线人消费额，总分红数额，已领取的分红数额
```javascript
{
    "stats": [
        {
            "address": "0x1Ac17e74df2e06FBeFF81a86565d7bd528044bE9",
            "spent": "38" // 消费额
        },
        {
            "address": "0xd164A3a6dc30a8fd816AD28CB6F8F57Aac201473",
            "spent": "0" // 消费额
        },
        {
            "address": "0xA2F3254855977Be1f1A43DF8556CC9A04FF853f9",
            "spent": "0" // 消费额
        }
    ],
    "totalCredit": "0.33333", // 邀请人总分红数额
    "totalClaimed": "0.0",    // 邀请额已领取的分红数额
    "credit": 19550,
    "code": "79c26f87"
}

```

### 2. 接受邀请
**HTTP POST**: `{{URL}}/refer?player=0xf04E1F8FEF444AB33EbceBd5Ef32e17ff2ef4150&code=879e9bf6`
* 参数 player: 新用户的地址
* 参数 code: 邀请人的邀请码

返回值：

如果用户不是新用户，不能建立邀请关系 **[STATUS CODE 400]**
```javascript 
{
    "referrer": "0x0000000000000000000000000000000000000000", 
    "credit": 19, // 该用户已有消费记录，代表不是新用户
    "msg": "User is not new"
}
```

用户是新用户，则被成功邀请 **[STATUS CODE 200]**
```javascript
{
    "referrer": "0xD4f821695cfb105822Ec1e1F111C7f863E939BEf", // 邀请码主人的地址
    "credit": 0,
    "msg": "Refer successful"
}
```

邀请码无效 **[STATUS CODE 404]**
```javascript
{
    "referrer": "0x0000000000000000000000000000000000000000",
    "credit": 0,
    "msg": "Invalid code"
}
```

用户如已被邀请过，不能再次被邀请 **[STATUS CODE 500]**
```javascript
{
    "msg": "Returned error: execution reverted: EXISTS"
}
```

### 3. 邀请人领取分红
**HTTP POST**: {{URL}}/refer/reward/claim?player=0xD4f821695cfb105822Ec1e1F111C7f863E939BEf
* 参数 player: 邀请人的地址

返回值
* `amount`: 能领取的数额，等于端口1获得的 `totalCredit - totalClaimed`。
* `proof`: 梅克尔树证明，用于向合约发起提款申请

```
{
    "amount": "0.16667",
    "proof": [
        "0x964e015d9c75827f744fe75dfd5bef36be4a22ddff481d153ca0c3bf30306656"
    ]
}
```

前端要求
* 前端需要显示：1. 邀请人总共积累的分红。2. 已被领取的分红。此信息能从端口1查询获得（字段`totalCredit`, `totalClaimed`）。两者之差则为还未被领取的分红。
* 前端需要显示每一位被邀请人的消费，也由端口1获得（字段`stats`)。
* 前端给用户提供一个按钮，点击领取分红。**无需用户输入待领取的数字，点击一次则领取全部分红**
