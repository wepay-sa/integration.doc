# Refund Contract Funds

After a contract has reached the `Escrow` status (funds collected from the buyer), or a `Dispute` has been raised, you can return funds to the buyer using the refund API.

## Unified Refund Endpoint

All refund operations (full / partial, contract / milestone scope) are handled by a **single endpoint**. The request body determines the refund type and scope.

`POST /apps/api/contracts/{externalContractId}/refund`

**Path parameters**

| Field | Type | Description | Required / Notes / Example |
|---|---|---|---|
| externalContractId | string | The external contract identifier | **Required**<br/>Example: `CNT-2604-00100002` |

**Headers**
- Authorization: Bearer {access_token}
- Content-Type: application/json

---

## Request Body

| Field Name | Type | Description | Required / Notes |
|---|---|---|---|
| type | string | Refund type: `"FULL"` or `"PARTIAL"` | **Required** |
| milestoneId | integer | When provided, the refund targets a specific milestone instead of the whole contract | Optional |
| amount | number | Amount to refund to the buyer | **Required** when `type = "PARTIAL"`<br/>Must **not** be provided when `type = "FULL"`<br/>Must be `> 0` and strictly less than the base amount<br/>Max 2 decimal places |
| reason | string | Mandatory reason for the refund (kept for audit trail) | **Required**<br/>Max 500 chars |

> **Note:** The buyer's IBAN and account holder name are **not** part of the request. WePay resolves them automatically from the verified buyer record (see [Buyer Eligibility & IBAN Resolution](#buyer-eligibility--iban-resolution) below).

---

## Buyer Eligibility & IBAN Resolution

Before any refund is created, WePay runs a pre-flight check against the buyer's onboarding record:

1. **Authorization** -- the buyer must have completed the WePay authorization step. If not, the refund is rejected and an onboarding link is sent to the buyer by SMS so they can self-serve.
2. **Bank account verification** -- when the platform is configured to require a verified IBAN for the buyer (`OnboardingRequirement = KycWithIban`), the buyer must have a bank account that has been verified with the banking provider. If not, the refund is rejected and an onboarding SMS is sent.
3. **IBAN resolution** -- once eligibility passes, WePay looks up the buyer IBAN and account holder name from the verified record and writes them onto the refund.

SMS dispatch to the buyer is best-effort; SMS delivery failure does **not** change the HTTP response you receive.

Failure responses are structured so you can react programmatically:

| Condition | HTTP | Message key |
|---|---|---|
| Buyer has not completed authorization | `400` | `BuyerNotAuthorized` |
| Buyer's bank account is not verified | `400` | `BuyerBankAccountNotVerified` |
| Buyer record not found for this business | `404` | `ExternalUserNotFound` |
| IBAN could not be resolved from the verified record | `500` | `FailedToResolveIban` |

---

## Preconditions

| Scope | Required contract status | Required milestone status |
|-------|--------------------------|---------------------------|
| Contract refund (full / partial) | `Escrow` or `Dispute` | -- |
| Milestone refund (full / partial) | `Escrow` or `Dispute` | `Escrow` |

- Refund operations are **idempotent** by intent: while a non-cancelled refund of the same type already exists for the same target (contract or milestone), a new refund request will be rejected.
- Both full and partial refunds transition the contract (and milestone, when applicable) to `RefundInProgress`. The refund type is preserved on the refund record itself.

---

## Full Refund -- Contract

Returns the **entire** escrowed contract principal to the buyer. WePay platform fees and taxes are retained.

### Example Request (cURL)
```bash
curl --location 'https://api.wepay.com.sa/apps/api/contracts/CNT-2604-00100002/refund' \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "FULL",
    "reason": "Order cancelled by buyer; goods never shipped."
  }'
```

### Example Response
```json
{
    "data": {
        "refundId": 4521,
        "settlementId": null,
        "milestoneId": null,
        "refundedAmount": 1000.00,
        "releasedToSellerAmount": null,
        "type": "FullRefund",
        "externalContractId": "CNT-2604-00100002"
    },
    "message": "RefundCreatedSuccessfully",
    "status": 200,
    "validationErrors": []
}
```

---

## Partial Refund -- Contract

Returns **part** of the escrowed contract principal to the buyer. The remaining portion is released to the seller (net of the proportional fees that apply to the released amount).

