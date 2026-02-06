# SAGA Transaction API Documentation v1

This document provides the necessary endpoints and workflows for integrating a POS system with the Voucher Management API using the SAGA pattern.

# API DOCUMENTATION v1

[1. Authentication](#Authentication)

[2. General Utilities](#General-Utilities)

[3. SAGA Transaction Workflow](#SAGA-Transaction-Workflow)

[3.1 SAGA Transaction Workflow](#SAGA-Transaction-Workflow)

--------------------------------------------------------------------------------
# Authentication
Before performing any operations, the POS must authenticate to receive a JWT token.
Log in
REQUEST METHOD: POST
ENDPOINT: /login
Request Body (JSON): | Parameter | Type | Description | Required | Example | | :--- | :--- | :--- | :--- | :--- | | userName | String | Username for the account. | YES | "test" | | userPass | String | Password for the account. | YES | "testingPassword" | | subObjId | Long | ID of the branch/sub-object. Defaults to user's default if omitted. | NO | 10 |
Response:
{
    "userToken": "Bearer eyJhbGciOiJIUzI1NiJ9...",
    "userRoles": ["CASHIER"],
    "userName": "Eltrade",
    "expiresAt": "1758976767043"
}

--------------------------------------------------------------------------------
# General Utilities
These methods assist in retrieving card states or appending data to finalized transactions.

## Get Card Information
Retrieve current balance and status by serial number.

• Method: GET

• Endpoint: /card/getCardBySerial

• Params: serial (String)

Response : 

    {
    "transactionId": 0,
    "serialNumber": "00021",
    "startDate": "2026-02-06",
    "endDate": "2026-08-06",
    "validMonths": 6,
    "startAmount": 0.00,
    "amountBeforeOperation": 20.00,
    "currentAmount": 20.00,
    "cashbackAmount": 0,
    "limitPerDay": 0.00,
    "initiated": true,
    "groupName": "Test",
    "forbiddenObjects": [],
    "active": true,
    "used": false,
    "cashBack": false,
    "vname": "hmmm",
    "rechargeable": false,
    "reusable": false
    }

## Add Bill Data
Attach detailed bill items or text to a finalized transaction.
• Method: POST

• Endpoint: /card/addBillData

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `transactionId`   | Integer   |Amount to spend from card. | YES | 52| 
| `billAsText`   | String   |Text representation of the bill | NO | "Bill text here"|

**Request body of type JSON**

----------

**!!! SAME JSON BODY TYPE AND PARAMETERS AS IN SPEND CARD METHOD!!!**

**Using JSON Body is NOT Require and can be skipped**

**Bill text and bill data can be added separetely**

--------------------------------------------------------------------------------
# SAGA Transaction Workflow

## Step 1: Start Transaction
Initiates a saga session. Sagas expire automatically after 15 minutes.

• Method: GET

• Endpoint: /card/saga/startTransaction

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|

Response:

    {
    "sagaId": "7d2f820e-da10-4edb-bb43-0be9dde56d80",
    "sagaStatus": "STARTED",
    "reasonCode": "",
    "message": ""
    }

--------------------------------------------------------------------------------
## Step 2: Add Transaction Steps
Operations added to a saga are idempotent. Each consecutive attempt with the same parameters will return the same result, but the duplicate field will be true. [User Query]

### A. Buy Card Intent
Requests the sale price of a card.

• Method: GET

• Endpoint: /card/saga/buy

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|

Response (VARIABLE_LOAD vs FIXED_PRICE): If cardSalePricingType is FIXED_PRICE, the card has a preset value (e.g., 50.00) that must be charged. If it is VARIABLE_LOAD, the price is 0, and the card must be loaded with funds via the deposit endpoint. [User Query]

    {
    "sagaId": "cbb58b7c-d271-4763-ae14-a7741117e156",
    "sagaStatus": "STARTED",
    "stepType": "BUY_CARD",
    "stepStatus": "RESERVED",
    "reasonCode": "PRICE_QUOTED",
    "message": "Card sell price requested.",
    "duplicate": false,
    "cardSellInfoDTO": {
    "cardSalePricingType": "FIXED_PRICE",
    "fixedPrice": 50.00,
    "minLoadAmount": null,
    "maxLoadAmount": null,
    "activationFee": null
    }
    }

### B. Spend Sum
Reserves a payment from the card.

• Method: POST

• Endpoint: /card/saga/spend

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `amount`   | double   |Amount to spend from card. | YES | 6.60| 
| `billAsText`   | String   |Text representation of the bill | NO | "Bill text here"|

Idempotent Response:

    {
    "sagaId": "cbb58b7c-d271-4763-ae14-a7741117e156",
    "sagaStatus": "STARTED",
    "stepType": "SPEND",
    "stepStatus": "RESERVED",
    "reasonCode": "SAGA_ACCEPTED",
    "message": "",
    "duplicate": true 
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `unp`   | String   | Informative field UNP bill Id. | YES | "UNP1223AE11231231231"|
| `billNumber`   | String   | Informative field bill system Id. | YES | "123551233123"|
| `billTransactionId`   | String   | Informative field bill system transaction Id. | YES | "1112233"|
| `billPaymentType`   | String   | Informative field bill primary payment type. | YES | "Card"|
| `objectId`   | int   | Id of the branch processing the transaction. | YES | 100|
| `transactionDate`   | String   | Informative date of bill.<br>No format limitations<br> | YES | "2025-17-10"|
| `itemName`   | String    |Informative name of the item included in the bill | YES | "Домат"|
| `itemGroup`   | String   | Informative name of the group item is part of. | YES | "Зеленчуци"|
| `itemBarcode`   | String   | Informative item barcode | YES | ""|
| `itemPrice`   | double   | Informative price of item. | YES | 4.80|
| `itemQuant`   | double   | Informative quont of item in sale. | YES | 2.30|
| `itemDiscount`   | double   | Informative discount from price. | YES | 0|

**RESPONSE**

	{
    "cardResponseDTO": {
        "transactionId": 54,
        "serialNumber": "1",
        "startDate": "2025-01-06",
        "endDate": "2025-07-06",
        "validMonths": 6,
        "startAmount": 50.00,
        "amountBeforeOperation": 46.00,
        "currentAmount": 41.00,
        "cashbackAmount": 0,
        "limitPerDay": 0.00,
	    "initiated": true,
        "groupName": "Test",
        "forbiddenObjects": [],
        "active": true,
        "cashBack": false,
        "reusable": true,
        "vname": "hmmm",
        "rechargeable": true,
        "used": false
    },
    "billData": [
        {
            "type": "BillData",
            "status": "Success",
            "errMsg": "No Bill data supplied"
        },
        {
            "type": "BillText",
            "status": "Success",
            "errMsg": "No errors"
        }
    ]
	}

### C. Unified Refund
Reverses a previous SPEND, WITHDRAW, or DEPOSIT.

• Method: POST

• Endpoint: /card/saga/refund

• Params: serial, referralTransId, amount

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `referralTransId`   | Long   |Id of the Spend transaction. | YES | 1234567|
| `amount`   | double   |Amount to deposit in card. | YES | 6.60|

Refused Step Example:

    {
    "sagaId": "7d2f820e-da10-4edb-bb43-0be9dde56d80",
    "sagaStatus": "STARTED",
    "stepType": "REFUND_SPEND",
    "stepStatus": "REFUSED",
    "reasonCode": "REFUND_EXCEEDING_AMOUNT",
    "message": "Refund amount exceeding the original operation amount",
    "duplicate": false
    }

### D. Deposit / Withdraw
• Deposit: Increases card balance.

• Withdraw: Decreases card balance.

• Method: POST

• Endpoint: /card/saga/deposit 

• Endpoint: /card/saga/withdraw 

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `referralTransId`   | Long   |Id of the Spend transaction. | YES | 1234567|
| `amount`   | double   |Amount to deposit in card. | YES | 6.60|
• Note: If referralTransId > 0, these route to refund logic automatically.

--------------------------------------------------------------------------------
## Step 3: Close Transaction
Executes or cancels all reserved steps in the saga.
• Method: GET
• Endpoint: /card/saga/closeTransaction
• Params:
**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|

Final Response:

    {
    "sagaId": "7d2f820e-da10-4edb-bb43-0be9dde56d80",
    "sagaStatus": "COMPLETED",
    "reasonCode": "",
    "message": "",
    "transactionSummaryDTO": [
    {
    "transactionId": 0,
    "stepType": "REFUND_SPEND",
    "stepStatus": "REFUSED"
    }
    ]
    }