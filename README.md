

# For Third-Party Developers - Integration Guide

This section is for developers integrating WePay Escrow Payment Services into their applications.

## Version
current version: v1.1.0 <br/>
last updated date: 01/02/2026

## Change Log


| Version        | What to Include              |
| -------------- | ---------------------------- |
| Minor (v1.1.0) | Webhook notifications support |
| Major (v1.0.0) | checkout on contract level public              |


[[_TOC_]]


## Overview

To integrate WePay payment services, you need to:

- Obtain an access token using client credentials
- Create a contract using the API
- Redirect users to the checkout URL provided in the response

## Step 1: Obtain Access Token

Before making any API calls, you need to authenticate and obtain an access token.

### Endpoint

`POST /connect/token`  
**Headers**:
- content-type: application/x-www-form-urlencoded
- grant_type: client_credentials
- client_id: YOUR_CLIENT_ID
- client_secret: YOUR_CLIENT_SECRET  

**Where to get Client ID and Client Secret:**

- These credentials are generated when you create a business account on the WePay platform
- Contact WePay support to obtain these credentials

### Example Request (cURL)

```
curl -X POST " https://test-api.welink-sa.com/connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"

```

### Example Response
```
{  
"access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjBNVUdGT1IxSjlWSURKVlFISV9UMjFfSlBDVTA5WlotS0dFQVRHWDciLCJ0eXAiOiJhdCtqd3QifQ...",  
"token_type": "Bearer",  
"expires_in": 1440  
}
```
**Response Fields:**

- **access_token**: The bearer token to use for authenticated API requests
- **token_type**: Always "Bearer" for this authentication method
- **expires_in**: Token expiration time in minutes (1440 = 24 hours)

**Important Notes:**

- Store the access token securely
- Tokens expire after the time specified in `expires_in` (typically 24 hours)
- You'll need to request a new token when it expires
- Never expose your client_secret in client-side code

## Step 2: Create a Contract

After obtaining the access token, use it to create a payment contract.

### Endpoint

`POST /apps/api/contracts`  

**Headers**
 - Authorization: Bearer {access_token}  Replace {access_token} with the token obtained from Step 1.
 - Content-Type: application/json   

### Request Body
```
{
    "title": "Sell a property111",
    "contractServiceType": "Product",
    "BuyerParty": {
        "phoneNumber": "966583944460",
        "firstName": "Buyer",
        "lastName": "Name aa"
    },
    "SellerParty": {
        "phoneNumber": "966583944461",
        "firstName": "Seller",
        "lastName": ""
    },
    "amount": 1000,
    "description": "Extenal Iphone mobile",
    "notes": "notes for mobile",
    "reference": "12321",
    "metaData": {
        "metadata1": "1",
        "metadata2": "2",
        "metadata3": "",
        "metadata4": ""
    },
    "callbackUrl": "http://localhost:3000/callback"
}
```
### Field Descriptions

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| title | string | Contract title | **Required**<br/>Example: "Sell a property111" |
| contractServiceType | string | Type of service for the contract | **Required**<br/>Expected Values: "Product", "Service" |
| BuyerParty | object (Party) | Buyer information | **Required** |
| SellerParty | object (Party) | Seller information | **Required** |
| amount | number | Contract amount | **Required**<br/>Example: 1000 |
| description | string | Contract description | Optional<br/>Example: "Extenal Iphone mobile" |
| notes | string | Additional notes | Optional<br/>Example: "notes for mobile" |
| reference | string | External reference identifier | Optional<br/>Example: "12321" |
| metaData | object (MetaData) | Custom key-value metadata | Optional |
| callbackUrl | string | Callback URL for contract events | Optional<br/>Example: "<http://localhost:3000/callback>" |

**Party (BuyerParty / SellerParty)**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| phoneNumber | string | Party phone number (including country code) | **Required**<br/>Example: "966583944460" |
| firstName | string | First name | **Required**<br/>Example: "Ahmad" |
| lastName | string | Last name | **Required**<br/>Example: "Ali" |

**MetaData**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| metadata1 | string | Custom metadata field | OptionalExample: "1" |
| metadata2 | string | Custom metadata field | OptionalExample: "2" |
| metadata3 | string | Custom metadata field | OptionalExample: "" |
| metadata4 | string | Custom metadata field | OptionalExample: "" |

## Response: 

**Result&lt;ExternalCreateContractResponse&gt;**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| succeeded | boolean | Indicates whether the request was successful | **Required**<br/>Example: true |
| message | string | Result message | Optional<br/>Example: "Contract created successfully" |
| errors | array of string | List of validation or business errors | Optional |
| data | object (ExternalCreateContractResponse) | Contract creation result data | Optional |

**ExternalCreateContractResponse**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| contractId | string | Unique external contract identifier | **Required**<br><br>Example: "c8f1a3c2-9d12-4c8b-9f0a-123456789abc" |
| status | string (ContractStatus) | Current contract status | **Required**<br><br>Example: "Pending" |
| contractServiceType | string (ContractServiceType) | Type of contract service | **Required**<br><br>Example: "Product" |
| checkoutUrl | string | Checkout URL for completing payment | **Required**<br><br>Example: "<https://checkout.example.com/contract/123>" |
| buyerParty | object (ExternalContractParty) | Buyer party details | **Required** |
| sellerParty | object (ExternalContractParty) | Seller party details | **Required** |
| reference | string | External reference identifier | Optional Nullable |
| metaData | object (ContractMetaData) | Custom metadata | Optional Nullable |
| milestones | array of Milestone | Contract milestones | **Required** |
| pricingLineItems | array of ExternalPriceLineItem | Pricing breakdown items | **Required** |
| createdDate | string (date-time) | Contract creation timestamp (UTC) | **Required**<br><br>Example: 2024-01-15T10:30:00Z |

