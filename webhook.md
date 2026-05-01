# Webhook Notifications

## Overview

Webhooks allow you to receive real-time notifications about contract events. Instead of polling the API for updates, WePay sends HTTP POST requests to your configured endpoint whenever important events occur.

**Key Benefits:**

- **Real-time Updates**: Get notified immediately when events occur.
- **Reduced API Calls**: No need to poll for status changes.
- **Reliable Delivery**: Automatic retries with exponential backoff.
- **Secure**: HMAC-SHA256 signature verification.

---

## Webhook Event Types

WePay sends webhooks for the following events:

| Event Type | Description |
|---|---|
| contract.created | A new contract has been created |
| contract.approved | Contract has been approved by the other party |
| contract.rejected | Contract has been rejected |
| contract.cancelled | Contract has been cancelled |
| payment.completed | Payment has been completed and funds are in escrow |
| contract.delivered | Seller marked the contract as delivered |
| contract.received | Buyer confirmed receipt of delivery |
| contract.released | Funds have been released to the seller |
| contract.disputed | A dispute has been raised on the contract |
| contract.refunded | Contract has been refunded to the buyer. Legacy combined event emitted on the `Refunded` status transition |
| refund.full-initiated | A full refund, contract or milestone, has been initiated. The contract enters `RefundInProgress` |
| refund.full-succeeded | A full refund has settled with the bank. The contract or milestone is now `Refunded` |
| refund.partial-initiated | A partial refund, contract or milestone, has been initiated. The contract enters `RefundInProgress` |
| refund.partial-succeeded | A partial refund has settled. The contract or milestone is now `Refunded` |
| contract.completed | Contract has been fully completed |
| webhook.test | Test webhook sent from `/webhooks/test` endpoint |

---

## Configure Webhook Subscription

Before receiving webhooks, register your webhook URL.

### Endpoint

`POST /apps/api/webhooks`

### Headers

```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "webhookUrl": "https://yourdomain.com/webhooks/wepay"
}
```

### Field Descriptions

| Field | Type | Required | Description |
|---|---|---|---|
| webhookUrl | string | Yes | Public HTTPS endpoint URL that receives webhook notifications. Example: `https://api.yoursite.com/webhooks/wepay` |

### Example Request

```bash
curl -X POST "{baseUrl}/apps/api/webhooks" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "webhookUrl": "https://yourdomain.com/webhooks/wepay"
  }'
```

### Example Response

```json
{
  "data": {
    "id": "019bb692-28e0-7ae1-926f-59b12c3b784c",
    "webhookUrl": "https://yourdomain.com/webhooks/wepay",
    "secretKey": "whsec_abc123xyz789defghijklmnopqrstuvwxyz",
    "isActive": true,
    "consecutiveFailures": 0,
    "lastSuccessAt": null,
    "lastFailureAt": null,
    "createdAt": "2026-01-19T12:00:00Z"
  },
  "message": "Webhook created successfully.",
  "status": 200,
  "validationErrors": []
}
```

### Response Fields

| Field | Description |
|---|---|
| id | Business entity ID used as webhook subscription identifier |
| webhookUrl | Registered webhook endpoint |
| secretKey | Secret key for signature verification. Save it securely |
| isActive | Whether the webhook subscription is active |
| consecutiveFailures | Number of consecutive delivery failures. Resets on success |
| lastSuccessAt | Timestamp of last successful delivery |
| lastFailureAt | Timestamp of last failed delivery |
| createdAt | Creation timestamp |

> **Important:** Save the `secretKey` securely. You need it to verify webhook signatures. The secret is only shown when creating or regenerating it. Never expose it in client-side code.

---

## Get Webhook Subscription

Retrieve current webhook configuration.

### Endpoint

`GET /apps/api/webhooks`

### Headers

```http
Authorization: Bearer {access_token}
```

### Example Request

