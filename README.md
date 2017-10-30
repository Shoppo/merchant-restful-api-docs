# Shoppo Merchant API

Merchant API是Shoppo为商户、ERP的API对接提供的API，对一下功能提供支持：
- 获取、更新（TODO）商户信息；
- 获取、更新（TODO）商品信息；
- 获取、更新订单信息；
- 对接Shoppo物流（TODO）；

## 基本设置

在开始之前，请跟联系Shoppo的对接人员开通API服务。API服务开通后，您可以在[开发者页面](http://localhost:8080/merchant/developer/profile)获取到商户ID以及API Key。

您可以使用下面的命令行指令来测试API Key的有效性：

```bash
$ curl https://api.shoppo.com/api/merchant/health_check --header "merchantid: <商户ID>" --header "apikey: <API Key>"
```

有效的API Key将会收到如下response：

```
Go <商户名称>! Go Shoppo!
```

此外，请注意以下几点：
- 所有的数据均适用`UTF-8`编码；
- 请发送HTTPS请求，我们不保证对HTTP协议的支持；
- 每个请求的header都必须包括`merchantid`以及`apiKey`；

### 错误信息

所有成功的API请求都将会返回一个`statusCode = 200`的response。错误的请求将会以非200的代码返回，一般包含一个错误信息。比如：

```bash
$ curl https://api-sandbox.shoppo.com/api/merchant/health_check --header "merchantid: <merchant id>" --header "apikey: wrong-key" -i
HTTP/1.1 403 FORBIDDEN
Access-Control-Allow-Origin: *
Content-Type: application/json
Date: Mon, 30 Oct 2017 03:35:58 GMT
Server: nginx/1.12.1
Content-Length: 55
Connection: keep-alive

{
  "error": "Unauthorized request: invalid api key"
}
```

### 测试环境

在开发测试阶段，您可能不希望一些测试的请求影响到生产数据。此时请要求对接人员提供一个测试用的商户ID以及API Key，并且将所有的测试请求发送到https://api-sandbox.shoppo.com/。比如

```bash
$ curl https://api-sandbox.shoppo.com/api/merchant/health_check --header "merchantid: <商户ID>" --header "apikey: <API Key>"
```

## API接口列表

### 读取

#### `/api/merchant/profile/` [GET]
获取当前商户信息。

参数：无

返回参数：
- `id (string!)`: 商户ID
- `merchant_name (string!)`: 商户名
- `status (string!)`: `VALID`, `PENDING_APPROVE`, `SUSPEND`或者`INVALID`。其中只有`VALID`是在经营的商户状态。

参考Response：
```json
{
  "data": {
    "id": "gKXzYvzAm1QsZ",
    "merchant_name": "Shoppo Dev",
    "status": "VALID"
  }
}
```

#### `/api/merchant/orders/` [GET]
读取当前商户订单。

参数：
- `bookmark (string?)`: 分页是使用的起始标示，相当于offset。
- `limit (int?)`: 返回的order最大数量，默认20。必须为一个0到200之间的整数。

返回参数：TODO
- `total (int!)`: 总订单数；
- `bookmark (string!)`: 分页时，下一次请求需要传入的参数；
- `orders ([Order]!)`: 订单列表，请注意再遇到非法的`Order`时列表中可能包含`null`（见下面的例子）

参考request：

```bash
https://api-sandbox.shoppo.com/api/merchant/orders/?limit=2&bookmark=fa7GD1JmmdOI1
```

参考Response：
```json
{
  "data": {
    "bookmark": "jzyGD1JmmdOI1",
    "orders": [
      null,
      {
        "buyer_id": "k7okizN7",
        "buyer_order_number": "26978695384001255400",
        "id": "jzyGD1JmmdOI1",
        "order_items": [
          {
            "id": "3V1q1JWqvb2tdW",
            "is_refunded": false,
            "product_snapshot_identifier": "test-prod-ea52ac4c5bd011e79332f45c89a3e2f9",
            "product_snapshot_name": "A dog",
            "quantity": 1,
            "refund_reason": "",
            "shipping_provider": null,
            "shipping_refunded": true,
            "sku_snapshot_identifier": "test-sku-ea52e05e5bd011e7b06bf45c89a3e2f9",
            "sku_snapshot_price": 171.31,
            "sku_snapshot_shipping_price": 5.5,
            "status": "PAID",
            "tracking_number": null
          },
          {
            "id": "6qvyvJWyx1VS5l",
            "is_refunded": false,
            "product_snapshot_identifier": "test-prod-ea54d47a5bd011e797c6f45c89a3e2f9",
            "product_snapshot_name": "A cat",
            "quantity": 1,
            "refund_reason": "",
            "shipping_provider": null,
            "shipping_refunded": false,
            "sku_snapshot_identifier": "test-sku-ea54fd805bd011e7ae35f45c89a3e2f9",
            "sku_snapshot_price": 107.37,
            "sku_snapshot_shipping_price": 1.0,
            "status": "PAID",
            "tracking_number": null
          }
        ],
        "order_time": "Mon, 10 Jul 2017 08:11:39 GMT",
        "order_total": 285.18,
        "shipping_address": {
          "address1": "1 Folsom St",
          "address2": "",
          "city": "San Francisco",
          "country": "United States",
          "phone_number": "3356432453",
          "recipient_name": "Test User",
          "state": "CA",
          "zipcode": "94106"
        }
      }
    ],
    "total": 90
  }
}
```

#### `/api/merchant/order/<order ID>/` [GET]
读取单个商户订单信息。

参数：无

返回参数：一个`Order`。

参考request：

```bash
https://api-sandbox.shoppo.com/api/merchant/order/jzyGD1JmmdOI1/
```

参考Response：

```json
{
  "data": {
    "buyer_id": "0Vw2T5j7",
    "buyer_order_number": "23904323511825880199",
    "id": "pkyrw2qmmlXfn",
    "order_items": [
      {
        "id": "KvL2Lb12jnMTpw",
        "is_refunded": false,
        "product_snapshot_identifier": "test-prod-274ffb685a8511e78e2af45c89a3e2f9",
        "product_snapshot_name": "test prod name 274ffb685a8511e78e2af45c89a3e2f9",
        "quantity": 1,
        "refund_reason": "",
        "shipping_provider": null,
        "shipping_refunded": false,
        "sku_snapshot_identifier": "test-sku-27500cae5a8511e7812bf45c89a3e2f9",
        "sku_snapshot_price": 176.4,
        "sku_snapshot_shipping_price": 1.0,
        "status": "IN_FULFILLMENT",
        "tracking_number": null
      }
    ],
    "order_time": "Fri, 27 Oct 2017 05:43:20 GMT",
    "order_total": 177.4,
    "shipping_address": {
      "address1": "test",
      "address2": "est",
      "city": "test",
      "country": "United States",
      "phone_number": "3356432453",
      "recipient_name": "Test test",
      "state": "te",
      "zipcode": "53216"
    }
  }
}
```

#### `/api/merchant/products/` [GET]
读取当前商户商品。

参数：
- `bookmark (string?)`: 分页是使用的起始标示，相当于offset。
- `limit (int?)`: 返回的`Product`最大数量，默认20。必须为一个0到200之间的整数。

返回参数：TODO
- `total (int!)`: 总商品数；
- `bookmark (string!)`: 分页时，下一次请求需要传入的参数；
- `products ([Product]!)`: 订单列表，请注意再遇到非法的`Product`时列表中可能包含`null`。

参考request：

```bash
https://api-sandbox.shoppo.com/api/merchant/products/?limit=1
```


参考Response：

```json
{
  "data": {
    "bookmark": "4YzNjz4KZOue",
    "products": [
      {
        "brand": "test brand",
        "category_id": "501010",
        "chinese_name": "中文名",
        "cover_image_url": "https://cdn.shoppo.com/images/7c7066ea67bcdae2603ebc10dcc41a1a.JPEG",
        "cover_video_url": null,
        "description": "test prod description 5e5b952c5a5a11e7a145f45c89a3e2f9",
        "extra_image_urls": ["https://cdn.shoppo.com/images/1c7066fa67bcdae2603ebc10dcc41a1a.JPEG"],
        "features": [],
        "id": "4YzNjz4KZOue",
        "identifier": "test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9",
        "name": "test prod name 5e5b952c5a5a11e7a145f45c89a3e2f9",
        "rich_detail_sections": "Coming soon",
        "skus": [
          {
            "color": "red",
            "cover_image_url": "https://cdn.shoppo.com/images/34768a6a185ff5c76780dc99fb3140b8.JPEG",
            "enabled": true,
            "height": "8",
            "hs_code": "1234",
            "id": "pvmj64QXLDaux",
            "identifier": "test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9",
            "inventory": 14,
            "length": "4",
            "material": "metal",
            "msrp": 174.44,
            "price": 164.44,
            "product_id": "4YzNjz4KZOue",
            "product_identifier": "test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9",
            "shipping_price": 1.0,
            "shipping_time": "10-30 Days",
            "size": "S",
            "upc": "1237513718",
            "weight": "0.5",
            "width": "1"
          }
        ],
        "tags": [],
        "target_user_type": "ALL",
        "white_background_image_url": null
      }
    ],
    "total": 347
  }
}
```

#### `/api/merchant/product/<merchant product identifier>/` [GET]
通过商户商品ID来读取单个商户商品信息。

参数：无

返回参数：一个`Product`。

参考request：

```bash
https://api-sandbox.shoppo.com/api/merchant/product/test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9/
```

参考Response：

```json
{
  "data": {
    "brand": "test brand",
    "category_id": "501010",
    "chinese_name": "中文名",
    "cover_image_url": "https://cdn.shoppo.com/images/7c7066ea67bcdae2603ebc10dcc41a1a.JPEG",
    "cover_video_url": null,
    "description": "test prod description 5e5b952c5a5a11e7a145f45c89a3e2f9",
    "extra_image_urls": ["https://cdn.shoppo.com/images/1c7066fa67bcdae2603ebc10dcc41a1a.JPEG"],
    "features": [],
    "id": "4YzNjz4KZOue",
    "identifier": "test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9",
    "name": "test prod name 5e5b952c5a5a11e7a145f45c89a3e2f9",
    "rich_detail_sections": "Coming soon",
    "skus": [
      {
        "color": "red",
        "cover_image_url": "https://cdn.shoppo.com/images/34768a6a185ff5c76780dc99fb3140b8.JPEG",
        "enabled": true,
        "height": "8",
        "hs_code": "1234",
        "id": "pvmj64QXLDaux",
        "identifier": "test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9",
        "inventory": 14,
        "length": "4",
        "material": "metal",
        "msrp": 174.44,
        "price": 164.44,
        "product_id": "4YzNjz4KZOue",
        "product_identifier": "test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9",
        "shipping_price": 1.0,
        "shipping_time": "10-30 Days",
        "size": "S",
        "upc": "1237513718",
        "weight": "0.5",
        "width": "1"
      }
    ],
    "tags": [],
    "target_user_type": "ALL",
    "white_background_image_url": null
  }
}
```

#### `/api/merchant/sku/<merchant sku identifier>/` [GET]
通过商户SKU ID来读取单个SKU的信息。

参数：无

返回参数：一个`Sku`。

参考request：

```bash
https://api-sandbox.shoppo.com/api/merchant/sku/test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9/
```

参考Response：

```json
{
  "data": {
    "color": "red",
    "cover_image_url": "https://cdn.shoppo.com/images/34768a6a185ff5c76780dc99fb3140b8.JPEG",
    "enabled": true,
    "height": null,
    "hs_code": null,
    "id": "pvmj64QXLDaux",
    "identifier": "test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9",
    "inventory": 14,
    "length": null,
    "material": null,
    "msrp": 174.44,
    "price": 164.44,
    "product_id": "4YzNjz4KZOue",
    "product_identifier": "test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9",
    "shipping_price": 1.0,
    "shipping_time": "10-30 Days",
    "size": "S",
    "upc": "",
    "weight": null,
    "width": null
  }
}

```

### 修改

#### `/api/merchant/order_item/<order item ID>/` [POST, PUT]
修改单个商户订单信息。**注意，仅修改提供的参数**。

参数：
- `tracking_number (String)`: 物流面单号。请注意面单号仅能够在该订单还未发货（`status=SHIPPED`）之前修改，发货之后无法通过API修改面单号。此外，我们仅支持几家常用的物流提供商，如果您的物流单号无法被识别，请联系Shoppo的客服人员解决。
- `shipping_provider (String)`: 物流商名称（比如EMS, China Post），注意`shipping_provider`必须和`tracking_number`同时修改。

返回参数：一个`OrderItem`。

参考request：

```
https://api-sandbox.shoppo.com/api/merchant/order_item/KvL2Lb12jnMTpw/
```

参考payload:

```json
{
	"tracking_number": "31837953231",
	"shipping_provider": "China Post"
}
```

参考Response:

```json
{
  "data": {
    "id":"KvL2Lb12jnMTpw",
    "is_refunded": false,
    "product_snapshot_identifier": "test-prod-274ffb685a8511e78e2af45c89a3e2f9",
    "product_snapshot_name": "test prod name 274ffb685a8511e78e2af45c89a3e2f9",
    "quantity": 1,
    "refund_reason": "",
    "shipping_provider": "China Post",
    "shipping_refunded": false,
    "sku_snapshot_identifier": "test-sku-27500cae5a8511e7812bf45c89a3e2f9",
    "sku_snapshot_price": 176.4,
    "sku_snapshot_shipping_price": 1.0,
    "status": "IN_FULFILLMENT",
    "tracking_number": "31837953231"
  }
}
```

#### `/api/merchant/order_items/ship` [POST, PUT]
将多个`OrderItem`标记为已发货。

参数：
- `order_item_ids ([String!]!)`: 发货的`OrderItem`ID列表。注意，仅有`PAID`和`IN_FULFILLMENT`状态、并且已经有合法`tracking_number`的订单可以发货。

返回参数：已发货的`OrderItem`列表。

参考request：

```
https://api-sandbox.shoppo.com/api/merchant/order_items/ship/
```

参考payload:

```json
{
	"order_item_list": ["KvL2Lb12jnMTpw"]
}
```

参考Response:
```json
{
  "data": {
  	"order_items": [
      {
        "id":"KvL2Lb12jnMTpw",
        "is_refunded": false,
        "product_snapshot_identifier": "test-prod-274ffb685a8511e78e2af45c89a3e2f9",
        "product_snapshot_name": "test prod name 274ffb685a8511e78e2af45c89a3e2f9",
        "quantity": 1,
        "refund_reason": "",
        "shipping_provider": "China Post",
        "shipping_refunded": false,
        "sku_snapshot_identifier": "test-sku-27500cae5a8511e7812bf45c89a3e2f9",
        "sku_snapshot_price": 176.4,
        "sku_snapshot_shipping_price": 1.0,
        "status": "IN_FULFILLMENT",
        "tracking_number": "31837953231"
      }
    ]
  }
}
```

#### `/api/merchant/order_item/<order item ID>/refund` [POST, PUT]
将单个`OrderItem`退款。注意，refund并不会改变`OrderItem`的状态。一个已经退款的`OrderItem`不能被重复退款（除非第一次没有退运费，第二次退运费）。

参数：
- `refund_shipping (Boolean!)`: 是否退运费。注意，一般情况下，未发货的订单一定要求退运费。
- `refund_reason (String!)`：退款原因。注：用户可见，必须用英文。

返回参数：退款的order item。

参考request：

```
https://api-sandbox.shoppo.com/api/merchant/order_item/KvL2Lb12jnMTpw/refund
```

参考payload:

```json
{
	"refund_shipping": true,
    "refund_reason": "Out of stock"
}
```

参考Response:
```json
{
  "data": {
    "id":"KvL2Lb12jnMTpw",
    "is_refunded": true,
    "product_snapshot_identifier": "test-prod-274ffb685a8511e78e2af45c89a3e2f9",
    "product_snapshot_name": "test prod name 274ffb685a8511e78e2af45c89a3e2f9",
    "quantity": 1,
    "refund_reason": "",
    "shipping_provider": "China Post",
    "shipping_refunded": true,
    "sku_snapshot_identifier": "test-sku-27500cae5a8511e7812bf45c89a3e2f9",
    "sku_snapshot_price": 176.4,
    "sku_snapshot_shipping_price": 1.0,
    "status": "IN_FULFILLMENT",
    "tracking_number": "31837953231"
  }
}
```



## 类型定义

### Order

一个订单。

| Field name        | Type           | Description  |
| ------------- |-------------| -----|
| id      | String! | 订单ID |
| order_time      | DateTime!      |   下单时间 |
| shipping_address | ShippingAddress!      | 送货地址 |
| order_total | float! | 订单金额（美元） |
| buyer_id | String! | 买家ID |
| buyer_order_number | String! | 买家订单号（显示给买家的一个数字订单号） |
| order_items | List[OrderItem!]! | 订单商品列表 |


### OrderItem

一个订单商品。注意：所有订单中的商品、地址都是一个快照，不会由于商家修改实际商品而变化。所以相关变量名中都加入了`snapshot`关键字。

| Field name        | Type           | Description  |
| ------------- |-------------| -----|
| id      | String! | 订单商品ID（注：并不是商品ID） |
| quantity | Int! | 购买数量 |
| sku_snapshot_identifier | String! | 所购买Sku的商户自定义ID |
| sku_snapshot_price | float! | 所购买Sku的价格（美元） |
| sku_snapshot_shipping_price | float! | 所购买Sku的运费价格（美元） |
| product_snapshot_identifier | String! | 所购买Product的商户自定义ID |
| product_snapshot_name | String! | 所购买Product的名称 |
| tracking_number | String | 面单号 |
| shipping_provider | String | 物流提供商 |
| status | String! | `PAID`, `IN_FULFILLMENT`, `SHIPPED`, `CANCELLED`或者`DELIVERED`。其中`PAID`状态会在2小时候自动变成`IN_FULFILLMENT` |
| is_refunded | Boolean! | 是否被退款 |
| shipping_refunded | Boolean | 是否退了运费 |
| refund_reason | String | 退款原因 |

### ShippingAddress

一个地址。

| Field name        | Type           | Description  |
| ------------- |-------------| -----|
| recipient_name      | String! | 收件人姓名 |
| address1 | String! | 第一行地址 |
| address2 | String! | 第二行地址 |
| city | String! | 城市 |
| state | String! | 州或者省 |
| country | String! | 国家 |
| zipcode | String! | 邮政编码 |
| phone_number | String! | 收件人电话 |

### Product

一个商品。

| Field name        | Type           | Description  |
| ------------- |-------------| -----|
| id      | String! | 商品ID |
| identifier | String! | 商户自定义的商品ID |
| cover_image_url | String! | 商品主图链接 |
| cover_video_url | String | 商品视频链接 |
| white_background_image_url | String! | 白色背景图链接 |
| name | String! | 商品名 |
| chinese_name | String! | 商品中文名 |
| description | String! | 商品描述 |
| features | List[String!]! | 商品特性。将通过列表的形式显示给用户 |
| brand | String | 品牌 |
| tags | List[String!]! | 商品标签，用于优化搜索和推荐 |
| category_id | String! | 商品分类号 |
| extra_image_urls | List[String!]! | 更多商品图片 |
| rich_detail_sections | ? | TODO |
| target_user_type | String! | `MEN`, `WOMEN`或者`ALL`。错误的标注可能导致商品被自动下架。 |
| skus | List[Sku!]! | 商品的子Sku列表 |

### Sku

一个Sku。

| Field name        | Type           | Description  |
| ------------- |-------------| -----|
| id      | String! | SkuID |
| identifier | String! | 商户自定义的SkuID |
| cover_image_url | String! | Sku主图 |
| product_id | String! | 商品ID |
| product_identifier | String! | 商户自定义的商品ID |
| color | String! | 颜色，不能为空 |
| size | String! | 尺寸，不能为空 |
| inventory | Int! | 库存 |
| price | float! | 价格，美元 |
| shipping_price | float! | 运费价格，美元 |
| msrp | float! | [制造商建议售价](https://en.wikipedia.org/wiki/List_price) |
| shipping_time | String! | 送货时间（天），格式为`<最少天数>-<最多天数>` | 
| upc | String! | [商品统一标示号](https://en.wikipedia.org/wiki/Universal_Product_Code)，用于推荐。也就是商品标签上的扫描码 |
| enabled | Boolean! | 是否上架 |
| length | String! | 长，纯数字则默认inch |
| width | String! | 宽，纯数字则默认inch |
| height | String! | 高，纯数字则默认inch |
| weight | String! | 高，纯数字则默认单位磅 |
| hs_code | String! | 海关编码，用于报关 |
| material | String! | 材质 |