**ExternalContractParty**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| firstName | string | First name | **Required**<br/>Example: "Ahmad" |
| lastName | string | Last name | **Required**<br/>Example: "Ali" |
| phoneNumber | string | Phone number including country code | **Required**<br/>Example: "966583944460" |

**ContractMetaData**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| metadata1 | string | Custom metadata field | Optional |
| metadata2 | string | Custom metadata field | Optional |
| metadata3 | string | Custom metadata field | Optional |
| metadata4 | string | Custom metadata field | Optional |

**Milestone**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| name | string | Milestone name | **Required**<br><br>Example: "Delivery" |
| description | string | Milestone description | Optional |
| amount | number | Milestone amount | **Required**<br><br>Example: 500 |
| dueDate | string (date-time) | Milestone due date (UTC) | **Required**<br><br>Example: 2024-02-01T00:00:00Z |

**ExternalPriceLineItem**

| Field Name | Type | Description | Required / Notes / Example |
| --- | --- | --- | --- |
| lineType | string (PricingLineType) | Pricing line type | **Required**<br><br>Example: "EscrowFee" |
| role | string (PricingRole) | Party responsible for the amount | **Required**<br><br>Example: "Buyer" |
| amount | number | Amount value | **Required**<br><br>Example: 50 |
| description | string | Pricing line description | Optional |
| order | integer | Display order | **Required**<br><br>Example: 1 |

### Example Request (cURL)
```
curl -X POST "https://api.wepay.com.sa/apps/api/contracts" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "TEST Sell a property111",
    "contractServiceType": "Product",
    "BuyerParty": {
      "phoneNumber": "966583944460",
      "firstName": "Buyer",
      "lastName": "Name aa"
    },
    "SellerParty": {
      "phoneNumber": "966583944461",
      "firstName": "Seller",
      "lastName": ""
    },
    "amount": 600,
    "description": "TEST External Iphone mobile",
    "notes": "notes for mobile",
    "callbackurl": "http://localhost:3000/en/payment-success"
  }'

```
### Example Response

```
{
	"data": {
		"contractId": "CNT-2601-00100003",
		"status": "pending",
		"contractServiceType": "product",
		"checkoutUrl": "<https://checkout.welink-sa.com?contractId=CNT-2601-00100003&token=RSGFF38PXVO>....",
		"buyerParty": {
			"firstName": "Buyer",
			"lastName": "Name aa",
			"phoneNumber": "966583944460"
		},
		"sellerParty": {
			"firstName": "Seller",
			"lastName": "",
			"phoneNumber": "966583944461"
		},
		"reference": "12321",
		"metaData": {
			"metadata1": "1",
			"metadata2": "2",
			"metadata3": "",
			"metadata4": ""
		},
		"milestones": [],
		"pricingLineItems": [
			{
				"lineType": "contractAmount",
				"role": "buyer",
				"amount": 1000,
				"description": null,
				"order": 1
			},
			{
				"lineType": "escrowFee",
				"role": "thirdParty",
				"amount": 25.0,
				"description": null,
				"order": 2
			},
			{
				"lineType": "escrowFeeTax",
				"role": "thirdParty",
				"amount": 3.75,
				"description": null,
				"order": 3
			},
			{
				"lineType": "externalPlatformFee",
				"role": "buyer",
				"amount": 100.0,
				"description": null,
				"order": 4
			},
			{
				"lineType": "externalPlatformFeeTax",
				"role": "buyer",
				"amount": 15.0,
				"description": null,
				"order": 5
			},
			{
				"lineType": "netEscrowAmountToSeller",
				"role": "seller",
				"amount": 1000,
				"description": null,
				"order": 6
			}
		],
		"createdDate": "2026-01-25T10:28:10.4874015Z"
	},
	"message": "Contract created successfully.",
	"status": 200,
	"validationErrors": []
}
```

## Get Contract
in case you want to get previously created contract by contract id, you can use the following endpoint 

**Endpoint** `GET {{baseUrl}}/apps/api/contracts/CNT-2601-00100002`
Curl:
```
```bash
curl -X 'GET' \
  '{{baseUrl}}/apps/api/contracts/CNT-2601-00100002' \
  -H 'accept: application/json'
```
**Response**
```
{
	"data": {
		"contractId": "CNT-2601-00100002",
		"reference": "12321",
		"title": "Sell a property111",
		"description": "Extenal Iphone mobile",
		"notes": "notes for mobile",
		"status": "pending",
		"contractServiceType": "product",
		"buyerParty": {
			"firstName": "Buyer",
			"lastName": "Name aa",
			"phoneNumber": "966583944460"
		},
		"sellerParty": {
			"firstName": "Seller",
			"lastName": "",
			"phoneNumber": "966583944461"
		},
		"businessName": "new 555",
		"createdDate": "2026-01-24T17:48:58.64975+01:00",
		"metaData": {
			"metadata1": "1",
			"metadata2": "2",
			"metadata3": "",
			"metadata4": ""
		},
		"milestones": [],
		"pricingLineItems": [
			{
				"lineType": "contractAmount",
				"role": "buyer",
				"amount": 1000,
				"description": null,
				"order": 1
			},
			{
				"lineType": "escrowFee",
				"role": "thirdParty",
				"amount": 25.0,
				"description": null,
				"order": 2
			},
			{
				"lineType": "escrowFeeTax",
				"role": "thirdParty",
				"amount": 3.75,
				"description": null,
				"order": 3
			},
			{
				"lineType": "externalPlatformFee",
				"role": "buyer",
				"amount": 100.0,
				"description": null,
				"order": 4
			},
			{
				"lineType": "externalPlatformFeeTax",
				"role": "buyer",
				"amount": 15.0,
				"description": null,
				"order": 5
			},
			{
				"lineType": "netEscrowAmountToSeller",
				"role": "seller",
				"amount": 1000,
				"description": null,
				"order": 6
			}
		],
		"transactions": []
	},
	"message": "ContractRetrievedSuccessfully",
	"status": 200,
	"validationErrors": []
}