```bash
curl -X GET "{baseUrl}/apps/api/webhooks" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "data": {
    "id": "019bb692-28e0-7ae1-926f-59b12c3b784c",
    "webhookUrl": "https://yourdomain.com/webhooks/wepay",
    "secretKey": "whsec_abc123xyz789defghijklmnopqrstuvwxyz",
    "isActive": true,
    "consecutiveFailures": 0,
    "lastSuccessAt": "2026-01-19T14:30:00Z",
    "lastFailureAt": null,
    "createdAt": "2026-01-19T12:00:00Z"
  },
  "message": "",
  "status": 200,
  "validationErrors": []
}
```

---

## Update Webhook URL

Update the existing webhook subscription with a new URL.

### Endpoint

`POST /apps/api/webhooks`

### Headers

```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "webhookUrl": "https://newdomain.com/webhooks/wepay"
}
```

### Example Request

```bash
curl -X POST "{baseUrl}/apps/api/webhooks" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "webhookUrl": "https://newdomain.com/webhooks/wepay"
  }'
```

### Example Response

```json
{
  "data": {
    "id": "019bb692-28e0-7ae1-926f-59b12c3b784c",
    "webhookUrl": "https://newdomain.com/webhooks/wepay",
    "secretKey": "whsec_abc123xyz789defghijklmnopqrstuvwxyz",
    "isActive": true,
    "consecutiveFailures": 0,
    "lastSuccessAt": "2026-01-19T14:30:00Z",
    "lastFailureAt": null,
    "createdAt": "2026-01-19T12:00:00Z"
  },
  "message": "Webhook updated successfully.",
  "status": 200,
  "validationErrors": []
}
```

> **Note:** Updating the webhook URL also re-enables the subscription if it was disabled due to consecutive failures.

---

## Regenerate Secret Key

Regenerate the secret key immediately if it is compromised.

### Endpoint

`POST /apps/api/webhooks/regenerate-secret`

### Headers

```http
Authorization: Bearer {access_token}
```

### Example Request

```bash
curl -X POST "{baseUrl}/apps/api/webhooks/regenerate-secret" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "data": {
    "secretKey": "whsec_xYz123AbCdEfGhIjKlMnOpQrStUvWxYz456"
  },
  "message": "Webhook secret regenerated successfully.",
  "status": 200,
  "validationErrors": []
}
```

> **Important:** After regenerating, update your server immediately to use the new secret. Webhooks signed with the old key will fail verification.

---

## Test Webhook

Send a test webhook to verify your endpoint.

### Endpoint

`POST /apps/api/webhooks/test`

### Headers

```http
Authorization: Bearer {access_token}
```

### Example Request

```bash
curl -X POST "{baseUrl}/apps/api/webhooks/test" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "data": {
    "webhookId": "123e4567-e89b-12d3-a456-426614174000",
    "message": "Test webhook has been queued for delivery"
  },
  "message": "Test webhook sent.",
  "status": 200,
  "validationErrors": []
}
```

Your endpoint receives a test webhook with event type `webhook.test`.

---

## View Delivery Logs

View webhook delivery attempts.

### Endpoint

`GET /apps/api/webhooks/logs?page=1&pageSize=20`

### Headers

```http
Authorization: Bearer {access_token}
```

### Query Parameters

| Parameter | Type | Default | Description |
|---|---|---:|---|
| page | integer | 1 | Page number |
| pageSize | integer | 20 | Number of records per page. Max 100 |

### Example Request

```bash
curl -X GET "{baseUrl}/apps/api/webhooks/logs?page=1&pageSize=20" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "data": {
    "logs": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "eventType": "payment.completed",
        "externalContractId": "CNT-2601-00100068",
        "contractId": 12345,
        "httpStatusCode": 200,
        "isSuccess": true,
        "attemptNumber": 1,
        "errorMessage": null,
        "durationMs": 245,
        "createdAt": "2026-01-19T14:30:00Z"
      }
    ],
    "totalCount": 150,
    "page": 1,
    "pageSize": 20
  },
  "message": "",
  "status": 200,
  "validationErrors": []
}
```