### Example Request (cURL)
```bash
curl --location 'https://api.wepay.com.sa/apps/api/contracts/CNT-2604-00100002/refund' \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "PARTIAL",
    "amount": 250.00,
    "reason": "Damaged item; partial credit agreed with seller."
  }'
```

### Example Response
```json
{
    "data": {
        "refundId": 4522,
        "settlementId": 7811,
        "milestoneId": null,
        "refundedAmount": 250.00,
        "releasedToSellerAmount": 712.50,
        "type": "PartialRefund",
        "externalContractId": "CNT-2604-00100002"
    },
    "message": "PartialRefundCreatedSuccessfully",
    "status": 200,
    "validationErrors": []
}
```

---

## Full Refund -- Milestone

Returns the **entire** escrowed amount of a single milestone to the buyer.

### Example Request (cURL)
```bash
curl --location 'https://api.wepay.com.sa/apps/api/contracts/CNT-2604-00100002/refund' \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "FULL",
    "milestoneId": 13,
    "reason": "Milestone deliverable rejected; refund agreed."
  }'
```

### Example Response
```json
{
    "data": {
        "refundId": 4523,
        "settlementId": null,
        "milestoneId": 13,
        "refundedAmount": 400.00,
        "releasedToSellerAmount": null,
        "type": "FullRefund",
        "externalContractId": "CNT-2604-00100002"
    },
    "message": "RefundCreatedSuccessfully",
    "status": 200,
    "validationErrors": []
}
```

---

## Partial Refund -- Milestone

Returns **part** of a milestone's escrowed amount to the buyer; the remainder is released to the seller (net of proportional fees).

### Example Request (cURL)
```bash
curl --location 'https://api.wepay.com.sa/apps/api/contracts/CNT-2604-00100002/refund' \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "PARTIAL",
    "milestoneId": 13,
    "amount": 100.00,
    "reason": "Partial defect on milestone deliverable."
  }'
```

### Example Response
```json
{
    "data": {
        "refundId": 4524,
        "settlementId": 7812,
        "milestoneId": 13,
        "refundedAmount": 100.00,
        "releasedToSellerAmount": 285.00,
        "type": "PartialRefund",
        "externalContractId": "CNT-2604-00100002"
    },
    "message": "PartialRefundCreatedSuccessfully",
    "status": 200,
    "validationErrors": []
}
```

---

## Response Fields

All four refund operations return the same unified response: `Result<RefundResponse>`

| Field | Type | Description |
|---|---|---|
| refundId | integer | Internal WePay refund identifier -- use it to correlate with subsequent `refund.*-succeeded` webhooks |
| settlementId | integer / null | Identifier of the settlement that releases the seller portion (only for partial refunds) |
| milestoneId | integer / null | The milestone that was refunded (only for milestone-scoped refunds) |
| refundedAmount | number | Amount returned to the buyer |
| releasedToSellerAmount | number / null | Net amount released to the seller after deducting proportional fees (only for partial refunds) |
| type | string | `"FullRefund"` or `"PartialRefund"` |
| externalContractId | string | The external contract id |

---

## Status Transitions

After a successful refund request (`200`), both full and partial refunds transition the contract to `RefundInProgress`. The refund type is preserved on the refund record.

| Event | Contract status | Milestone status (if applicable) |
|-------|----------------|----------------------------------|
| Refund initiated | `RefundInProgress` | `RefundInProgress` |
| Refund succeeded (full) | `Refunded` | `Refunded` |
| Refund succeeded (partial) | `Refunded` | `Refunded` |

A webhook is fired both when the refund is **initiated** and when it **succeeds** -- see [Webhook Notifications](webhook.md).

---

## HTTP Status Codes

| Status | Description |
|---|---|
| 200 | Refund initiated successfully |
| 400 | Validation failed, contract/milestone not in a refundable status, refund amount invalid, another active refund already exists, **buyer has not completed authorization** (`BuyerNotAuthorized`), or **buyer's bank account is not verified** (`BuyerBankAccountNotVerified`) -- in the last two cases an onboarding SMS is sent to the buyer |
| 401 | Unauthorized |
| 403 | The caller is not authorized to refund this contract |
| 404 | Contract, milestone, or buyer record not found (`ExternalUserNotFound`) |
| 409 | Concurrent refund race condition detected (retry with a fresh check) |
| 500 | Buyer IBAN could not be resolved from the verified record (`FailedToResolveIban`) -- treat as a transient data issue and contact WePay support if it persists |