```
 **Response Fields Description**
`Result<GetContractResponse>`
| Field Name | Type                         | Description                                  | Required / Notes / Example |
| ---------- | ---------------------------- | -------------------------------------------- | -------------------------- |
| succeeded  | boolean                      | Indicates whether the request was successful | **Required**               |
| message    | string                       | Response message                             | Optional                   |
| errors     | array of string              | Errors if the request failed                 | Optional                   |
| data       | object (GetContractResponse) | Contract details                             | Optional                   | 


`GetContractResponse`
| Field Name          | Type                                     | Description                      | Required / Notes / Example |
| ------------------- | ---------------------------------------- | -------------------------------- | -------------------------- |
| id                  | string (GUID)                            | Contract identifier              | **Required**               |
| title               | string                                   | Contract title                   | **Required**               |
| description         | string                                   | Contract description             | Optional                   |
| notes               | string                                   | Additional notes                 | Optional                   |
| amount              | number                                   | Total contract amount            | **Required**               |
| status              | string (ContractStatus)                  | Current contract status          | **Required**               |
| contractServiceType | string (ContractServiceType)             | Contract service type            | **Required**               |
| reference           | string                                   | External reference               | Optional                   |
| buyerParty               | object (ContractPartyResponse)           | Buyer party details              | **Required**               |
| sellerParty              | object (ContractPartyResponse)           | Seller party details             | **Required**               |
| metaData            | object (ContractMetaData)                | Custom metadata                  | Optional                   |
| milestones          | array of ContractMilestoneResponse       | Contract milestones              | **Required**               |
| pricingLineItems    | array of ContractPricingLineItemResponse | Pricing breakdown                | **Required**               |
| createdDate         | string (date-time) (UTC)                      | Contract creation date (UTC)     | **Required**               |
| updatedDate         | string (date-time) (UTC)                      | Last update date (UTC)           | **Required**               |

`ContractPartyResponse`
| Field Name  | Type   | Description                    | Required / Notes / Example |
| ----------- | ------ | ------------------------------ | -------------------------- |
| firstName   | string | First name                     | **Required**               |
| lastName    | string | Last name                      | Optional                   |
| phoneNumber | string | Phone number with country code | **Required**               |

`ContractMetaData`
| Field Name | Type   | Description     | Required / Notes / Example |
| ---------- | ------ | --------------- | -------------------------- |
| metadata1  | string | Custom metadata | Optional                   |
| metadata2  | string | Custom metadata | Optional                   |
| metadata3  | string | Custom metadata | Optional                   |
| metadata4  | string | Custom metadata | Optional                   |

`ContractMilestoneResponse`

| Field Name  | Type               | Description           | Required / Notes / Example |
| ----------- | ------------------ | --------------------- | -------------------------- |
| id          | string (GUID)      | Milestone identifier  | **Required**               |
| name        | string             | Milestone name        | **Required**               |
| description | string             | Milestone description | Optional                   |
| amount      | number             | Milestone amount      | **Required**               |
| status      | string             | Milestone status      | **Required**               |
| dueDate     | string (date-time (UTC)) | Milestone due date    | **Required**               |

`ContractPricingLineItemResponse`

| Field Name  | Type          | Description                   | Required / Notes / Example |
| ----------- | ------------- | ----------------------------- | -------------------------- |
| id          | string (GUID) | Pricing line identifier       | **Required**               |
| lineType    | string        | Pricing line type             | **Required**               |
| role        | string        | Party responsible for the fee | **Required**               |
| amount      | number        | Amount value                  | **Required**               |
| description | string        | Pricing line description      | Optional                   |
| order       | integer       | Display order                 | **Required**               |


`HTTP Status Codes`

| Status Code | Description                     |
| ----------- | ------------------------------- |
| 200         | Contract retrieved successfully |
| 404         | Contract not found              |
| 400         | Invalid contract ID             |
| 401         | Unauthorized                    |
| 500         | Internal server error           |




## Step 3: Redirect Users to Checkout

After receiving the response, extract the `checkoutUrl` from `data.checkoutUrl` and redirect your users to this URL.

### Implementation Examples

**HTML Link:**

```
<a href="https://checkout.wepay.com.sa?contractId=CNT-2601-00100068&token=...">Pay with WePay</a>
```

**JavaScript Redirect:**
```
// After receiving the API response  
const checkoutUrl = response.data.checkoutUrl;  
window.location.href = checkoutUrl;
```

**React/Next.js:**
```
// After receiving the API response  
const checkoutUrl = response.data.checkoutUrl;  
<Link href={checkoutUrl}>Pay with WePay</Link>  
  
// Or redirect programmatically  
router.push(checkoutUrl);
```

**PHP:**

```
// After receiving the API response  
$checkoutUrl = $response['data']['checkoutUrl'];  
header("Location: " . $checkoutUrl);  
exit;
```

**Python (Flask/Django):**

```
<![endif]-->

# Flask  
from flask import redirect  
checkout_url = response['data']['checkoutUrl']  
return redirect(checkout_url)  
  
