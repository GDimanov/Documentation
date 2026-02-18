# SAGA Transaction API Documentation v1

This document provides the necessary endpoints and workflows for integrating a POS system with the Voucher Management API using the SAGA pattern.

# API DOCUMENTATION v1

[1. Authentication](#Authentication)

[2. General Utilities](#General-Utilities)

[3. SAGA Transaction Workflow](#SAGA-Transaction-Workflow)

--------------------------------------------------------------------------------
# Authentication

## Request Authentication

All API endpoints (with the exception of /login) require a valid JSON Web Token (JWT) to be included in the request headers.

1. Obtain a Token: Send a POST request to the /login endpoint with valid credentials.

2. Authorize Requests: Include the returned userToken in the header of every subsequent request.

3. Header Format:

    ◦ Key: Authorization

    ◦ Value: Bearer <your_token_here>

Example Header: Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

Note: Access to specific endpoints is restricted based on user roles (e.g. OWNER, MANAGER, CASHIER etc..). Ensure your authenticated user has the required permissions for the operation you are attempting.

Before performing any operations, the POS must authenticate to receive a JWT token.

## Log in

REQUEST METHOD: POST

* Endpoint: /login

***Reqest body of type JSON***

    {
    "userName" : "test",
    "userPass" : "testingPassword",
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `userName`   | String   | The username of the user trying to log in. | YES | "test"|
| `userPass`   | String   | The password for the user account. | YES | "test"|

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

# SAGA Information 

## Get current user active transactions 
Returns the transactions with status "STARTED"

• Method: GET

• Endpoint: /card/saga/getActiveTransaction

Response:

[
    {
        "sagaId": "4044f00e-ea65-4ca9-ab1b-f97a1cb54248",
        "sagaStatus": "STARTED",
        "errCode": "",
        "errMsg": "",
        "cardSerial": "00025",
        "transactionSummaryDTO": [
            {
                "transactionId": 14,
                "stepType": "REFUND_BUY",
                "stepStatus": "REFUSED"
  		  	    "errCode": "",
    		    "errMsg": ""
            }
        ]
    },
    {
        "sagaId": "fb1e34bc-13a7-4a8b-9753-3e45c1f98796",
        "sagaStatus": "STARTED",
        "errCode": "",
        "errMsg": "",
        "cardSerial": "00024",
        "transactionSummaryDTO": [
            {
                "transactionId": 15,
                "stepType": "SPEND",
                "stepStatus": "RESERVED"
  		        "errCode": "",
    		    "errMsg": ""
            }
        ]
    }
]


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
    "errCode": "",
    "errMsg": "",
	"cardSerial": "00024"
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

> Upon execution the buy card intent will activate and initialize the card.

Response (VARIABLE_LOAD vs FIXED_PRICE): If cardSalePricingType is FIXED_PRICE, the card has a preset value (e.g., 50.00) that must be charged. If it is VARIABLE_LOAD, the price is 0, and the card must be loaded with funds via the deposit endpoint. [User Query]

    {
    "sagaId": "cbb58b7c-d271-4763-ae14-a7741117e156",
    "sagaStatus": "STARTED",
    "stepType": "BUY_CARD",
    "stepStatus": "RESERVED",
    "errCode": "PRICE_QUOTED",
    "errMsg": "Card sell price requested.",
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

Response:

    {
    "sagaId": "cbb58b7c-d271-4763-ae14-a7741117e156",
    "sagaStatus": "STARTED",
    "stepType": "SPEND",
    "stepStatus": "RESERVED",
    "errCode": "SAGA_ACCEPTED",
    "errMsg": "",
    "duplicate": true 
    }

### C. Unified Refund
Reverses a previous SPEND, WITHDRAW, DEPOSIT, BUY

> Refunding a BUY transaction will deactivate the card.

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
    "errCode": "REFUND_EXCEEDING_AMOUNT",
    "errMsg": "Refund amount exceeding the original operation amount",
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

Response : 

	Accepted :

    {
    "sagaId": "445bf7fd-e442-49b8-af01-19153ee6a1dc",
    "sagaStatus": "STARTED",
    "stepType": "DEPOSIT",
    "stepStatus": "RESERVED",
    "errCode": "SAGA_ACCEPTED",
    "errMsg": "",
    "duplicate": false
    }

	Refused :

    {
    "sagaId": "258b319d-cbbc-4373-88f4-462433972477",
    "sagaStatus": "STARTED",
    "stepType": "DEPOSIT",
    "stepStatus": "REFUSED",
    "errCode": "CARD_USED",
    "errMsg": "Тази карта вече е била използвана!\nПовторното и използване е забранено!",
    "duplicate": false
    }

### E. Refund by saga UUID 

By supplying the UUID you can execute refund on all refundable operations within the transaction.
If no amount is send than the full amount will be refunded if possible!

• Method: POST

• Endpoint: /card/saga/refundSaga

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `uuid`   | String   |SAGA unique id | YES | "ff6d5270-3fb1-4deb-9110-864fda98e0d3"|
| `amount`   | double   |Amount to deposit in card. | NO | 6.60|


Response :
 
    [
    {
    "sagaId": "8f71518d-7790-4332-817e-a47434b986c3",
    "sagaStatus": "STARTED",
    "stepType": "REFUND_BUY",
    "stepStatus": "RESERVED",
    "errCode": "SAGA_ACCEPTED",
    "errMsg": "",
    "duplicate": false
    },
    {
    "sagaId": "8f71518d-7790-4332-817e-a47434b986c3",
    "sagaStatus": "STARTED",
    "stepType": "REFUND_DEPOSIT",
    "stepStatus": "RESERVED",
    "errCode": "SAGA_ACCEPTED",
    "errMsg": "",
    "duplicate": false
    }
    ]
--------------------------------------------------------------------------------
## Step 3: Close Transaction
Executes or cancels all reserved steps in the saga.

• Method: GET

• Endpoint: /card/saga/closeTransaction

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `isCanceled`   | int   |Default value 0. Set to 1 to cancel all steps! | No | 0 |

Response:

 All steps pass

	{
    "sagaId": "4044f00e-ea65-4ca9-ab1b-f97a1cb54248",
    "sagaStatus": "COMPLETED",
    "errCode": "",
    "errMsg": "",
    "cardSerial": "00025",
    "transactionSummaryDTO": [
        {
            "transactionId": 0,
            "stepType": "REFUND_BUY",
            "stepStatus": "REFUSED"
  		    "errCode": "",
    		"errMsg": ""
        }
    ]
	}

 Fail step

    {
   	 "sagaId": "7deeb2b2-e9ef-4411-ac8e-f97cec94f848",
    "sagaStatus": "COMPLETED",
    "errCode": "",
    "errMsg": "",
    "cardSerial": "00027",
    "transactionSummaryDTO": [
    {
   		 "transactionId": 0,
   		 "stepType": "REFUND_SPEND",
   		 "stepStatus": "REFUSED",
   		 "errCode": "CARD_USED",
   		 "errMsg": "Тази карта вече е била използвана!\nПовторното и използване е забранено!"
    }
    ]
    }


--------------------------------------------------------------------------------
# SAGA Reference
 
## 1. Saga Status (SagaStatus)
These codes represent the high-level state of the entire multi-step transaction (Saga).

| Code     | Description  																					  |
| :---     | :---  																							  |
|STARTED   | The transaction has been initiated and is currently active for adding steps.					  |
|COMPLETED | The transaction was successfully closed and all reserved steps have been committed to the ledger.|
|CANCELLED | The transaction was manually aborted by the user/POS before finalization.						  |
|REJECTED  | The transaction failed to start or was aborted due to business logic or technical errors. 		  |
|EXPIRED   |The 15-minute validity window passed without activity, rendering the saga inactive.				  |

* EXPIRED validation is turned off. Saga lifecycle will end only if close transaction is called on them!

--------------------------------------------------------------------------------
## 2. Saga Step Types (SagaStepType)
These codes identify the specific financial or administrative operation being performed within a step.

| Code  		  | Description   												 |
| :---     | :---  																 |
| BUY_CARD  	  | Requesting the sale price and intent to activate a new card. |
| SPEND	    	  | A standard debit operation to pay for goods or services. 	 |
| DEPOSIT  		  | A credit operation to increase the card's current balance.	 |
| WITHDRAW    	  | A debit operation to withdraw funds from the card balance. 	 |
| REFUND_SPEND    | Specifically reverses a previous SPEND transaction (Credit). |
| REFUND_WITHDRAW | Reverses a previous WITHDRAW (acts as a Deposit/Credit).  	 |
| REFUND_DEPOSIT  | Reverses a previous DEPOSIT (acts as a Withdraw/Debit). 	 |
| REFUND_BUY      | Reverse the buy intent and deactivates the card.		     |


--------------------------------------------------------------------------------
## 3. Saga Step Status (SagaStepStatus)
These codes track the lifecycle of an individual step within an active or finalized Saga.

| Code      | Description   |
| :---      | :---  		|
| RESERVED  | The step has passed initial validation against the shadow balance and is pending final execution. |
| EXECUTED  | The step has been successfully processed and recorded in the permanent transaction ledger (card_transaction). |
| REFUSED | The step was rejected during the reservation phase (e.g., due to insufficient funds or daily limits). |
| CANCELLED | The step was part of a saga that was manually aborted or reached its 15-minute TTL without being closed. |

--------------------------------------------------------------------------------
## 4. Financial Result Codes (SagaResultCode)
These codes are returned in the reasonCode field of API responses to provide granular feedback.

• SAGA_ACTIVE: Attempted to start a new transaction while one is already in progress for the card.

• SAGA_ACCEPTED: The operation was successfully reserved and the shadow balance updated.

• PRICE_QUOTED: Returned after a BUY_CARD intent to provide card sale information.

• SAGA_NOT_ACTIVE: Attempted to close or add a step to a card with no active saga.

• INTERNAL_ERROR: An unexpected server-side error occurred during processing.