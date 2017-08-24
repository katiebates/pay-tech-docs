#API reference

The GOV.UK Pay platform is based on REST principles with endpoints returning data in JSON format, and standard HTTP error response codes. The platform API allows you to:

- initiate and complete payments
- view the event history for individual payments
- view transactions within a specified time period
- provide full or partial refunds
<br /><br />


#### API overview

The base URL for the API is https://publicapi.payments.service.gov.uk/

The same base URL is now used for testing and production. The API key you use determines if the actions are treated as sandbox test payments or processed as real payments.

For full details of each API action, see the API Browser:

+ [General functions](https://gds-payments.gelato.io/docs/versions/1.0.0/resources/general)
+ [Payment ID functions](https://gds-payments.gelato.io/docs/versions/1.0.0/resources/payment-id)

You can also use our interactive [API Explorer](https://gds-payments.gelato.io/api-explorer/gov-uk-pay-api/versions/1.0.0/) to try out API calls and view responses.

See the [Quick Start Guide](https://govukpay-docs.cloudapps.digital/#quick-start-guide) section for how to set up the API Explorer. Make sure you enter your sandbox API key to avoid generating real payments!



#### API authentication

GOV.UK Pay authenticates API calls with OAuth2 HTTP bearer tokens. These are easy to use and consist of one component: your API key. Bearer tokens are specified in [RFC 6750](http://tools.ietf.org/html/rfc6750).

When making an API call, you’ll need to add your API key to an "Authorization" HTTP header and prefix it with "Bearer ". This is an example of how a header would look.

```text
Authorization: Bearer 2o9bkusxaicha5c9gnsj790of7asggo1feraoufbai4a
```


#### Payment status lifecycle

This diagram gives an overview of the payment status lifecycle and the possible outcomes.

![](https://s3-eu-west-1.amazonaws.com/pay-govuk-documentation/pay-transaction-states.png)

You can check the status of a given payment using the [**Find payment by ID**](https://gds-payments.gelato.io/api-explorer/gov-uk-pay-api/versions/1.0.0/v1/find-payment-by-id) API call.

The response will include a ``status`` value as described in the table below, and a true/false ``finished`` value which indicates if the status can change.

| status value | Meaning                                                                                        | finished value |
|--------------|------------------------------------------------------------------------------------------------|----------------|
| created      | Payment created; user has not yet visited next_url                                             | false          |
| started      | User has visited next_url and is entering payment details                                      | false          |
| submitted    | User has submitted payment details but has not yet clicked Confirm                             | false          |
| success      | User successfully completed the payment                                                        | true           |
| failed       | User didn't complete the payment, due to invalid or expired payment details, fraud block, etc. | true           |
| cancelled    | Your service cancelled the payment using an API call or the self-service site.                 | true           |
| error        | Something went wrong with GOV.UK Pay or the underlying Payment Service Provider.               | true           |


#### HTTP status codes

You will encounter typical HTTP success and error response codes when using the Pay API. Generally any:

- 100 code is informational
- 200 code indicates you’ve been successful
- 300 code indicates a redirection
- 400 code indicates a client error (your error)
- 500 code indicates a server error (something went wrong on the GOV.UK Pay end)

These are the known error codes you are likely to receive:

|Common error code| Meaning|
| --------  | -------|
|200 | Payment information request succeeded |
|201|Payment has been created |
| 204 |The server successfully processed the request, but is not returning any content |
| 400|The server cannot process the request due to a client error, e.g. missing details in the request or a failed payment cancellation|
|401|Required authentication has failed or not been provided|
|404|The resource you want cannot be found|
|412|Precondition failed: e.g. mismatch in expected refund amount available|
|422|Unprocessable entity obtained on a request validation|
|Any 500 error|Something is wrong with GOV.UK Pay - please contact us|


#### API error codes

When an error occurs, you will receive these API codes in the body of the response.

This is the format of the general JSON error response body:

```javascript
{
  "code": "PXXXX",
  "description": "Message explaining the error"
}
```

Note that the description provided is written to be informative to you, the developer, and is not intended for the end user.

Also note that extra keys, e.g ```field```, may be provided on a per-error basis.

These error codes provide more information about why a request failed.

| Request type | Error code | Meaning | Cause |
|-----------------------------|------------|-----------------------------------|---------------------------------------------------------------------------------------------------------|
| Create payment | P0101 | Missing mandatory attribute | The request you sent is missing a required attribute |
| Create payment | P0102 | Invalid attribute value | The value of an attribute you sent is invalid |
| Create payment | P0197 | Unable to parse JSON | The JSON you sent in the request body is invalid |
| Create payment | P0198 | Downstream system error | Internal error with GOV.UK Pay: contact us, quoting the error code |
| Create payment | P0199 | Account error | There is a problem with your service account: contact us, quoting the error code |
| Find payment by ID | P0200 | paymentId not found | No payment matched the paymentId you provided |
| Find payment by ID | P0298 | Downstream system error | Internal error with GOV.UK Pay: contact us, quoting the error code |
| Return payment events by ID | P0300 | paymentId not found | No payment matched the paymentId you provided |
| Return payment events by ID | P0398 | Downstream system error | Internal error with GOV.UK Pay: contact us, quoting the error code |
| Search payments | P0401 | Invalid parameters | The parameters you sent are invalid. |
| Search payments | P0402 | Page not found | The requested page of search results does not exist |
| Search payments | P0498 | Downstream system error | Internal error with GOV.UK Pay: contact us, quoting the error code |
| Cancel payment | P0500 | paymentId not found | No payment matched the paymentId you provided |
| Cancel payment | P0501 | Cancellation failed | Cancelling the payment failed; contact us, quoting the error code |
| Cancel payment | P0598 | Downstream system error | Internal error with GOV.UK Pay; contact us, quoting the error code |
| Create refund | P0600 | paymentId not found | No payment matched the paymentId you provided |
| Create refund | P0601 | Missing mandatory attribute | The request you sent is missing a required attribute |
| Create refund | P0602 | Invalid attribute value | The value of an attribute you sent is invalid |
| Create refund | P0603 | Refund not available | The payment is not available for refund. |
| Create refund | P0604 | Refund amount available mismatch | The **refund_amount_available** value you provided does not match the true amount available to refund. |
| Create refund | P0697 | Unable to parse JSON | The JSON you sent in the request body is invalid |
| Create refund | P0698 | Downstream system error | Internal error with GOV.UK Pay; contact us, quoting the error code |
| Get refund | P0700 | refundId not found | No refund matched the refundId you provided |
| Get refund | P0798 | Downstream system error | Internal error with GOV.UK Pay; contact us, quoting the error code |
| Get refunds | P0800 | refundId not found | No refund match the refundId you provided |
| Get refunds | P0898 | Downstream system error | Internal error with GOV.UK Pay; contact us, quoting the error code |
| General | P0900 | Too many requests | Your service account is sending requests above the allowed rate; try the request again in a few seconds |
| General | P0920 | Request blocked by security rules | Our firewall blocked your request. See Troubleshooting section for details. |
| General | PO999 | GOV.UK Pay is unavailable | The GOV.UK Pay service is temporarily down. |


#### Card types

These are the possible values of the card_brand parameter.

| card_brand       | type   | label            |
|------------------|--------|------------------|
| visa             | DEBIT  | Visa             |
| visa             | CREDIT | Visa             |
| master-card      | DEBIT  | Mastercard       |
| master-card      | CREDIT | Mastercard       |
| american-express | CREDIT | American Express |
| diners-club      | CREDIT | Diners Club      |
| discover         | CREDIT | Discover         |
| jcb              | CREDIT | Jcb              |
| unionpay         | CREDIT | Union Pay        |


#### API rate limits

There is a maximum rate limit for requests to the API from your service account. The limit is high and most services are unlikely ever to exceed it.

If you do exceed the limit (that is, send a large number of requests in a short amount of time), you will receive P0900 errors. If this happens, you can attempt any rate-limited requests again after a second has passed.

Please contact us if you want to discuss the rate limiting applied to your service account.