# Django  
from django.shortcuts import redirect  
checkout_url = response['data']['checkoutUrl']  
return redirect(checkout_url)
```

## Complete Integration Flow

1. Your backend calls POST /connect/token.   Receive access_token.  
1. Your backend calls POST /apps/api/contracts with access_token.  
1.  Receive checkoutUrl in response.   Redirect user to checkoutUrl.  
1. User completes payment on WePay platform.   User is redirected to
1.  your callbackurl.   Handle payment success/failure on your callback
    page.

```mermaid
sequenceDiagram
    autonumber
    participant Backend as Your Backend
    participant Auth as Auth Server
    participant API as WePay API
    participant User as User
    participant WePay as WePay Platform
    participant Callback as Callback URL

    Backend ->> Auth: POST /connect/token
    Auth -->> Backend: access_token

    Backend ->> API: POST /apps/api/contracts (access_token)
    API -->> Backend: checkoutUrl

    Backend -->> User: Redirect to checkoutUrl
    User ->> WePay: Complete payment
    WePay -->> Callback: Redirect user (success/failure)

    Callback ->> Backend: Handle payment result
    Backend -->> User: Show success / failure page
```

## Error Handling

### Token Request Errors

**401 Unauthorized:**

- Invalid client_id or client_secret
- Solution: Verify your credentials

**400 Bad Request:**

- Missing required fields
- Solution: Ensure grant_type, client_id, and client_secret are provided

### Contract Creation Errors

**401 Unauthorized:**

- Invalid or expired access token
- Solution: Request a new token

**400 Bad Request:**

- Missing required fields
- Invalid field values
- Check \`validationErrors\` array in response for details

**Example Error Response:**

```
{
	"message": "Validation failed",
	"status": 400,
	"validationErrors": [
		{
			"field": "amount",
			"message": "Amount must be greater than 0"
		}
	]
}
```
# Enums Documentation

## ContractStatus

| Value | Name | Description |
|---:|---|---|
| 1 | Draft | Contract created but not yet progressed. |
| 2 | Pending | Awaiting counterparty action or response. |
| 3 | Accepted | Counterparty has accepted the contract terms. |
| 4 | Approved | Contract approved (e.g., by platform/admin). |
| 5 | Rejected | Contract was rejected by a party or reviewer. |
| 6 | Expired | Contract validity period has expired. |
| 7 | Completed | All contract obligations fulfilled. |
| 8 | Cancelled | Contract cancelled before completion. |
| 9 | WaitingForBankPayment | Awaiting an external bank payment to arrive. |
| 10 | Escrow | Funds are held in escrow for the contract. |
| 11 | Dispute | A dispute has been raised on the contract. |
| 12 | Released | Escrowed funds have been released. |
| 13 | Refunded | Full refund has been processed. |
| 14 | PartiallyRefunded | A partial refund has been processed. |
| 15 | Terminated | Contract forcibly terminated (end-of-life). |

## ContractServiceType

| Value | Name | Description |
|---:|---|---|
| 1 | Product | A tangible or digital product (goods). |
| 2 | Service | A service (time, labor, or expertise). |

## PricingRole

| Value | Name | Description |
|---:|---|---|
| 1 | Buyer | The buyer (payer) in the pricing flow. |
| 2 | Seller | The seller (payee) in the pricing flow. |
| 3 | SplitEqually | Costs split equally between parties. |
| 4 | ThirdParty | A third party is responsible for the charge. |

## PricingLineType

| Value | Name | Description |
|---:|---|---|
| 1 | ContractAmount | Primary contract amount (base price). |
| 2 | EscrowFee | Platform escrow fee charged on the contract. |
| 3 | EscrowFeeTax | Tax applied to the escrow fee. |
| 4 | ExternalPlatformFee | Fee charged by an external platform. |
| 5 | ExternalPlatformFeeTax | Tax applied to the external platform fee. |
| 100 | NetEscrowAmountToSeller | Derived informational line (not a charge); net amount to seller. |

## FeePayer

| Value | Name | Description |
|---:|---|---|
| 1 | Buyer | Buyer pays the fee. |
| 2 | Seller | Seller pays the fee. |
| 3 | SplitEqually | Fee split equally between buyer and seller. |
| 4 | ThirdParty | A third party covers the fee. |
| 5 | PerContract | Fee applied per contract (billing unit). |


# Check out page getting started

## Accessing the Platform

- You will receive a payment link from your broker or service provider
- The link will include:

- A **token** (for authentication)
- A **contract ID** (identifying your specific contract)

- Click the link or paste it into your browser
- The platform will automatically detect your preferred language (English or Arabic)

## Prerequisites

Before starting, ensure you have:

- A valid Saudi National ID or Iqama number
- Access to the phone number registered with Absher (for Absher verification)
- A valid payment method (credit/debit card)


## 1. Order Summary Card

Displays your contract details in a summary table.

### Contract Details

| Field                  | Description                                |
| ---------------------- | ------------------------------------------ |
| **Contract Title**     | The name or description of your contract   |
| **Description**        | Additional details about the contract      |
| **Escrow Fees**        | Fee charged for escrow protection services |
| **Tax on Escrow Fees** | VAT applied to escrow fees                 |
| **Amount**             | Total contract amount to be paid           |

> 💡 All amounts are displayed in **SAR (﷼)**.

---

## 2. Escrow Protection Information

Below the order summary, information about escrow protection is displayed.

### Protection Features

* Funds are held securely in escrow and released when work is completed
* Both parties can raise disputes through the **WePay dashboard**
* Your payment remains protected until delivery milestones are met

---

## 3. Action Buttons

### **Pay with WePay** Button

Click this button to proceed to the next step.

**Behavior:**

* If **KYC is not completed** → Redirects to **KYC verification**
* If **KYC is completed** → Redirects to **payment summary**

## When This Screen Appears

- Always appears first when you access the payment link
- Shows your contract details and total amount due

# KYC Verification Process

## Overview

KYC (Know Your Customer) verification is required to ensure secure transactions. The process consists of up to three steps, depending on your verification status.

## Step Flow Logic

The system determines which steps you need to complete:

- **Personal Info Step**: Appears if KYC is not Completed.
- **Income Info Step**: Appears if KYC is not Completed.
- **Absher Step**: Appears if is not Verified with Absher yet.

**Note**: Steps are skipped if you've already completed them in a previous session.

## Step 1: Personal Information

![fP9t2V9.png](https://iili.io/fP9t2V9.png)

### When This Step Appears

- Appears when you haven't completed KYC verification yet
- First step in the KYC process
- Required before proceeding to payment

### Required Fields

| Field | Required? | Validation |
| --- | ---: | --- |
| First Name | ✅ | <ul><li>Required field (cannot be empty)</li><li>Maximum 100 characters</li></ul> |
| Last Name | ✅ | <ul><li>Required field (cannot be empty)</li><li>Maximum 100 characters</li></ul> |
| National / Iqama ID | ✅ | <ul><li>Required field (cannot be empty)</li><li>Must be exactly 10 digits</li><li>Must start with either 1 or 2</li></ul> |


## Step 2: Income Information
![fP9bjwl.png](https://iili.io/fP9bjwl.png)
### When This Step Appears

- Appears after completing Personal Information step
- Only shown if KYC verification is not yet completed

### Questions Display

The system fetches KYC questions from the server. Questions may vary, but typically include:

**Common Questions:**

- **Tax Resident Question**
**Question**: "Are you a tax resident in any country outside Saudi Arabia?" / "هل أنت مقيم ضريبيًا في أي دولة خارج المملكة العربية السعودية؟"
***Options***:
	- "Yes" / "نعم"
	- "No" / "لا"
	- **Required**: Yes

- **Political Exposure Question**

- **Question**: "Are you a politically exposed person, or is any of your first-degree relatives involved in politics, or do you have a close relationship with any political party or politician?" / "هل أنت شخص مُعرّض سياسيًا، أو هل يشارك أي من أقاربك من الدرجة الأولى في العمل السياسي، أو هل تربطك علاقة وثيقة بأي حزب سياسي أو شخصية سياسية؟"
- **Options**:
- "Yes" / "نعم"
- "No" / "لا"
- **Required**: Yes

**Note**: Actual questions are fetched from the server and may differ.

### How to Answer

- Read each question carefully
- Select your answer by clicking on "Yes" or "No"
- All questions are required before proceeding
- You can see your selected answers highlighted

### After Submission

- If Absher is not verified: Proceeds to **Step 3: Absher**
- If Absher is already verified: Redirects to **Payment Summary**

## Step 3: Absher Verification

### When This Step Appears

- Appears when it's not Absher Verified yet.
- Can appear after Personal Info (if KYC is complete) or after Income Info
- Required for completing the verification process

### What is Absher?

Absher is Saudi Arabia's national digital identity platform. This step verifies your identity using the phone number registered with your Absher account.

![fP9m8WN.png](https://iili.io/fP9m8WN.png)

### Process Overview

- **Automatic OTP Send**: When you reach this step, an OTP is automatically sent to your registered Absher phone number
- **Enter OTP**: You receive a 6-digit code via SMS
- **Verify**: Enter the code to complete verification

### Resend OTP Section

**Timer:**

- After requesting OTP, a 2-minute countdown timer appears
- Format: "2:00 minute" / "2:00 دقيقة" or "30 second" / "30 ثانية"
- Resend button is disabled during countdown
- After timer expires, you can click "Resend"

**How to Resend:**

- Wait for the timer to reach 0:00
- Click the "Resend" button
- A new OTP will be sent to your Absher-registered phone number

### Validation

- **Required**: All 6 digits must be entered
- **Format**: Only numeric digits (0-9) are accepted
- **Error**: If verification fails, an error message will appear

### After Verification

- **Success**: Redirects to **Payment Summary** screen
- **Failure**: Error message displayed, you can try again or resend OTP

# Payment Summary Screen

## Overview

After completing KYC verification, you'll see the payment summary screen where you can review details and select a payment method.

## When This Screen Appears

- Appears after completing all required KYC steps
- Also appears if you've already completed KYC in a previous session

### 1. Payment Summary Card

**Contract Details:**

- **Contract ID**: Your unique contract identifier
- **Contract Title**: Name of your contract
- **Buyer**: Your name (from Personal Info step)
- **Seller**: The seller's name

**Financial Information:**

- **Contract Amount**: The base amount of the contract
- **You Will Pay**: Total amount highlighted in blue (larger font)

### 2. Payment Methods Section

Below the summary, you'll see available payment options:

**Supported Payment Gateways:**

- Credit/debit card payments
- STC Pay**: STC Pay wallet
- Mada**: Mada card payments
- Apple Pay**: Apple Pay (if available)

## How to Proceed

- Review all contract details carefully
- Verify the total amount you'll pay
- Select your preferred payment method
- You'll be redirected to the payment gateway's secure page
- Complete payment on the gateway's page
- After successful payment, you'll be redirected to the Payment Success screen

![fPHdebs.png](https://iili.io/fPHdebs.png)

# Test Data

You can test the payment using this information:

| Field | Value |
| --- | --- |
| Name On Card | WePay Solutions |
| IBAN | 5123 4500 0000 0008 |
| Date | 01 / 27 |
| CCV | 100 |


# Payment Success Screen

## Overview

You'll get redirected to the callbackurl you provided when created the contract with more params in it about the transaction

### Callbackurl Details Card

- **Status**: The status of the payment process either success or failed.
- **paymentId**: It's the payment id you can use it see payment details in the dashboard or call API from the broker to read it.
- **contractId**: It's the contract id that is related to the payment.

## What Happens Next

- **Payment Held**: Your payment is securely held in escrow
- **Work Completion**: The seller completes the work/service
- **Approval**: You approve the completed work
- **Release**: Funds are released to the seller
- **Dispute**: If needed, you can raise a dispute through the WePay dashboard

# Language Support

## Supported Languages

The platform supports two languages:

- **English (en)**: Default language
- **Arabic (ar)**: Full RTL (Right-to-Left) support

## Language Detection

- Language is automatically detected based on:
- Your browser settings
- URL locale parameter (e.g., `en` or `ar`)
- Previously selected language preference

## Language Switching

- Language is typically set by the URL you receive
- All text, buttons, and messages are translated
- Form fields and validation messages adapt to the selected language

## RTL Support

- Arabic interface uses RTL layout
- Text alignment, buttons, and forms adjust automatically
- OTP input fields remain LTR for numeric entry

# Troubleshooting

## Common Issues and Solutions

### 1. Cannot Access the Payment Link

**Problem**: Link doesn't work or shows an error

**Solutions**:

- Verify the link is complete (includes token and contract ID)
- Check your internet connection
- Try copying and pasting the link directly into the browser
- Contact your broker or service provider

### 2. Personal Information Validation Errors

**Problem**: ID number is rejected

**Solutions**:

- Ensure you're entering exactly 10 digits
- Check that it starts with 1 (Saudi national) or 2 (resident)
- Remove any spaces or dashes
- Verify you're using the correct ID/Iqama number

**Example of Valid IDs**:

- ✅ `1234567890` (Saudi national)
- ✅ `2234567890` (Resident)
- ❌ `123456789` (only 9 digits)
- ❌ `3234567890` (doesn't start with 1 or 2)

### 3. OTP Not Received

**Problem**: Didn't receive Absher OTP code

**Solutions**:

- Wait for the 2-minute countdown timer to finish (Make sure your National/Iqama ID is correct )
- Click "Resend" after the timer expires
- Verify your phone number is registered with Absher
- Check your SMS inbox and spam folder
- Ensure your phone has signal
- Contact support if the issue persists

### 4. OTP Verification Fails

**Problem**: Entered OTP but verification fails

**Solutions**:

- Double-check all 6 digits are correct
- Ensure no extra spaces or characters
- Request a new OTP and try again
- Verify the OTP hasn't expired (usually valid for a few minutes)
- Make sure you're using the most recent OTP sent

### 5\. Payment Method Not Working

**Problem**: Cannot complete payment with selected method

**Solutions**:

- Try a different payment method
- Verify your card details are correct
- Check your card has sufficient funds
- Ensure your card is enabled for online payments
- Contact your bank if the issue persists

### 6. Page Loading Issues

**Problem**: Page takes too long to load or doesn't load

**Solutions**:

- Refresh the page (F5 or Ctrl+R)
- Clear your browser cache
- Try a different browser
- Check your internet connection
- Disable browser extensions that might interfere

### 7. Form Submission Stuck

**Problem**: "Submitting..." message doesn't go away

**Solutions**:

- Wait a few moments (may be processing)
- Refresh the page
- Check your internet connection
- Try again from the previous step
- Contact support if the issue persists

### 8. Wrong Language Displayed

**Problem**: Interface shows in wrong language

**Solutions**:

- Check the URL contains the correct locale (`en` or `ar`)
- Clear browser cookies and try again
- Contact your broker to provide the correct language link

## Getting Help

If you encounter issues not covered here:

**Contact Your Broker**: They can provide support for contract-specific issues

**WePay Support**: For payment and platform issues

**Absher Support**: For Absher verification problems

## Important Notes

- **Never share your token or contract ID** with unauthorized parties
- **Keep your transaction ID** for records and support
- **Screenshots** of error messages can help support resolve issues faster
- **Complete the process in one session** when possible to avoid timeouts

# Quick Reference Guide

## Step-by-Step Checklist

- ✅ **Access Payment Link**

- Click the link provided by your broker
- Verify contract details on checkout screen

- ✅ **Complete Personal Information**

- Enter first name
- Enter last name
- Enter 10-digit ID/Iqama (starts with 1 or 2)
- Click "Next"

- ✅ **Answer Income Questions** (if required)

- Answer all KYC questions
- Click "Next"

- ✅ **Verify with Absher** (if required)

- Wait for OTP SMS
- Enter 6-digit code
- Click "Complete Verification"

- ✅ **Review Payment Summary**

- Verify contract details
- Check total amount
- Select payment method

- ✅ **Complete Payment**

- Enter payment details on gateway
- Confirm payment

- ✅ **Payment Success**

- Review transaction details
- Copy payment ID and contract ID to review the details on the dashboard.

## Field Requirements Summary

## Validation Rules Summary

- **Personal Info**: All fields required, ID must be valid format
- **Income Info**: All questions must be answered
- **Absher**: OTP must be exactly 6 digits, must match sent code

# Security and Privacy

## Data Protection

- All personal information is encrypted during transmission
- Payment details are processed through secure gateways
- Your data is protected according to Saudi Arabia's data protection regulations

## Escrow Protection

- Funds are held securely until work completion
- Payment is only released upon your approval

## Best Practices

- Use secure, private networks when making payments
- Never share your payment link or token
- Keep your transaction ID for records
- Log out after completing payment if using a shared device

# Frequently Asked Questions (FAQ)

## Q: What happens if I close the browser during payment?

**A**: If you've completed KYC, you can return to the link and proceed directly to payment. If payment was in progress, check with your bank or payment gateway.

## Q: Is my payment information stored?

**A**: Payment details are processed securely through payment gateways. WePay doesn't store your full card details.

# Conclusion

This guide covers all aspects of using the WePay Third-Party Payment Integration platform. Follow the steps carefully, and don't hesitate to contact support if you need assistance.

**Remember**:

- Complete all required fields accurately
- Keep your payment ID for records
- Your payment is protected by escrow
- Support is available if you encounter issues

Thank you for using WePay!

---

# Webhook Notifications

## Overview

Webhooks allow you to receive real-time notifications about contract events. Instead of polling the API for updates, WePay will send HTTP POST requests to your specified endpoint whenever important events occur.

**Key Benefits:**

- **Real-time Updates**: Get notified immediately when events occur
- **Reduced API Calls**: No need to poll for status changes
- **Reliable Delivery**: Automatic retries with exponential backoff
- **Secure**: HMAC-SHA256 signature verification

## Webhook Event Types

WePay sends webhooks for the following events:

| Event Type | Description |
| --- | --- |
| contract.created | A new contract has been created |
| contract.approved | Contract has been approved by the other party |
| contract.rejected | Contract has been rejected |
| contract.cancelled | Contract has been cancelled |
| payment.completed | Payment has been completed and funds are in escrow |
| contract.delivered | Seller marked the contract as delivered |
| contract.received | Buyer confirmed receipt of delivery |
| contract.released | Funds have been released to the seller |
| contract.disputed | A dispute has been raised on the contract |
| contract.refunded | Contract has been refunded to the buyer |
| contract.completed | Contract has been fully completed |
| webhook.test | Test webhook (sent via /webhooks/test endpoint) |

---

## Configure Webhook Subscription

Before receiving webhooks, you need to register your webhook URL.

### Endpoint

`POST /apps/api/webhooks`

**Headers:**
```
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

