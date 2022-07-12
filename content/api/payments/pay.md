---
summary:  MixPay payment link page for quickly accept crypto payments.
---

## GET /pay

You only need to set the amount and currency of the payment link, and MixPay will do the rest.

### Endpoint URL

```
https://mixpay.me/pay
```

### Authentication and options

|  |  |
| -- | -- |
| Authorization | Public Access |
| Limitation | No limitation |

### Parameters

|  Param | Optional | Type | Description |
| --- | --- | --- | --- |
| `payeeId` | <span class="required">*required</span> | String |  three settlement modes are supported, normal user, robot, and multisig group, so it is usually the Mixin UUID of a normal user or robot, and you can also specify the multisigId of a sub-account. |
| `orderId` | <span class="required">*required</span> if no `traceId` | String | Unique in your system. String length **between 6-36**, must be letters, numbers, dashes and underscores and NOT space. `orderId` and `payeeId` makes a payment unique. |
| `traceId` | optional | String |  UUID, used to prevent double payment and checking the payment result. You should use `orderId` instead.  |
| `settlementAssetId` | optional | String |  assetId of settlement cryptocurrency. Settlement assets you prefer. If left blank, the payee will receive the cryptocurrency the user pays for. |
| `quoteAssetId` | optional | String |  assetId of quote cryptocurrency, the asset include cryptocurrency and fiat currency. |
| `quoteAmount` | optional | Numeric | Amount of cryptocurrency received, if left blank, the user can enter manually. |
| `remark` | optional | String |  maximum 50. Payment remark viewable by the payee. |
| `settlementMethod` | optional | String | Instant settlement wallet. This parameter has two values, mixin and mixpay, the default is mixin. |
| `settlementMemo` | optional | String |  maximum 200. A memo similar to Mixin Snapshots, this parameter you can customize. This parameter only takes effect when your settlementMethod is equal to mixin. |
| `returnTo` | optional | String | After successful payment, the page will jump to returnTo URL. |
| `failedReturnTo` | optional | String | After payment failure, the page will jump to failedReturnTo URL. |
| `callbackUrl` | optional | String | After a payment is finish (either success or failure), MixPay will issue a POST request to this URL in our server side. For security reason, URL only support `https`.  |
| `expiredTimestamp` | optional | int | Set a expired [timestamp](https://en.wikipedia.org/wiki/Unix_time). This value must greater than 10s, and less than 240min. After this time period the payment result `status` field will be mark as `failed`, and the `failureReason` will be `Payment overtime`. If you not setting this value, the payer can have a unlimit time to complete this payment. |

## Callback

As mentioned above, you can pass a `callbackUrl` parameter to this API. 

After a payment is finish (either success or failure), MixPay will issue a POST request to this URL, with the following JSON content as a example:

```json
{
  "orderId": "xxxxxxxxxxxx",
  "traceId": "xxxxxxxxxxxx",
  "payeeId": "xxxxxxxxxxxx"
}
```

**For security reason, we can not passing the payment result in this proccess.**

When your callback endpoint receive a call:

- First in your database look for the incoming `orderId` or `traceId` value. **This step is important, be careful anyone can post a fake value the your endpoint**;
- If the previous step have a match, then call the [payments-results API](https://developers.mixpay.me/api/payments/payments-results), and check for `status` field to be `success`;
- Only when the `status` filed is `success`, now you can safely mark your order as completed. 

Your endpoint should return a HTTP status 200 with the following JSON data:

```json
{  
  "code": "SUCCESS"
}
```

Anything `code` not equal to `SUCCESS`, we will see it as failure, then our server will retry at 0s/15s/15s/30s/180s/1800s/1800s/1800s/1800s/3600s, total 10 time。

:::note
You can use [postbin](https://www.toptal.com/developers/postbin/) to test it out.
:::

### Example request - Get Payment Link

You can copy the link and open it in a brower:

```bash
https://mixpay.me/pay?payeeId=a38ed284-5689-4fb9-8c2a-91fba5e32ce9&settlementAssetId=c6d0c728-2624-429b-8e0d-d9d19b6592fa&quoteAssetId=usd&quoteAmount=10
```