### Log Fields

| Field | Description |
|---|---|
| id | Unique log entry identifier |
| eventType | Event type sent |
| externalContractId | External contract ID |
| contractId | Internal contract ID |
| httpStatusCode | HTTP status returned by your endpoint |
| isSuccess | Whether delivery was successful |
| attemptNumber | Delivery attempt number |
| errorMessage | Error message if delivery failed |
| durationMs | Request duration in milliseconds |
| createdAt | Delivery attempt timestamp |

---

## Delete Webhook Subscription

Remove your webhook subscription to stop receiving notifications.

### Endpoint

`DELETE /apps/api/webhooks`

### Headers

```http
Authorization: Bearer {access_token}
```

### Example Request

```bash
curl -X DELETE "{baseUrl}/apps/api/webhooks" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "data": true,
  "message": "Webhook deleted successfully.",
  "status": 200,
  "validationErrors": []
}
```

---

## Webhook Payload Format

All webhooks are sent as HTTP POST requests with JSON body.

### Headers Sent with Each Webhook

| Header | Description |
|---|---|
| Content-Type | `application/json` |
| X-WePay-Signature | HMAC-SHA256 signature for verification |
| X-WePay-Event | Event type, for example `payment.completed` |
| X-WePay-Webhook-Id | Unique identifier for this webhook delivery |
| X-WePay-Timestamp | ISO 8601 timestamp when webhook was created |

### Webhook Payload Structure

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "event": "payment.completed",
  "createdAt": "2026-01-19T14:30:00Z",
  "data": {
    "contractId": "CNT-2601-00100068",
    "reference": "your-reference-123",
    "status": "Escrow",
    "previousStatus": "Approved",
    "amount": 600.00,
    "currency": "SAR",
    "paymentId": "PAY-123456",
    "transactionId": "TXN-789012",
    "invoiceId": "INV-345678",
    "timestamp": "2026-01-19T14:30:00Z",
    "metadata": {
      "metadata1": "your-custom-data-1",
      "metadata2": "your-custom-data-2",
      "metadata3": "your-custom-data-3",
      "metadata4": "your-custom-data-4",
      "reference": "your-reference-123"
    }
  }
}
```

### Payload Field Descriptions

| Field | Description |
|---|---|
| id | Unique webhook delivery ID |
| event | Event type that triggered the webhook |
| createdAt | Webhook creation timestamp |
| data.contractId | External contract ID |
| data.reference | Your reference from contract creation, if provided |
| data.status | Current contract status |
| data.previousStatus | Previous contract status |
| data.amount | Event amount in SAR |
| data.currency | Currency code. Always `SAR` |
| data.paymentId | Payment ID for payment events |
| data.transactionId | Transaction ID for payment events, or refund ID for refund events |
| data.invoiceId | Invoice ID for payment events |
| data.timestamp | Event timestamp |
| data.metadata | Metadata from contract creation and event-specific metadata |

> **Note:** Some fields like `paymentId`, `transactionId`, and `invoiceId` are only present for relevant events.

---

## Refund Event Payloads

Refund operations fire two webhooks for each refund:

1. An `*-initiated` event when the refund request is accepted.
2. An `*-succeeded` event when the refund has settled with the bank.

Use `data.transactionId`, which carries the WePay refund identifier, to correlate initiated and succeeded events.

For milestone-level refunds, the affected milestone is identified using `data.metadata.milestoneId`.

---

### `refund.full-initiated`

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "event": "refund.full-initiated",
  "createdAt": "2026-04-14T14:30:00Z",
  "data": {
    "contractId": "CNT-2604-00100002",
    "reference": "your-reference-123",
    "status": "RefundInProgress",
    "previousStatus": "Escrow",
    "amount": 1000.00,
    "currency": "SAR",
    "transactionId": "4521",
    "timestamp": "2026-04-14T14:30:00Z",
    "metadata": {
      "metadata1": "your-custom-data-1",
      "reference": "your-reference-123"
    }
  }
}
```