| Field | Type | Required? | Description |
| --- | --- | --- | --- |
| webhookUrl | String | Yes | Your HTTPS endpoint URL to receive webhook notifications. Must be publicly accessible. Example: `https://api.yoursite.com/webhooks` |

### Example Request (cURL)
```bash
curl -X POST "https://api.wepay.com.sa/apps/api/webhooks" \
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
| --- | --- |
| id | Your business entity ID (used as webhook subscription identifier) |
| webhookUrl | Your registered webhook endpoint |
| secretKey | Secret key for signature verification (**Save this securely**) |
| isActive | Whether the webhook is currently active (false if disabled due to failures) |
| consecutiveFailures | Number of consecutive delivery failures (resets to 0 on success) |
| lastSuccessAt | Timestamp of last successful delivery |
| lastFailureAt | Timestamp of last failed delivery |
| createdAt | When the business entity was created |

> **Important:** Save the `secretKey` securely. You'll need it to verify webhook signatures. The secret is only shown once when creating or regenerating. Never expose your secret key in client-side code.

---

## Get Webhook Subscription

Retrieve your current webhook configuration.

### Endpoint

`GET /apps/api/webhooks`

**Headers:**
```
Authorization: Bearer {access_token}
```

### Example Request (cURL)
```bash
curl -X GET "https://api.wepay.com.sa/apps/api/webhooks" \
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

