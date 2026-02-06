SAGA Transaction API Documentation v1
This document provides the necessary endpoints and workflows for integrating a POS system with the Voucher Management API using the SAGA pattern.

--------------------------------------------------------------------------------
1. Authentication
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
2. General Utilities
These methods assist in retrieving card states or appending data to finalized transactions.
Get Card Information
Retrieve current balance and status by serial number.
• Method: GET
• Endpoint: /card/getCardBySerial
• Params: serial (String)
Add Bill Data
Attach detailed bill items or text to a finalized transaction.
• Method: POST
• Endpoint: /card/addBillData
• Params: serial, transactionId, init (0 or 1).
• Body: SpendCardRequestDTO (JSON)

--------------------------------------------------------------------------------
3. SAGA Transaction Workflow
Step 1: Start Transaction
Initiates a saga session. Sagas expire automatically after 15 minutes.
• Method: GET
• Endpoint: /card/saga/startTransaction
• Params: cardId (Serial Number)
Response:
{
    "sagaId": "7d2f820e-da10-4edb-bb43-0be9dde56d80",
    "sagaStatus": "STARTED",
    "reasonCode": "",
    "message": ""
}

--------------------------------------------------------------------------------
Step 2: Add Transaction Steps
Operations added to a saga are idempotent. Each consecutive attempt with the same parameters will return the same result, but the duplicate field will be true. [User Query]
A. Buy Card Intent
Requests the sale price of a card.
• Method: GET
• Endpoint: /card/saga/buy
• Params: sagaId, serial
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
B. Spend Sum
Reserves a payment from the card.
• Method: POST
• Endpoint: /card/saga/spend
• Params: sagaId, serial, amount
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
C. Unified Refund
Reverses a previous SPEND, WITHDRAW, or DEPOSIT.
• Method: POST
• Endpoint: /card/saga/refund
• Params: serial, referralTransId, amount
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
D. Deposit / Withdraw
• Deposit: Increases card balance.
• Withdraw: Decreases card balance.
• Note: If referralTransId > 0, these route to refund logic automatically.

--------------------------------------------------------------------------------
Step 3: Close Transaction
Executes or cancels all reserved steps in the saga.
• Method: GET
• Endpoint: /card/saga/closeTransaction
• Params:
    ◦ cardId: Serial Number
    ◦ isCanceled: 1 to cancel, 0 to execute.
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