---

### `refund.full-succeeded`

```json
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
  "event": "refund.full-succeeded",
  "createdAt": "2026-04-14T15:05:00Z",
  "data": {
    "contractId": "CNT-2604-00100002",
    "reference": "your-reference-123",
    "status": "Refunded",
    "previousStatus": "RefundInProgress",
    "amount": 1000.00,
    "currency": "SAR",
    "transactionId": "4521",
    "timestamp": "2026-04-14T15:05:00Z",
    "metadata": {
      "reference": "your-reference-123"
    }
  }
}
```

---

### `refund.partial-initiated`

```json
{
  "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
  "event": "refund.partial-initiated",
  "createdAt": "2026-04-14T14:32:00Z",
  "data": {
    "contractId": "CNT-2604-00100002",
    "reference": "your-reference-123",
    "status": "RefundInProgress",
    "previousStatus": "Escrow",
    "amount": 250.00,
    "currency": "SAR",
    "transactionId": "4522",
    "timestamp": "2026-04-14T14:32:00Z",
    "metadata": {
      "reference": "your-reference-123"
    }
  }
}
```

> `data.amount` on partial refund events is the buyer refund amount, not the seller release amount.

---

### `refund.partial-succeeded`

```json
{
  "id": "d4e5f6a7-b8c9-0123-def0-456789012345",
  "event": "refund.partial-succeeded",
  "createdAt": "2026-04-14T15:10:00Z",
  "data": {
    "contractId": "CNT-2604-00100002",
    "reference": "your-reference-123",
    "status": "Refunded",
    "previousStatus": "RefundInProgress",
    "amount": 250.00,
    "currency": "SAR",
    "transactionId": "4522",
    "timestamp": "2026-04-14T15:10:00Z",
    "metadata": {
      "reference": "your-reference-123"
    }
  }
}
```

---

### Milestone-level Refund Metadata

Milestone-level refunds use the same refund event types as contract-level refunds.

The affected milestone is identified by `data.metadata.milestoneId`.

```json
{
  "event": "refund.full-initiated",
  "data": {
    "contractId": "CNT-2604-00100002",
    "status": "RefundInProgress",
    "previousStatus": "Escrow",
    "amount": 400.00,
    "currency": "SAR",
    "transactionId": "4523",
    "metadata": {
      "milestoneId": "13",
      "reference": "your-reference-123"
    }
  }
}
```

> **Important:** Refund API usage is documented in `README.md`. This file only documents webhook configuration, delivery, verification, and payloads.

---

## Verifying Webhook Signatures

Verify the webhook signature before processing any webhook.

### Signature Format

`X-WePay-Signature` contains:

```txt
sha256={hex_signature}
```

### Verification Process

1. Extract the signature from `X-WePay-Signature`.
2. Read the raw JSON request body.
3. Compute HMAC-SHA256 using your webhook secret key.
4. Compare the computed signature with the received signature using constant-time comparison.

---