Update your existing webhook subscription with a new URL.

### Endpoint

`POST /apps/api/webhooks`

**Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body
```json
{
  "webhookUrl": "https://newdomain.com/webhooks/wepay"
}
```

### Example Request (cURL)
```bash
curl -X POST "https://api.wepay.com.sa/apps/api/webhooks" \
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

If your secret key is compromised, regenerate it immediately.

### Endpoint

`POST /apps/api/webhooks/regenerate-secret`

**Headers:**
```
Authorization: Bearer {access_token}
```

### Example Request (cURL)
```bash
curl -X POST "https://api.wepay.com.sa/apps/api/webhooks/regenerate-secret" \
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

> **Important:** After regenerating, update your server immediately to use the new secret key. Webhooks signed with the old key will fail verification.

---

## Test Webhook

Send a test webhook to verify your endpoint is working correctly.

### Endpoint

`POST /apps/api/webhooks/test`

**Headers:**
```
Authorization: Bearer {access_token}
```

### Example Request (cURL)
```bash
curl -X POST "https://api.wepay.com.sa/apps/api/webhooks/test" \
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

Your endpoint will receive a test webhook with event type `webhook.test`. This is a special event type used only for testing and will not be triggered by actual contract operations.

---

## View Delivery Logs

View the history of webhook delivery attempts.

### Endpoint

`GET /apps/api/webhooks/logs?page=1&pageSize=20`

**Headers:**
```
Authorization: Bearer {access_token}
```

### Query Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| page | Integer | 1 | Page number for pagination |
| pageSize | Integer | 20 | Number of records per page (max 100) |

### Example Request (cURL)
```bash
curl -X GET "https://api.wepay.com.sa/apps/api/webhooks/logs?page=1&pageSize=20" \
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
| --- | --- |
| id | Unique log entry identifier (UUID format) |
| eventType | Type of event that was sent |
| externalContractId | External contract ID (e.g., "CNT-2601-00100068") |
| contractId | Internal contract ID (integer) |
| httpStatusCode | HTTP status code returned by your endpoint (null if connection failed) |
| isSuccess | Whether delivery was successful (2xx response) |
| attemptNumber | Which attempt this was (1 = first attempt, up to 6 with retries) |
| errorMessage | Error message if delivery failed (null on success) |
| durationMs | How long the request took in milliseconds |
| createdAt | When the delivery attempt was made |

