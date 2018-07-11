# Shoppo Merchant API

Merchant API是 `Shoppo` 为商户、ERP的 `API` 对接提供的API，对一下功能提供支持：

- [x] 获取、更新（TODO）商户信息；
- [x] 获取、更新（TODO）商品信息；
- [x] 获取、更新订单信息；
- [ ] 对接Shoppo物流（TODO）；

## API 请求地址
  * 正式环境：[`https://api.shoppo.com`](https://api.shoppo.com)，对应的后台地址：[`https://merchant.shoppo.com`](https://merchant.shoppo.com)
  * sanbox 环境：[`https://api-sandbox.shoppo.com`](https://api-sandbox.shoppo.com)，对应的后台地址：[`https://merchant-dev.shoppo.com`](https://merchant-dev.shoppo.com)

## 基本设置

在开始之前，请跟联系 `Shoppo` 的对接人员开通 `API` 服务。 `API` 服务开通后，您可以在 **开发者页面** ([正式环境](https://merchant.shoppo.com/merchant/developer/profile)，[sandbox环境](https://merchant-dev.shoppo.com/developer/profile)) 获取到 `商户ID` 以及 `API Key`
- **如果 `API Key` 没有刷新，请尝试注销重新登录**

您可以使用下面的命令行指令来测试 `API Key` 的有效性：

```bash
$ curl {api key对应环境的请求地址}/api/merchant/health_check/ --header "merchantid: <商户ID>" --header "apikey: <API Key>"
```

有效的 `API Key` 将会收到如下 `response`：

```
Go <商户名称>! Go Shoppo!
```

此外，请注意以下几点：
- 所有的数据均适用 `UTF-8` 编码；
- 请发送 `HTTPS` 请求，我们不保证对 `HTTP` 协议的支持；
- 每个请求的 `header` 都必须包括 `merchantid` 以及 `apiKey`；
- 对于请求的 `body`，如果需要传内容，请以 `json` 格式进行编码传输;

### 错误信息

所有成功的 `API` 请求都将会返回一个 `statusCode = 200` 的 `response`。错误的请求将会以非 `200` 的代码返回，一般包含一个错误信息。比如：

```bash
$ curl https://api-sandbox.shoppo.com/api/merchant/health_check/ --header "merchantid: <merchant id>" --header "apikey: wrong-key" -i
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

在开发测试阶段，您可能不希望一些测试的请求影响到生产数据。此时请要求对接人员提供一个测试用的 `商户ID` 以及 `API Key`，并且将所有的测试请求发送到 `https://api-sandbox.shoppo.com/`。比如

```bash
$ curl https://api-sandbox.shoppo.com/api/merchant/health_check/ --header "merchantid: <商户ID>" --header "apikey: <API Key>"
```

如果您需要一些额外的测试数据，请发邮件到 `Shoppo` 的 `API` 对接邮箱说明您的数据需求。

## API接口列表

### 读取

#### `/api/merchant/profile/` [GET]
获取当前商户信息。

参数：无

返回参数：
- `id (string!)`: 商户ID
- `merchant_name (string!)`: 商户名
- `status (string!)`: `VALID`, `PENDING_APPROVE`, `SUSPEND`或者`INVALID`。其中只有`VALID`是在经营的商户状态。

参考 `Response`：
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
- `time_created_from (timestamp?)`: 订单创建时间范围起点，类型为十位的UTC时间戳
- `time_created_to (timestamp?)`: 订单创建时间范围终点，类型为十位的UTC时间戳
- `status_in`: 订单的状态，譬如：`PAID`。如果需要填多个，则按照英文逗号分隔，譬如：`PAID,IN_PROGRESS`。
  - 订单状态筛选说明
    * `PAID`: 用户已付款等待商户操作。注意，用户付款的订单在 **2小时** 以内用户可以自由取消，请注意判断。
    * `IN_PROGRESS`: 订单已经部分发货。
    * `SHIPPED`: 订单内所有商品已发货。
    * `DELIVERED`: 用户手动点击收货。
    * `CLOSED`: 订单已关闭。

返回参数：TODO
- `total (int!)`: 总订单数；
- `bookmark (string!)`: 分页时，下一次请求需要传入的参数；
- `orders ([Order]!)`: 订单列表，请注意在遇到非法的`Order`时列表中可能包含`null`（见下面的例子）

参考 Request：

```bash
https://api-sandbox.shoppo.com/api/merchant/orders/?limit=2&bookmark=fa7GD1JmmdOI1
```

参考 Response：
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
        "status": "IN_PROGRESS",
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

参考 Request：

```bash
https://api-sandbox.shoppo.com/api/merchant/order/jzyGD1JmmdOI1/
```

参考 Response：

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
- `products ([Product]!)`: 商品列表，请注意在遇到非法的`Product`时列表中可能包含`null`。

参考 Request：

```bash
https://api-sandbox.shoppo.com/api/merchant/products/?limit=1
```


参考 Response：

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

#### `/api/merchant/products/` [POST]
创建商品

|   |Type|Description|Required|
|---|----|-----------|--------|
|**name**|`string`|商品名称| :white_check_mark: Yes|
|**identifier**|`string`|商户商品ID| :white_check_mark: Yes|
|**chinese_name**|`string`|商品中文名称|No|
|**cover_image_url**|`string`|商品主图链接, `URL`| :white_check_mark: Yes|
|**white_background_image_url**|`string`, `URL`|商品白色背景图片链接|No|
|**extra_image_urls**|`string` `[*-5]`|商品细节图, `URL`|No|
|**description**|`string`|商品描述| :white_check_mark: Yes|
|**brand**|`string`|商品品牌|No|
|**features**|`string` `[1-6]`|产品特性| :white_check_mark: Yes|
|**tags**|`string` `[]`|商品标签|No|
|**target_user_type**|`string`|适用性别, `ALL` / `MEN` / `WOMEN`| :white_check_mark: Yes|
|**skus**|`[Sku]`|Sku列表| :white_check_mark: Yes|

参数：

返回参数：
- 新创建的商品，请参考类型定义`Product`

参考 Request：

```bash
curl -X POST -s -H "Content-Type: application/json" -H "apikey: API_KEY" -H "merchantid: MERCHANT_ID" -d '{"identifier": "0.22207682727070244", "name": "product name", "chinese_name": "\u5546\u54c1\u4e2d\u6587\u540d\u79f0", "cover_image_url": "https://cdn.shoppo.com/images/10f615395c677b7fc9a56a75cf0f3bad.JPEG", "white_background_image_url": "https://cdn.shoppo.com/images/e76ffd110fb3031a325899991feb844f.JPEG", "description": "description", "features": ["feature 1", "feature 2"], "brand": "shoppo", "tags": ["beauty", "fashion"], "target_user_type": "MEN", "extra_image_urls": ["https://cdn.shoppo.com/images/10f615395c677b7fc9a56a75cf0f3bad.JPEG", "https://cdn.shoppo.com/images/e76ffd110fb3031a325899991feb844f.JPEG"], "category_id": "5010101", "skus": [{"identifier": "0.9290457919886833", "cover_image_url": "https://cdn.shoppo.com/images/10f615395c677b7fc9a56a75cf0f3bad.JPEG", "price": 1.11, "inventory": 10, "shipping_price": 0.11, "msrp": 2.2, "color": "white", "size": "L", "length": "1", "height": "2.5", "width": "1.1", "weight": "0.1", "material": "plastic", "upc": "xxxxxxx", "hs_code": "aaaaa", "shipping_time": "12-15"}]}' https://api-sandbox.shoppo.com/api/merchant/products/
```

参考 Response：

```json
{
  "data": {
    "brand": "shoppo",
    "category_id": "5010101",
    "chinese_name": "\u5546\u54c1\u4e2d\u6587\u540d\u79f0",
    "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
    "cover_video_url": null,
    "description": "description",
    "extra_image_urls": [
      "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
      "https://cdn.shoppo.com/images/719444693b58750ad7b88a8318d68353.JPEG"
    ],
    "features": [
      "feature 1",
      "feature 2"
    ],
    "id": "Mzjq1jXGNqH2",
    "identifier": "0.22207682727070244",
    "name": "product name",
    "rich_detail_sections": "Coming soon",
    "skus": [
      {
        "color": "white",
        "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
        "enabled": false,
        "height": "2.5",
        "hs_code": "aaaaa",
        "id": "5w3kXrAVlqdHA",
        "identifier": "0.9290457919886833",
        "inventory": 10,
        "length": "1",
        "material": "plastic",
        "msrp": 2.2,
        "price": 1.11,
        "product_id": "Mzjq1jXGNqH2",
        "product_identifier": "0.22207682727070244",
        "shipping_price": 0.11,
        "shipping_time": "12-15",
        "size": "L",
        "upc": "xxxxxxx",
        "weight": "0.1",
        "width": "1.1"
      }
    ],
    "tags": [
      "beauty",
      "fashion"
    ],
    "target_user_type": "MEN",
    "white_background_image_url": "https://cdn.shoppo.com/images/719444693b58750ad7b88a8318d68353.JPEG"
  }
}
```

#### `/api/merchant/product/<merchant product identifier>/` [GET]
通过商户商品ID来读取单个商户商品信息。

参数：无

返回参数：一个`Product`。

参考 Request：

```bash
https://api-sandbox.shoppo.com/api/merchant/product/test-prod-5e5b952c5a5a11e7a145f45c89a3e2f9/
```

参考 Response：

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


#### `/api/merchant/product/<merchant product identifier>/` [POST, PUT]
通过商户商品ID修改商品信息。

参数：

|   |Type|Description|Required|
|---|----|-----------|--------|
|**name**|`string`|商品名称|No|
|**chinese_name**|`string`|商品中文名称|No|
|**cover_image_url**|`string`|商品主图链接, `URL`|No|
|**extra_image_urls**|`string` `[*-5]`|商品细节图, `URL`|No|
|**white_background_image_url**|`string`|商品白色背景图, `URL`|No|
|**description**|`string`|商品描述|No|
|**brand**|`string`|商品品牌|No|
|**features**|`string` `[1-6]`|产品特性|No|
|**tags**|`string` `[]`|商品标签|No|
|**target_user_type**|`string`|适用性别, `ALL` / `MEN` / `WOMEN`|No|

返回参数：`Product`。

参考 Request：

```bash
curl -X POST -s -H "Content-Type: application/json" -H "apikey: API_KEY" -H "merchantid: MERCHANT_ID" -d '{"name": "new name from api"}' https://api-sandbox.shoppo.com/api/merchant/product/0.09917838114011723/
```

参考 Response：

```json

{
  "data": {
    "brand": "shoppo",
    "category_id": "5010101",
    "chinese_name": "\u5546\u54c1\u4e2d\u6587\u540d\u79f0",
    "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
    "cover_video_url": null,
    "description": "description",
    "extra_image_urls": [
      "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
      "https://cdn.shoppo.com/images/719444693b58750ad7b88a8318d68353.JPEG"
    ],
    "features": [
      "feature 1",
      "feature 2"
    ],
    "id": "WZ5Le5RxNjU4",
    "identifier": "0.09917838114011723",
    "name": "new name from api",
    "rich_detail_sections": "Coming soon",
    "skus": [
      {
        "color": "white",
        "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
        "enabled": false,
        "height": "2.5",
        "hs_code": "aaaaa",
        "id": "kAmwdzaXLryIX",
        "identifier": "0.9343675469927027",
        "inventory": 10,
        "length": "1",
        "material": "plastic",
        "msrp": 2.2,
        "price": 1.11,
        "product_id": "WZ5Le5RxNjU4",
        "product_identifier": "0.09917838114011723",
        "shipping_price": 0.11,
        "shipping_time": "12-15",
        "size": "L",
        "upc": "xxxxxxx",
        "weight": "0.1",
        "width": "1.1"
      },
      {
        "color": "white",
        "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
        "enabled": false,
        "height": "2.5",
        "hs_code": "aaaaa",
        "id": "Ne5AXKByjEpu1",
        "identifier": "0.09088102182121438",
        "inventory": 10,
        "length": "1",
        "material": "plastic",
        "msrp": 2.2,
        "price": 1.11,
        "product_id": "WZ5Le5RxNjU4",
        "product_identifier": "0.09917838114011723",
        "shipping_price": 0.11,
        "shipping_time": "12-15",
        "size": "L",
        "upc": "xxxxxxx",
        "weight": "0.1",
        "width": "1.1"
      }
    ],
    "tags": [
      "beauty",
      "fashion"
    ],
    "target_user_type": "MEN",
    "white_background_image_url": "https://cdn.shoppo.com/images/719444693b58750ad7b88a8318d68353.JPEG"
  }
}
```


#### `/api/merchant/product/<product id>/status/<status: enabled, disabled>` [POST, PUT]
通过 `Product ID` 来上下架整个商品。
**注意**，这个接口将会修改整个商品下关联的所有 `SKU` 的上下架状态。

**参数说明**
`product_id` -> 通过 Product 接口获取到的 `id` 字段
`status` -> `enabled` 或者 `disabled`，分别代表上架或者下架

**返回结果**：修改后的Product

参考 Request:
```
https://api-sandbox.shoppo.com/api/merchant/product/4YzNjz4KZOue/status/enabled/
```

参考 Response:
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

#### `/api/merchant/product/<merchant product identifier>/skus/` [POST]
商品添加Sku

参数：

|   |Type|Description|Required|
|---|----|-----------|--------|
|**identifier**|`string`|Sku商户自定义ID| :white_check_mark: Yes|
|**cover_image_url**|`string`|Sku主图链接, `URL`| :white_check_mark: Yes|
|**color**|`string`|颜色| :white_check_mark: Yes|
|**price**|`number`|单价, `>= 0`| :white_check_mark: Yes|
|**msrp**|`number`|吊牌价, `>= 0`|No|
|**inventory**|`integer`|库存数量, `>= 0`| :white_check_mark: Yes|
|**shipping_price**|`number`|运费, `>= 0`|No|
|**shipping_time**|`string`|送货时间(天)，格式为<最少天数>-<最多天数>|No|
|**size**|`string`|尺寸| :white_check_mark: Yes|
|**weight**|`string`|重量，默认单位为磅|No|
|**length**|`string`|长，默认单位为inch|No|
|**width**|`string`|宽，默认单位为inch|No|
|**height**|`string`|高，默认单位为inch|No|
|**upc**|`string`|商品统一标示号|No|
|**hs_code**|`string`|海关编码|No|

返回参数：一个`Sku`。

参考 Request：

```bash
curl -X POST -s -H "Content-Type: application/json" -H "apikey: API_KEY" -H "merchantid: MERCHANT_ID" -d '{"identifier": "0.20843588399138824", "cover_image_url": "https://cdn.shoppo.com/images/10f615395c677b7fc9a56a75cf0f3bad.JPEG", "price": 1.11, "inventory": 10, "shipping_price": 0.11, "msrp": 2.2, "color": "white", "size": "L", "length": "1", "height": "2.5", "width": "1.1", "weight": "0.1", "material": "plastic", "upc": "xxxxxxx", "hs_code": "aaaaa", "shipping_time": "12-15"}' https://api-sandbox.shoppo.com/api/merchant/product/10927/skus/
```

参考 Response：
```json
{
  "data": {
    "color": "white",
    "cover_image_url": "https://cdn.shoppo.com/images/2999d5a2775bf6bf773f30382f3718d9.JPEG",
    "enabled": false,
    "height": "2.5",
    "hs_code": "aaaaa",
    "id": "RmpMdQNXEGKfq",
    "identifier": "0.20843588399138824",
    "inventory": 10,
    "length": "1",
    "material": "plastic",
    "msrp": 2.2,
    "price": 1.11,
    "product_id": "mREDzEAQr6iq",
    "product_identifier": "10927",
    "shipping_price": 0.11,
    "shipping_time": "12-15",
    "size": "L",
    "upc": "xxxxxxx",
    "weight": "0.1",
    "width": "1.1"
  }
}
```


#### `/api/merchant/sku/<merchant sku identifier>/` [GET]
通过商户SKU ID来读取单个SKU的信息。

参数：无

返回参数：一个`Sku`。

参考 Request：

```bash
https://api-sandbox.shoppo.com/api/merchant/sku/test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9/
```

参考 Response：

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

#### `/api/merchant/sku/<merchant sku identifier>/` [POST, PUT]
修改单个 `SKU` 的基本信息。

参数：

|   |Type|Description|Required|
|---|----|-----------|--------|
|**cover_image_url**|`string`|Sku主图链接|No|
|**color**|`string`|颜色|No|
|**price**|`number`|单价|No|
|**msrp**|`number`|吊牌价|No|
|**inventory**|`integer`|库存数量|No|
|**shipping_price**|`number`|运费|No|
|**shipping_time**|`string`|送货时间(天)，格式为<最少天数>-<最多天数>|No|
|**size**|`string`|尺寸|No|
|**weight**|`string`|重量，默认单位为磅|No|
|**length**|`string`|长，默认单位为inch|No|
|**width**|`string`|宽，默认单位为inch|No|
|**height**|`string`|高，默认单位为inch|No|
|**upc**|`string`|商品统一标示号|No|
|**hs_code**|`string`|海关编码|No|

返回结果：修改后的 SKU 数据
参考 Request:
```
curl -X POST --header "merchantid: <商户ID>" --header "apikey: <API Key> https://api-sandbox.shoppo.com/api/merchant/sku/test-sku-5e5d4f485a5a11e7a58df45c89a3e2f9/ -d '{"inventory": 15}'
```

参考 Response:
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
    "inventory": 15,
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

#### `/api/merchant/sku/<sku id>/status/<status: enabled, disabled>` [POST, PUT]
修改单个 `SKU` 的上下架状态。
参数：
- `sku id`: 通过 product 或者 sku 接口获取到的 sku 数据的 id 字段
- `status`: enabled 或者 disabled，分别表示上架和下架状态

返回结果：修改后的 SKU 数据
参考 Request:
```
https://api-sandbox.shoppo.com/api/merchant/sku/pvmj64QXLDaux/status/enabled/
```

参考 Response:
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

#### `/api/merchant/create-logistics-order/` [POST]

OrderItem自动生成面单号，**多个order item只能属于同一个商户订单**，每次请求会生成一个新的面单号

参数：
* `courier_code(String)`: 物流方式代码, `Enum`
  * `PEKING_EMS`, 物流方式名称 `SHOPPO - Post`，支持美国地区订单
  * `OFFLINE_EMS`，物流方式名称 `SHOPPO - E邮宝`，支持美国地区订单
  * `SHANGHAI_EMS`，物流方式名称 `SHOPPO - E邮包`，支持印度地区订单
* `order_item_ids([String])`: 要生成面单的order items id列表，这些order item会使用同一个包裹发货；如果需要用多个包裹发货，需要拆成多次请求，获取多个面单号。

返回参数：order item列表

参考 Request：

```
https://api-sandbox.shoppo.com/api/merchant/create-logistics-order/
```

参考 Payload:

```json
{
	"courier_code": "",
	"order_item_ids": ["mEDWDgOWK8gc6g", "pRwWwg8WOeet6d"]
}
```

参考 Response:

```json
{
  "data": {
    "order_items": [
      {
        "id": "mEDWDgOWK8gc6g",
        "is_refunded": false,
        "product_snapshot_identifier": "test-prod-5e5dc7525a5a11e7beb5f45c89a3e2f9",
        "product_snapshot_name": "test prod name 5e5dc7525a5a11e7beb5f45c89a3e2f9",
        "quantity": 1,
        "refund_reason_type": null,
        "shipping_provider": "wise-express",
        "shipping_refunded": false,
        "sku_snapshot_identifier": "test-sku-5e5de7ac5a5a11e79e6bf45c89a3e2f9",
        "sku_snapshot_price": 111.62,
        "sku_snapshot_shipping_price": 2.0,
        "status": "IN_FULFILLMENT",
        "tracking_number": "UG856257945CN"
      },
      {
        "id": "pRwWwg8WOeet6d",
        "is_refunded": false,
        "product_snapshot_identifier": "test-prod-124c234c5a5d11e793a3f45c89a3e2f9",
        "product_snapshot_name": "test prod name 124c234c5a5d11e793a3f45c89a3e2f9",
        "quantity": 1,
        "refund_reason_type": null,
        "shipping_provider": "wise-express",
        "shipping_refunded": false,
        "sku_snapshot_identifier": "test-sku-124c72d25a5d11e7850ef45c89a3e2f9",
        "sku_snapshot_price": 158.18,
        "sku_snapshot_shipping_price": 5.5,
        "status": "IN_FULFILLMENT",
        "tracking_number": "UG856257945CN"
      }
    ]
  }
}
```

#### `/api/merchant/order_item/<order item ID>/` [POST, PUT]
修改单个商户订单信息。**注意，仅修改提供的参数**。

参数：
- `tracking_number (String)`: 物流面单号。请注意面单号仅能够在该订单还未发货（`status=SHIPPED`）之前修改，发货之后无法通过API修改面单号。此外，我们仅支持几家常用的物流提供商，如果您的物流单号无法被识别，请联系Shoppo的客服人员解决。
- `shipping_provider (String)`: 物流商名称（比如EMS, China Post），注意`shipping_provider`必须和`tracking_number`同时修改。

返回参数：一个`OrderItem`。

参考 Request：

```
https://api-sandbox.shoppo.com/api/merchant/order_item/KvL2Lb12jnMTpw/
```

参考 Payload:

```json
{
	"tracking_number": "31837953231",
	"shipping_provider": "China Post"
}
```

参考 Response:

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

#### `/api/merchant/order_items/ship/` [POST, PUT]
将多个`OrderItem`标记为已发货。

参数：
- `order_item_ids ([String!]!)`: 发货的`OrderItem`ID列表。注意，仅有`PAID`和`IN_FULFILLMENT`状态、并且已经有合法`tracking_number`的订单可以发货。

返回参数：已发货的`OrderItem`列表。

参考 Request：

```
https://api-sandbox.shoppo.com/api/merchant/order_items/ship/
```

参考 Payload:

```json
{
	"order_item_ids": ["KvL2Lb12jnMTpw"]
}
```

参考 Response:
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

#### `/api/merchant/order_item/<order item ID>/refund/` [POST, PUT]
将单个`OrderItem`退款。注意，refund并不会改变`OrderItem`的状态。一个已经退款的`OrderItem`不能被重复退款（除非第一次没有退运费，第二次退运费）。

参数：
- `refund_shipping (Boolean!)`: 是否退运费。注意，一般情况下，未发货的订单一定要求退运费。
- `refund_reason_type (String!)`：退款原因。`OUT_OF_STOCK`还是`BAD_SHIPPING_ADDRESS`还是`AGREEMENT_WITH_USER`。

返回参数：退款的order item。

参考 Request：

```
https://api-sandbox.shoppo.com/api/merchant/order_item/KvL2Lb12jnMTpw/refund
```

参考 Payload:

```json
{
	"refund_shipping": true,
    "refund_reason_type": "OUT_OF_STOCK"
}
```

参考 Response:
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

#### `/api/merchant/supported-couriers/` 获取支持的物流方式信息
获取当前系统中支持的物流方式以及对应的说明。其中，`code` 字段为 `API` 接受的格式，`name` 为对应方式的名称/说明。

参考 Request:
```
curl -X GET -H 'merchantid:<MerchantID>' -H 'apikey:<ApiKey>' -H 'mimetype:Application/json' 'https://api.shoppo.com/api/merchant/supported-couriers/'
```

参考 Response:
```
{
  "data": [
    {
      "code": "sunyou",
      "name": "Sunyou / \u987a\u53cb"
    },
    {
      "code": "amazon-fba-us",
      "name": "Amazon FBA USA"
    },
    {
      "code": "ppbyb",
      "name": "PayPal Package"
    },
    {
      "code": "4px",
      "name": "4PX"
    },
    {
      "code": "usps",
      "name": "USPS"
    },
    {
      "code": "yanwen",
      "name": "Yanwen"
    },
    {
      "code": "china-ems",
      "name": "China EMS (ePacket)"
    },
    {
      "code": "dhl",
      "name": "DHL Express"
    },
    {
      "code": "ups",
      "name": "UPS"
    },
    {
      "code": "yunexpress",
      "name": "Yun Express"
    },
    {
      "code": "fedex",
      "name": "FedEx"
    },
    {
      "code": "china-post",
      "name": "China Post"
    },
    {
      "code": "wise-express",
      "name": "Wise Express"
    }
  ]
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
| weight | String! | 重量，纯数字则默认单位磅 |
| hs_code | String! | 海关编码，用于报关 |
| material | String! | 材质 |