### Node.js Example

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secretKey) {
  const expectedSignature = 'sha256=' + crypto
    .createHmac('sha256', secretKey)
    .update(payload, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

app.post('/webhooks/wepay', (req, res) => {
  const signature = req.headers['x-wepay-signature'];
  const payload = JSON.stringify(req.body);

  if (!verifyWebhookSignature(payload, signature, YOUR_SECRET_KEY)) {
    return res.status(401).send('Invalid signature');
  }

  const event = req.body;

  switch (event.event) {
    case 'payment.completed':
      break;
    case 'contract.released':
      break;
    case 'refund.full-initiated':
    case 'refund.full-succeeded':
    case 'refund.partial-initiated':
    case 'refund.partial-succeeded':
      break;
  }

  res.status(200).send('OK');
});
```

---

### PHP Example

```php
<?php
function verifyWebhookSignature($payload, $signature, $secretKey) {
    $expectedSignature = 'sha256=' . hash_hmac('sha256', $payload, $secretKey);
    return hash_equals($expectedSignature, $signature);
}

$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_WEPAY_SIGNATURE'] ?? '';

if (!verifyWebhookSignature($payload, $signature, YOUR_SECRET_KEY)) {
    http_response_code(401);
    exit('Invalid signature');
}

$event = json_decode($payload, true);

switch ($event['event']) {
    case 'payment.completed':
        break;
    case 'contract.released':
        break;
    case 'refund.full-initiated':
    case 'refund.full-succeeded':
    case 'refund.partial-initiated':
    case 'refund.partial-succeeded':
        break;
}

http_response_code(200);
echo 'OK';
?>
```

---

### Python Example

```python
import hmac
import hashlib
from flask import Flask, request, abort

app = Flask(__name__)
SECRET_KEY = 'your_webhook_secret_key'

def verify_signature(payload, signature, secret):
    expected = 'sha256=' + hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

@app.route('/webhooks/wepay', methods=['POST'])
def webhook_handler():
    signature = request.headers.get('X-WePay-Signature', '')
    payload = request.get_data(as_text=True)

    if not verify_signature(payload, signature, SECRET_KEY):
        abort(401)

    event = request.get_json()

    if event['event'] == 'payment.completed':
        pass
    elif event['event'] == 'contract.released':
        pass
    elif event['event'] in [
        'refund.full-initiated',
        'refund.full-succeeded',
        'refund.partial-initiated',
        'refund.partial-succeeded'
    ]:
        pass

    return 'OK', 200
```

---

### C# Example

```csharp
using System.Security.Cryptography;
using System.Text;
using System.Text.Json;

public class WebhookController : ControllerBase
{
    private readonly string _secretKey = "your_webhook_secret_key";

    private bool VerifyWebhookSignature(string payload, string signature)
    {
        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(_secretKey));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(payload));
        var expectedSignature = "sha256=" + Convert.ToHexString(hash).ToLowerInvariant();

        return CryptographicOperations.FixedTimeEquals(
            Encoding.UTF8.GetBytes(signature),
            Encoding.UTF8.GetBytes(expectedSignature)
        );
    }

    [HttpPost("webhooks/wepay")]
    public async Task<IActionResult> HandleWebhook()
    {
        var signature = Request.Headers["X-WePay-Signature"].ToString();

        using var reader = new StreamReader(Request.Body);
        var payload = await reader.ReadToEndAsync();

        if (!VerifyWebhookSignature(payload, signature))
        {
            return Unauthorized();
        }

        var webhookEvent = JsonSerializer.Deserialize<WebhookEvent>(payload);

        switch (webhookEvent?.Event)
        {
            case "payment.completed":
                break;
            case "contract.released":
                break;
            case "refund.full-initiated":
            case "refund.full-succeeded":
            case "refund.partial-initiated":
            case "refund.partial-succeeded":
                break;
        }

        return Ok();
    }
}
```

> **Important:** Always use constant-time comparison (`timingSafeEqual`, `hash_equals`, `compare_digest`, `FixedTimeEquals`) to prevent timing attacks.

---

## Retry Policy

If your endpoint does not respond with a 2xx status code, WePay retries delivery with exponential backoff.

| Attempt | Delay |
|---|---|
| 1st retry | 10 seconds |
| 2nd retry | 30 seconds |
| 3rd retry | 2 minutes |
| 4th retry | 10 minutes |
| 5th retry | 1 hour |

After 5 failed retries, delivery is marked as failed.

### Retryable Status Codes

These status codes will trigger retries:

- 408 Request Timeout
- 429 Too Many Requests
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout

### Non-Retryable Status Codes

These status codes will NOT trigger retries (considered permanent failures):
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found

### Auto-Disable Policy

After **10 consecutive failed deliveries**, your webhook subscription will be automatically disabled to prevent unnecessary retries. To re-enable:
1. Fix your endpoint issue.
2. Update the webhook URL using `POST /apps/api/webhooks`.

---

## Webhook Best Practices

### 1. Respond Quickly

- Return `200 OK` as soon as you receive the webhook.
- Process the event asynchronously if needed.
- Timeout is 30 seconds. Respond faster to avoid retries.

### 2. Handle Duplicates

- Webhooks may be delivered more than once.
- Use `X-WePay-Webhook-Id` to deduplicate.
- Make processing idempotent.

### 3. Verify Signatures

- Always verify `X-WePay-Signature` before processing.
- Reject unsigned or incorrectly signed requests.
- Use constant-time comparison.

### 4. Use HTTPS

- Production webhook URLs must use HTTPS.
- Keep SSL certificates valid and renewed.

### 5. Log Everything

- Log received webhooks.
- Log signature verification results.
- Keep logs for troubleshooting failed deliveries.

### 6. Monitor Your Endpoint

- Check delivery logs through `/apps/api/webhooks/logs`.
- Set alerts for consecutive failures.
- Test periodically using `/apps/api/webhooks/test`.

---

## Webhook Error Handling

### Webhook Configuration Errors

| Error | Solution |
|---|---|
| 400 Invalid URL | Ensure URL is valid and uses HTTP or HTTPS |
| 401 Unauthorized | Check access token validity |
| 404 Not Found | Create webhook subscription first |

### Example Error Response

```json
{
  "message": "Invalid webhook URL",
  "status": 400,
  "validationErrors": [
    {
      "field": "webhookUrl",
      "message": "URL must be a valid HTTP or HTTPS URL"
    }
  ]
}
```

---

## Complete Webhook Integration Flow

1. **Register Webhook URL**
   - Call `POST /apps/api/webhooks`
   - Save the `secretKey` securely

2. **Implement Webhook Handler**
   - Create endpoint at your `webhookUrl`
   - Implement signature verification
   - Handle different event types

3. **Test Your Integration**
   - Call `POST /apps/api/webhooks/test`
   - Verify your endpoint receives the test webhook
   - Check signature verification works

4. **Go Live**
   - Create contracts via API
   - Monitor delivery logs for any failures

5. **Ongoing Maintenance**
   - Check `/apps/api/webhooks/logs` periodically
   - Regenerate secret if compromised
   - Update URL if endpoint changes

---

## Webhook FAQ

### Q: How do I know if my webhook subscription is working?

**A:** Use the test endpoint (`POST /apps/api/webhooks/test`) and check delivery logs via `GET /apps/api/webhooks/logs`.

### Q: My webhook was disabled. How do I re-enable it?

**A:** Update your webhook URL via `POST /apps/api/webhooks`. This resets the failure counter and re-enables the subscription.

### Q: Can I have multiple webhook URLs?

**A:** No, each business can have only one webhook subscription. All events are sent to the same URL. The webhook configuration is tied to your business entity.

### Q: What if I miss a webhook?

**A:** You can always query the contract status via the API. Webhooks are for real-time updates, but the API is the source of truth.

### Q: Is the webhook payload the same for all events?

**A:** The structure is the same, but some fields (like `paymentId`, `transactionId`) are only present for relevant events (e.g., `payment.completed`).

### Q: How long are webhook delivery logs retained?

**A:** WePay retains delivery logs that you can access via the API. We recommend keeping your own logs for at least 90 days for troubleshooting.

### Q: What happens if my server is down when a webhook is sent?

**A:** WePay will retry delivery up to 5 times with exponential backoff (10s, 30s, 2min, 10min, 1hr). If all retries fail, check your delivery logs when your server is back up.