---

## Delete Webhook Subscription

Remove your webhook subscription to stop receiving notifications.

### Endpoint

`DELETE /apps/api/webhooks`

**Headers:**
```
Authorization: Bearer {access_token}
```

### Example Request (cURL)
```bash
curl -X DELETE "https://api.wepay.com.sa/apps/api/webhooks" \
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
| --- | --- |
| Content-Type | application/json |
| X-WePay-Signature | HMAC-SHA256 signature for verification |
| X-WePay-Event | The event type (e.g., "payment.completed") |
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
| --- | --- |
| id | Unique webhook delivery ID |
| event | Event type that triggered this webhook |
| createdAt | When the webhook was created |
| data.contractId | External contract ID (e.g., "CNT-2601-00100068") |
| data.reference | Your reference from contract creation (if provided) |
| data.status | Current contract status |
| data.previousStatus | Previous contract status |
| data.amount | Contract amount in SAR |
| data.currency | Currency code (always "SAR") |
| data.paymentId | Payment ID (for payment events) |
| data.transactionId | Transaction ID (for payment events) |
| data.invoiceId | Invoice ID (for payment events) |
| data.timestamp | Event timestamp |
| data.metadata | Your metadata from contract creation (metadata1-4, reference) |

> **Note:** Some fields like `paymentId`, `transactionId`, and `invoiceId` are only present for relevant events (e.g., `payment.completed`).

---

## Verifying Webhook Signatures

To ensure webhooks are genuinely from WePay, verify the signature.

### Signature Format

The `X-WePay-Signature` header contains: `sha256={hex_signature}`

### Verification Process

1. Extract the signature from `X-WePay-Signature` header
2. Get the raw JSON body of the request
3. Compute HMAC-SHA256 of the body using your secret key
4. Compare your computed signature with the received signature

### Example Implementation (Node.js)
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

// In your webhook handler:
app.post('/webhooks/wepay', (req, res) => {
  const signature = req.headers['x-wepay-signature'];
  const payload = JSON.stringify(req.body);

  if (!verifyWebhookSignature(payload, signature, YOUR_SECRET_KEY)) {
    return res.status(401).send('Invalid signature');
  }

  // Process the webhook
  const event = req.body;
  console.log('Received event:', event.event);

  switch(event.event) {
    case 'payment.completed':
      // Handle payment completed
      break;
    case 'contract.released':
      // Handle contract released
      break;
  }

  // Always respond with 200 OK quickly
  res.status(200).send('OK');
});
```

