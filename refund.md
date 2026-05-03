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

> **Note:** The buyer's IBAN and account holder name are **not** part of the request. The refund API records the refund intent only; WePay resolves the buyer's verified IBAN later, when the refund is executed (see [Buyer IBAN handling](#buyer-iban-handling) below).

---

## Buyer IBAN handling

The third-party refund API does **not** check buyer onboarding status, KYC state, or IBAN availability. Calling the endpoint always records the refund intent (and transitions the contract to `RefundInProgress`) as long as the contract-level preconditions are met.

The buyer's IBAN is resolved later — at admin read time and at refund execution time — from the verified KYC source. If the buyer has not yet provided a verified IBAN when the admin tries to execute the refund, WePay drives the buyer to add one through its own onboarding SMS flow. **The third party is not involved in this loop and does not need to handle buyer-eligibility error codes** on the refund endpoint.

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
| 400 | Validation failed, contract/milestone not in a refundable status, refund amount invalid, or another active refund already exists |
| 401 | Unauthorized |
| 403 | The caller is not authorized to refund this contract |
| 404 | Contract or milestone not found |
| 409 | Concurrent refund race condition detected (retry with a fresh check) |