### Example Implementation (PHP)
```php
<?php
function verifyWebhookSignature($payload, $signature, $secretKey) {
    $expectedSignature = 'sha256=' . hash_hmac('sha256', $payload, $secretKey);
    return hash_equals($expectedSignature, $signature);
}

// In your webhook handler:
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_WEPAY_SIGNATURE'] ?? '';

if (!verifyWebhookSignature($payload, $signature, YOUR_SECRET_KEY)) {
    http_response_code(401);
    exit('Invalid signature');
}

$event = json_decode($payload, true);

switch($event['event']) {
    case 'payment.completed':
        // Handle payment completed
        break;
    case 'contract.released':
        // Handle contract released
        break;
}

http_response_code(200);
echo 'OK';
?>
```

### Example Implementation (Python)
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
        # Handle payment completed
        pass
    elif event['event'] == 'contract.released':
        # Handle contract released
        pass

    return 'OK', 200
```

### Example Implementation (C#)
```csharp
using System.Security.Cryptography;
using System.Text;

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

        switch (webhookEvent.Event)
        {
            case "payment.completed":
                // Handle payment completed
                break;
            case "contract.released":
                // Handle contract released
                break;
        }

        return Ok();
    }
}
```

> **Important:** Always use constant-time comparison (`timingSafeEqual`, `hash_equals`, `compare_digest`, `FixedTimeEquals`) to prevent timing attacks.

---

## Retry Policy

If your endpoint fails to respond with a 2xx status code, WePay will retry delivery with exponential backoff:

| Attempt | Delay |
| --- | --- |
| 1st retry | 10 seconds |
| 2nd retry | 30 seconds |
| 3rd retry | 2 minutes |
| 4th retry | 10 minutes |
| 5th retry | 1 hour |

After 5 failed retries, the webhook delivery is marked as failed.

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

1. Fix your endpoint issues
2. Update the webhook URL via `POST /apps/api/webhooks`

---

## Webhook Best Practices

### 1. Respond Quickly

- Return `200 OK` as soon as you receive the webhook
- Process the event asynchronously if needed
- Timeout is 30 seconds - respond faster to avoid retries

### 2. Handle Duplicates

- Webhooks may be delivered more than once
- Use the webhook ID (`X-WePay-Webhook-Id`) to deduplicate
- Make your processing idempotent

### 3. Verify Signatures

- Always verify `X-WePay-Signature` before processing
- Reject unsigned or incorrectly signed requests
- Use constant-time comparison functions

### 4. Use HTTPS

- Your webhook URL must use HTTPS in production
- Ensure your SSL certificate is valid and not expired

### 5. Log Everything

- Log all received webhooks for debugging
- Log signature verification results
- Keep logs for troubleshooting failed deliveries

### 6. Monitor Your Endpoint

- Check delivery logs regularly via `/apps/api/webhooks/logs`
- Set up alerts for consecutive failures
- Test your endpoint periodically using `/apps/api/webhooks/test`

---

## Webhook Error Handling

### Webhook Configuration Errors

| Error | Solution |
| --- | --- |
| 400 Invalid URL | Ensure URL is valid and uses http:// or https:// |
| 401 Unauthorized | Check your access token is valid and not expired |
| 404 Not Found | No webhook subscription exists - create one first |

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
