# ELTRADE VOUCHER MANAGEMENT API

API to manage the workflow of using custom vouchers in different trading shops.


# API DOCUMENTATION v1

[1. Authentication](#log-in)

[2. Billing](#billing)

[3. User managing](#user)

[4. Company managing](#company)

[5. Branch managing](#subObject)

[6. Groups](#groups)

[7. Card](#card)

[8. Report](#report)

[9. Error handling](#error-handling)

## DATA FORMATS 

    **Strings**
    
    All strings should have UTF-8 encoding.
    
    **DATE**
    
    All Datesmust be of type `YYYY-MM-DD` 
    
    *Example: "2025-01-01"*

----------
## Log in
**REQUEST METHOD : `POST`**/login

***Reqest body of type JSON***

    {
    "userName" : "test",
    "userPass" : "testingPassword",
	"subObjId" : 10
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `userName`   | String   | The username of the user trying to log in. | YES | "test"|
| `userPass`   | String   | The password for the user account. | YES | "test"|
| `subObjId`   | Long   | (Optional) The ID of the sub-object or branch the user<br>is trying to log in to. If left blank or not provided,<br>the system will use the user's default sub-object.<br> | NO | 2|

**RESPONSE**

    {
    "userToken": "Bearer eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwidXNlciI6IkVsdHJhZGUiLCJhZG1pbiI6dHJ1ZSwibWFuYWdlciI6dHJ1ZSwiY2FzaGllciI6dHJ1ZSwib2JzZXJ2ZXIiOnRydWUsImV4cCI6MTcyMTIwNDc1M30.hAvIOc_YiceibHiRt5r2ARdz06yX2HVE_EJl663GV54",
    "userRoles": [
    "ADMIN",
    "MANAGER",
    "OWNER",
    "CASHIER",
    "OBSERVER"
    ],
    "userName": "Eltrade"
    }

----------

# Billing 

## **Get data for registered Cash points**

**REQUEST METHOD : `GET`**/billing/getValidityData

- **Must include in header Authentication : Bearer ... with 'ADMIN','OWNER','MANAGER' ROLE**

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `companyEIK`   | Integer   | Compay EIK | YES | 8202626260|
| `userName`   | String   | If supplied will only return the results of this cash point | NO | "MyCashPoint"|

**RESPONSE**

	[
    {
        "cashModule": "Georgito",
        "cashModuleName": "Моя касиер",
        "validUntilDate": "2025-11-01"
    }
	]

## Set new validity date**

**REQUEST METHOD : `POST`**/billing/setValidity

- **Must include in header Authentication : Bearer ... with 'ADMIN'

***Request body of type JSON***

	{
    "cashModule" : "Georgito",
    "validUntilDate" : "2025-11-01"
	}

**RESPONSE**

    {
        "cashModule": "Georgito",
        "cashModuleName": "Моя касиер",
        "validUntilDate": "2025-11-01"
    }

----------

# USER

## **Create Owner**

**REQUEST METHOD : `POST`**/user/createOwner

- **Must include in header Authentication : Bearer ... with 'ADMIN' ROLE**

***Request body of type JSON***

    {
    "name" : "myUserName",
	"displayName," : "Ivan",
    "password" : "myStrongPassword",
    "accEIK" : "832076302",
    "accBulstat" : "BG832076302",
    "accName" : "Eltrade OOD",
    "accCity" : "Sofiq",
    "accAddress" : "Sofiq G.Delchev_102",
    "accPhone" : "056/813659",
    "accEmail" : "eltrade@eltrade.com",
    "accActiveUntil" : "2025-01-01"
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | The username of the user. | YES | "test"|
| `userPass`   | String   | The password for the user account. | YES | "test"|
| `accEIK`   | String   | Unique identifier of the company. | YES | "832076302"|
| `accBulstat`   | String   | Company bulstat identifier.<br>Can be left blank if none.<br> | YES | "BG832076302"|
| `accName`   | String   | The name of the company.<br>Can be left blank if none.<br> | YES | "Eltrade OOD"|
| `accCity`   | String   | Company main office city.<br>Can be left blank if none.<br> | YES | "Sofiq"|
| `accAddress`   | String   | Company address.<br>Can be left blank if none.<br> | YES | "Sofiq G.Delchev_102"|
| `accPhone`   | String   | Company contact phone.<br>Can be left blank if none.<br> | YES | "056/813659"|
| `accEmail`   | String   | Company contact e-mail.<br>Can be left blank if none.<br> | YES | "eltrade@eltrade.com"|
| `accEmail`   | String   | Company valid date.<br>Date format must be YYYY-MM-DD<br> | YES | "2025-01-01"|

    Important Notes
    
    - Company Creation: When an owner is created, a company is automatically created and assigned to the owner.
    
    - Central SubObject: A central branch is also created and linked to the company.
    
    - Date Format: The accActiveUntil parameter must use the YYYY-MM-DD format.

**RESPONSE**

    {
    "name" : "myUserName",
    "status" : "Created"
    }

## **Create Manager**

**REQUEST METHOD : `POST`**/user/createManagerAccount

- **Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

***Reqest body of type JSON***

    {
    "name" : "manager2",
    "password" : "myStrongPassword"
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | The username of the user. | YES | "manager2"|
| `password`   | String   | The password for the user account. | YES | "myStrongPassword"|

Created user will be automatically assigned to the Company of the user who's creting him!

**RESPONSE**

    {
    "name" : "manager2",
    "status" : "Created"
    }

## **Create Cashier**

**REQUEST METHOD : `POST`**/user/createCashierAccount

- **Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

***Reqest body of type JSON***

    {
    "name" : "cashier",
    "password" : "myStrongPassword"
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | The username of the user. | YES | "cashier"|
| `password`   | String   | The password for the user account. | YES | "myStrongPassword"|

Created user will be automatically assigned to the Company of the user who's creting him!

**RESPONSE**

    {
    "name" : "cashier",
    "status" : "Created"
    }

## **Create Observer**

**REQUEST METHOD : `POST`**/user/createObserverAccount

- **Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

***Reqest body of type JSON***

    {
    "name" : "observer",
    "password" : "myStrongPassword"
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | The username of the user. | YES | "observer"|
| `password`   | String   | The password for the user account. | YES | "myStrongPassword"|

Created user will be automatically assigned to the Company of the user who's creting him!

**RESPONSE**

    {
    "name" : "observer",
    "status" : "Created"
    }

## **Delete User**

**REQUEST METHOD : `POST`**/user/deleteUser

- **Must include in header Authentication : Bearer ... with 'ADMIN','OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | Username to deactivate. | YES | "observer"|


**RESPONSE**

    {
    "name" : "observer",
    "status" : "Deleted"
    }

## **Activate deleted User**

**REQUEST METHOD : `POST`**/user/activateUser


- **Must include in header Authentication : Bearer ... with 'ADMIN','OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | Username to deactivate. | YES | "observer"|


**RESPONSE**

    {
    "name" : "observer",
    "status" : "Restored"
    }

## **List all users**

**REQUEST METHOD : `GET`**/user/viewAllOwners

- **Must include in header Authentication : Bearer ... with 'ADMIN'ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|


**RESPONSE**

    [
    {
		"id": 29,
        "name": "Eltrade",
		"displayName," : "Ivan",
        "defObjId": 1,
        "companyId": 1,
        "userRoles": [
            "ADMIN",
            "MANAGER",
            "OWNER",
            "CASHIER",
            "OBSERVER"
        ],
        "forbiddenObjList": [],
        "active": true
    },
    {
		"id": 28,
        "name": "owner",
		"displayName," : "Petar",
        "defObjId": 4,
        "companyId": 2,
        "userRoles": [
            "MANAGER",
            "OWNER",
            "CASHIER",
            "OBSERVER"
        ],
        "forbiddenObjList": [],
        "active": true
    },
    {
		"id": 27,
        "name": "owner1",
		"displayName," : "Ivan",
        "defObjId": 3,
        "companyId": 3,
        "userRoles": [
            "MANAGER",
            "OWNER",
            "CASHIER",
            "OBSERVER"
        ],
        "forbiddenObjList": [],
        "active": true
    }
]

## *Get company owner**

**REQUEST METHOD : `GET`**/user/getCompanyOwner

- **Must include in header Authentication : Bearer ... with 'ADMIN'ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `companyEIK`   | int   | Company EIK umber | YES | 0|


**RESPONSE**

	{
    "id": 2,
    "name": "GeorgiDD",
    "displayName": "",
    "defObjId": 2,
    "companyId": 2,
    "userRoles": [
        "OWNER"
    ],
    "forbiddenObjList": [],
    "active": true
	}

## **List all users by Account owner**

**REQUEST METHOD : `GET`**/user/viewAllUsersByAcc

- **Must include in header Authentication : Bearer ... with 'ADMIN','MANAGER','OWNER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|

**RESPONSE**

    [
    {
	"id": 29,
    "name": "1",
	"displayName," : "Ivan",
    "defObjId": 1,
	"companyId": 2,
    "userRoles": [
    "MANAGER",
    "OWNER",
    "CASHIER",
    "OBSERVER"
    ],
    "forbiddenObjList": [],
    "active": true
    },
    {
	"id": 28,
    "name": "manager2",
	"displayName," : "Ivan",
    "defObjId": 1,
	"companyId": 2,
    "userRoles": [
    "MANAGER"
    ],
    "forbiddenObjList": [
    1,
    2
    ],
    "active": true
    },
    {
	"id": 27,
    "name": "manager1",
	"displayName," : "Ivan",
    "defObjId": 1,
	"companyId": 2,
    "userRoles": [
    "MANAGER"
    ],
    "forbiddenObjList": [],
    "active": true
    },
    {
	"id": 26,
    "name": "manager3",
	"displayName," : "Dragan",
    "defObjId": 1,
	"companyId": 2,
    "userRoles": [
    "MANAGER"
    ],
    "forbiddenObjList": [],
    "active": true
    }
    ]

## **Get count of all users by Account owner**

**REQUEST METHOD : `GET`**/user/viewAllUsersByAcc/count

- **Must include in header Authentication : Bearer ... with 'ADMIN','MANAGER','OWNER' ROLE**

**RESPONSE**

    - will return the total user count for the company of the user : 10 
    - If the user is 'ADMIN' will return the total user count ignoring the company : 24


## **Get count of all owners**

**REQUEST METHOD : `GET`**/user/viewAllOwners/count

- **Must include in header Authentication : Bearer ... with 'ADMIN' ROLE**

**RESPONSE**

    - will return the total owners count : 3

## **Change user data**


**REQUEST METHOD : `POST`**/user/changeUserData


- **Must include in header Authentication : Bearer ... with 'ADMIN','MANAGER','OWNER' ROLE**


***Reqest body of type JSON***

    {
	"userToEdit" : "existingUserName",
    "newName" : "myNewCashier",
	""displayName," : "newDisplayName",
    "newPassword" : "myNewStrongPassword",
	"isAdmin" : 0,
	"isManager" : 0,
	"isCashier" : 1,
	"isObserver" : 0,
	"isAccountOwner" : 0
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `userToEdit`   | String   | User name of the edited user | YES | "Observer"|
| `newName`   | String   | New name to set for the selected user | YES | "ObserverNew"|
| `newPassword`   | String   | New password to set for the selected user.<br>If "newPassword" field is left blank password won't be changed  <br> | YES | "ObserverNew"|
| `isAdmin`   | int   | Set user to Admin<br>integer values 0 - False, 1 - True<br> | YES | 0|
| `isManager`   | int   | Set user to Manager<br>integer values 0 - False, 1 - True<br> | YES | 0|
| `isCashier`   | int   | Set user to Cashier<br>integer values 0 - False, 1 - True<br> | YES | 1|
| `isObserver`   | int   | Set user to Observer<br>integer values 0 - False, 1 - True<br> | YES | 0|
| `isAccountOwner`   | int   | Set user to Owner<br>integer values 0 - False, 1 - True<br> | YES | 0|

**RESPONSE**

    {
    "name" : "observer",
    "status" : "Updated"
    }

## **Change user additional settings data**

**REQUEST METHOD : `POST`**/user/changeUserSettings

- **Must include in header Authentication : Bearer ... with 'ADMIN','MANAGER','OWNER' ROLE**

***Reqest body of type JSON***

    {
	"userToEdit" : "userToEdit",
    "newSubObjId" : 2,
    "forbiddenObjIdList" : [1, 2]
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `userToEdit`   | String   | The username of the user to edit. | YES | "observer"|
| `newSubObjId`   | long   | Set new default branch for this user. | YES | 12345|
| `forbiddenObjIdList`   | List<long>   | A list of IDs representing the branches that should be marked as forbidden for the card. | YES | 101,102,103|

**RESPONSE**

    {
    "status" : "Updated"
    }

## **Resets the password of the selected owner**

**REQUEST METHOD : `POST`**/user/resetOwnerPass

- **Must include in header Authentication : Bearer ... with 'ADMIN' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`     | String | User name of the account owner | YES | MyComanyOnwer@mail.bg|

**RESPONSE**

	{
    "name": "MyComanyOnwer@mail.bg",
    "newPass": "DW]G9903tZk"
	}

----------
# Company

## **Edit company data**
**REQUEST METHOD : `POST`**/company/editMyCompany
- **Must include in header Authentication : Bearer ... with 'OWNER'ROLE**

***Request body of type JSON***

    {
		"accId": 1,
        "accEIK": 832076301,
        "accBulstat": "BG832076301",
        "accName": "Eltrade1 OOD",
        "accCity": "Botevgrad",
        "accAddress": "Sofiq G.Delchev_102",
        "accPhone": "8320760",
        "accEmail": "eltrade1@eltrade.com",
		"voucherDefValidMonths": 6
    }

***Request body parameters***

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `accId`   | long   | Id of the company to edit. | YES | 12345678|
| `accEIK`   | String   | Set new EIK. | YES | "12345"|
| `accBulstat`   | String   | Set new bulstat. | YES | "BG12345"|
| `accName`   | String   | Set new name. | YES | "new name"|
| `accCity`   | String   | Set new registration city. | YES | "Varna"|
| `accAddress`   | String   | Set new address. | YES | "new address here"|
| `accPhone`   | String   | Set new phone. | YES | "092/112255"|
| `voucherDefValidMonths`   | int   | Default validity of the vouchers/cards this company creates. | YES | 6|

**RESPONSE**

    {
	"accId": 1,
    "accEIK": 832076301,
    "accBulstat": "BG832076301",
    "accName": "Eltrade1 OOD",
    "accCity": "Botevgrad",
    "accAddress": "Sofiq G.Delchev_102",
    "accPhone": "8320760",
    "accEmail": "eltrade1@eltrade.com",
    "accRegisterDate": "2024-07-22T15:15:47.067+00:00",
    "accActiveUntil": "2025-01-01",
	"voucherDefValidMonths": 6
    }

## **List all companies**

**REQUEST METHOD : `GET`**/company/ListAll

- **Must include in header Authentication : Bearer ... with 'ADMIN'ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|
| `companyEIK`   | int   | Filter buy comapny EIK | NO | 11223344|
| `companyName`   | String   | Filter by company name | NO | Eltrade|
| `companyEmail`   | String   | Filter by company e-mail | NO | eltrade@eltrade.com|
| `companyCity`   | String   | Filter by company city | NO | Sofia|
| `companyPhone`   | String   | Filter by company phone | NO | 818209|
| `activeUntilDate`   | Date   | Filter will show companys that are active until the exact selected date | NO | 2025-01-01|
| `fromDate`   | Date   | Filter will show companys between fromDate and toDate | NO | 2024-01-01|
| `toDate`   | Date   | Filter will show companys between fromDate and toDate | NO | 2025-01-01|

**RESPONSE**

    [
    {
	"accId": 1,
    "accEIK": 832076302,
    "accBulstat": "BG832076302",
    "accName": "Eltrade OOD",
    "accCity": "Botevgrad",
    "accAddress": "Sofiq G.Delchev_102",
    "accPhone": "8320760",
    "accEmail": "eltrade@eltrade.com",
    "accRegisterDate": "2024-07-17T12:03:18.242+00:00",
    "accActiveUntil": "2025-01-01",
	"voucherDefValidMonths": 6
    },
    {
	"accId": 2,
    "accEIK": 832076301,
    "accBulstat": "BG832076301",
    "accName": "Eltrade1 OOD",
    "accCity": "Botevgrad",
    "accAddress": "Sofiq G.Delchev_102",
    "accPhone": "8320760",
    "accEmail": "eltrade1@eltrade.com",
    "accRegisterDate": "2024-07-22T15:15:47.067+00:00",
    "accActiveUntil": "2025-01-01",
	"voucherDefValidMonths": 6
    }
    ]

## **Count all companies**

**REQUEST METHOD : `GET`**/company/count

- **Must include in header Authentication : Bearer ... with 'ADMIN'ROLE**

**RESPONSE**

	{
    "companyCount": 6
	}

## **Get company data for current user**
**REQUEST METHOD : `GET`**/company/getMyCompany

- **Must include in header Authentication : Bearer ... with 'OWNER', 'MANAGER' ROLE**

**RESPONSE**

    {
	"accId": 1,
    "accEIK": 832076302,
    "accBulstat": "BG832076302",
    "accName": "Eltrade OOD",
    "accCity": "Botevgrad",
    "accAddress": "Sofiq G.Delchev_102",
    "accPhone": "8320760",
    "accEmail": "eltrade@eltrade.com",
    "accRegisterDate": "2024-07-17T12:03:18.242+00:00",
    "accActiveUntil": "2025-01-01",
	"voucherDefValidMonths": 6
    }

## **Update compani activity period**

**REQUEST METHOD : `GET`**/company/updatePeriod

- **Must include in header Authentication : Bearer ... with 'ADMIN'ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `date`   | Date   | New date to set as Active untill for the company | YES | 2025-03-01|
| `companyEIK`   | int   | Comapny EIK id | YES | 832076302| 

**RESPONSE**

	{
    "newDate": "2025-03-01"
	}

----------
# SubObject

## **Create SubObject**

**REQUEST METHOD : `POST`**/subObject/create

- **Must include in header Authentication : Bearer ... with 'OWNER'ROLE**

***Reqest body of type JSON***

    {
        "name": "ELTRADE",
        "city": "Sofia",
        "address": "G.Delchev 102",
        "description": "Botevgrad",
        "isActive": 1
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | Branch name. | YES | "Eltrade"|
| `city`   | String   | Branch city. | YES | "Varna"|
| `address`   | String   | Branch address. | YES | "Varna"|
| `description`   | String   | Branch description. | YES | "Regional office"|
| `isActive`   | int   | Branch is active.<br>integer values 0 - False, 1 - True<br>. | YES | 1|

**RESPONSE**

    {
    "objectName" : "ELTRADE",
    "status" : "Created"
    }

## **Edit subobject status**

**REQUEST METHOD : `GET`**/subObject/changeObjectStatus/{objectId}/{status}

**Request path variable params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `objectId`   | long   | Branch id to edit. | YES | "Eltrade"|
| `status`   | boolean   | Accepts boolean true/false. | YES | "true"|

- **Must include in header Authentication : Bearer ... with 'OWNER'ROLE**

**RESPONSE**

    {
    "objectName" : "ELTRADE",
    "status" : "Updated"
    }

## **Edit subObject**

**REQUEST METHOD : `POST`**/subObject/changeData/{objectId}

**Request path variable params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `objectId`   | long   | Branch id to edit. | YES | "Eltrade"|

- **Must include in header Authentication : Bearer ... with 'OWNER'ROLE**

***Reqest body of type JSON***

    {
        "name": "NEW NAME",
        "city": "New City",
        "address": "New Address",
        "description": "My new description"
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `name`   | String   | Branch new name. | YES | "Eltrade"|
| `city`   | String   | Branch new city. | YES | "Varna"|
| `address`   | String   | Branch new address. | YES | "Varna"|
| `description`   | String   | Branch new description. | YES | "Regional office"|
 
## **Get all subobjects by Account Owner**

**REQUEST METHOD : `GET`**/subObject/getAllByAcc

- **Must include in header Authentication : Bearer ... with 'OWNER'ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|

**RESPONSE**

    {
		"objId": 12345,
        "name": "NEW NAME",
        "city": "New City",
        "address": "New Address",
        "description": "My new description",
		"isActive": true,
    }

## **Get count of all Subobjects by Account owner**

**REQUEST METHOD : `GET`**/subObject/getAllByAcc/count

- **Must include in header Authentication : Bearer ... with 'OWNER' ROLE**

**RESPONSE**

    - will return the total Subobject count for the company of the user : 10 
    - If the user is 'ADMIN' will return the total user count ignoring the company : 24

----------
# Groups

## **Create voucher group**


**REQUEST METHOD : `POST`**/voucherGroup/create

- **Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

***Reqest body of type JSON***

	{
       "groupName": "BirthDay",
	   "groupComment": "Some text here"
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `groupName`   | String   | Name of the group. | YES | "BirthDay"|

**RESPONSE**

    {
		"status": "CREATED"
    }

## **Create voucher group**

**REQUEST METHOD : `GET`**/voucherGroup/getMyGroups

- **Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**RESPONSE**
    
[
    {
        "groupId": 1,
        "name": "Test",
        "comment": "Тестова Група!",
        "cardsCount": 21
    },
    {
        "groupId": 2,
        "name": "Честит Рожден Ден",
        "comment": "Най-добрата Група",
        "cardsCount": 0
    },
    {
        "groupId": 3,
        "name": "New",
        "comment": "New group here !",
        "cardsCount": 1
    }
]

# CARD

**REQUEST METHOD : `Post`**/card/createCard

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

It is not necessary to write "startAmount" (default 0) and "cardName" (default cardName = " ").

    {
        "serialNumber": "111222333111",
        "startDate": "2024-10-30",
        "validMonths": 3,
        "startAmount": 500,
        "limitPerDay": 0.0,
        "active" : 1,
        "rechargeable": 1,
        "reusable": 1,
        "cashBack": 0,
        "groupName": "Test",
        "cardName" : "CardA",
        "forbiddenObjects": []
    }

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serialNumber`   | String   | Name of the group. | YES | "111222333111"|
| `startDate`   | String   | Active from this date.<br>Date format must be YYYY-MM-DD<br> | YES | "2024-10-30"|
| `validMonths`   | int   | Valid will set the end date by adding this value to the startDate.<br>Set 0 to use company default.<br> | YES | 6|
| `startAmount`   | Double   | Amoun to initialy load in card. | NO | 100|
| `limitPerDay`   | Double   | Sets limit avaialble to spend each day. | YES | 100|
| `active`   | int   | Set if card is active.<br>integer values 0 - False, 1 - True<br> | YES | 1|
| `rechargeable`   | int   | Set if card is Rechargeable.<br>integer values 0 - False, 1 - True<br> | YES | 1|
| `reusable`   | int   | Set whether the card is reusable. A non-reusable card is treated as a Voucher<br> and must be used only once, with the spent amount being equal to or less than the requested transaction amount.<br>integer values 0 - False, 1 - True<br> | YES | 1|
| `cashBack`   | int   | Set if card can benefit from Cashbacks.<br>integer values 0 - False, 1 - True<br> | YES | 0|
| `cardName`   | String   | Optional name of the Card. | NO | "MyCard"|
| `forbiddenObjects`   | List<long>   | A list of IDs representing the branches that should be marked as forbidden for the card. | YES | 101,102,103|


**RESPONSE**

    {
	"transactionId": 10,
    "serialNumber": "111",
    "startDate": "2024-10-30",
    "endDate": "2025-01-30",
    "validMonths": 3,
    "startAmount": 500.0,
	"amountBeforeOperation":0,
    "currentAmount": 500.0,
	"cashbackAmount": 0.0,
    "limitPerDay": 0.0,
    "groupName": "Test",
    "forbiddenObjects": [],
	"cardName": "CardA",
    "active": true,
    "reusable": true,
    "rechargeable": true,
    "cashBack": false,
    "used": false,
    "vname": ""
}

## **ACTIVATE CARD**

**This method can be used once for initial activation of the card**

**REQUEST METHOD : `Post`**/card/activate

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|
| `months`   | int   | Card validity in months.<br>If skipped default value for company will be used<br> | NO | 10|
| `init`   | int   | Default vlue is set to 1 and the card will be activated and initiated.Set to 0 to used addBillData endpoint to initiate the card | NO | 1|

**RESPONSE**

	{
    "transactionId": 0,
    "serialNumber": "1",
    "startDate": "2025-02-27",
    "endDate": "2026-02-27",
    "validMonths": 12,
    "startAmount": 0.00,
    "amountBeforeOperation": 0.00,
    "currentAmount": 0.00,
    "cashbackAmount": 0,
    "limitPerDay": 0.00,
    "initiated": true,
    "groupName": "Test",
    "forbiddenObjects": [],
    "active": true,
    "cashBack": false,
    "rechargeable": false,
    "reusable": false,
    "vname": "hmmm",
    "used": false
	}

## **DEACTIVATE CARD**

**REQUEST METHOD : `Get`**/card/deactivate

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|

**RESPONSE**

    {
        "serial": "07",
        "status": "Deactivate"
    }

## **RESTORE CARD**

**REQUEST METHOD : `Get`**/card/restore

**This method must be used to chane the status of the card to active after deactivation**

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|

**RESPONSE**

    {
        "serial": "07",
        "status": "Success"
    }

## **Edit Card's list of forbidden branches**

**REQUEST METHOD : `Post`**/card/editForbiddenList

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|
| `forbiddenObjects`   | List<long>   | A list of IDs representing the branches that should be marked as forbidden for the card. | YES | 101,102,103|

**RESPONSE**

    {
        "serial": "07",
        "status": "Success"
    }

## **Edit Card Group**

**REQUEST METHOD : `Post`**/card/editGroup

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|
| `newGroupName`   | String   |Name of the new group. | YES | "Clothes"|

**RESPONSE**

    {
        "serial": "07",
        "status": "Success"
    }

## **Edit Card daily sped limit**

**REQUEST METHOD : `Post`**/card/editLimit

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   | Card serial number | YES | 1112222|
| `newLimit`   | Double   | New value to set as limit  | YES | 100.00|

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|
| `newLimit`   | Double   |New daily limit. | YES | 20.50|

**RESPONSE**

    {
        "serial": "07",
        "status": "Success"
    }

## **GET CARD BY SERIAL**

**REQUEST METHOD : `Get`**/card/getCardBySerial

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "12345666"|

**RESPONSE**
    
    {
	"transactionId": 0,
    "serialNumber": "12345666",
    "startDate": "2024-10-15",
    "endDate": "2025-08-15",
    "validMonths": 10,
    "startAmount": 500.0,
	"amountBeforeOperation": 0,
    "currentAmount": 500.0,
	"cashbackAmount": 0.0,
    "limitPerDay": 0.0,
    "initiated": true,
    "groupName": "Test",
    "forbiddenObjects": [],
    "cardName": "",
    "active": true,
    "rechargeable": true,
    "cashBack": false,
    "reusable": true,
    "used": false
    }

## **GET CARD COUNT**

**REQUEST METHOD : `Get`**/card/count

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**RESPONSE**

     8

## **GET CARDS BY COMPANY**

**REQUEST METHOD : `Get`**/card/getCompanyCards

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|
| `onlyExpired`   | int   | Filter show only expired 0-false,1-true | NO | 0|
| `onlyActive`   | int   | Filter show only active 0-false,1-true | NO | 10|
| `onlyLimited`   | int   | Filter show only limited 0-false,1-true. | NO | 0|
| `onlyUsed`   | int   | Filter show only used 0-false,1-true | NO | 10|
| `startDate`   | int   | Show results between start and end date | NO | 0|
| `endDate`   | int   | Show results between start and end date | NO | 10|
| `groupFilter`   | String   | Show only in selected group | NO | 0|

**RESPONSE**

    [
    {
		"transactionId": 0,
        "serialNumber": "111",
        "startDate": "2024-10-15",
        "endDate": "2025-08-15",
        "validMonths": 10,
        "startAmount": 500.0,
		"amountBeforeOperation": 0,
        "currentAmount": 500.0,
		"cashbackAmount": 0.0,
        "limitPerDay": 0.0,
    	"initiated": true,
        "groupName": "Test",
        "forbiddenObjects": [],
        "cardName": "",
        "active": true,
        "rechargeable": true,
        "cashBack": false,
        "reusable": true,
        "used": false
    },
    {
		"transactionId": 0,
        "serialNumber": "11",
        "startDate": "2024-10-30",
        "endDate": "2025-01-30",
        "validMonths": 3,
        "startAmount": 500.0,
		"amountBeforeOperation": 0,
        "currentAmount": 500.0,
		"cashbackAmount": 0.0,
        "limitPerDay": 0.0,
    	"initiated": true,
        "groupName": "Test",
        "forbiddenObjects": [],
        "cardName": "",
        "active": true,
        "rechargeable": false,
        "cashBack": false,
        "reusable": false,
        "used": false
    }
    ]

## **SPEND CARD**

**REQUEST METHOD : `Post`**/card/spend

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `amount`   | double   |Amount to spend from card. | YES | 6.60| 
| `billAsText`   | String   |Text representation of the bill | NO | "Bill text here"|

**Using JSON Body is NOT Require and can be skipped**

**Request body of type JSON**

> transactionDate - String type.

    {
    	"unp" : "UNP1223AE11231231231",
    	"billNumber" : "123551233123",
    	"billTransactionId" : "1112233",
    	"billAmount" : 12.84,
    	"billPaymentType" : "Card",
    	"objectId" : 1,
    	"transactionDate" : "2025-17-10",
    	"items" : [
    {
    	"itemName" : "Домат",
    	"itemGroup" : "Зеленчуци",
    	"itemBarcode" : "",
    	"itemPrice" : 4.80,
    	"itemQuant" : 2.30,
    	"itemDiscount" : 0
    
    },
    {
    	"itemName" : "Репички",
    	"itemGroup" : "Зеленчуци",
    	"itemBarcode" : "",
    	"itemPrice" : 1.80,
    	"itemQuant" : 1,
    	"itemDiscount" : 0
    }
    ]
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

## **Add Bill Data**

**REQUEST METHOD : `Post`**/card/addBillData

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `transactionId`   | Integer   |Amount to spend from card. | YES | 52| 
| `billAsText`   | String   |Text representation of the bill | NO | "Bill text here"|
| `init`   | int   | Default vlue is set to 0 no attempt to initiate the card will be made.Set to 1 to used initiate the card only if the initialization was skipped at the activation part!| NO | 0|

**Request body of type JSON**

----------

**!!! SAME JSON BODY TYPE AND PARAMETERS AS IN SPEND CARD METHOD!!!**

**Using JSON Body is NOT Require and can be skipped**

**Bill text and bill data can be added separetely**

----------

**RESPONSE**

	[
    {
        "type": "BillData",
        "status": "Success",
        "errMsg": "No errors"
    },
    {
        "type": "BillText",
        "status": "Fail",
        "errMsg": "Error billText data for this transaction exists!"
    }
	]


## **Refund amount to CARD**

**REQUEST METHOD : `Post`**/card/refund

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `referralTransId`   | Long   |Id of the Spend transaction. | YES | 1234567|
| `amount`   | double   |Amount to deposit in card. | YES | 6.60|

**Request body of type JSON**

**RESPONSE**
    
    {
	"transactionId": 28,
    "serialNumber": "1",
    "startDate": "2025-01-06",
    "endDate": "2025-07-06",
    "validMonths": 6,
    "startAmount": 50.00,
    "amountBeforeOperation": 103.00,
    "currentAmount": 104.00,
    "cashbackAmount": 0,
    "limitPerDay": 0.00,
    "initiated": true,
    "groupName": "Test",
    "forbiddenObjects": [],
    "active": true,
    "rechargeable": true,
    "reusable": true,
    "vname": "hmmm",
    "cashBack": false,
    "used": false
    }

## **Deposit amount to CARD**

**REQUEST METHOD : `Post`**/card/deposit

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `amount`   | double   |Amount to deposit in card. | YES | 6.60|

**RESPONSE**

    {
    "serialNumber": "10",
    "startDate": "2024-10-30",
    "endDate": "2025-01-30",
    "validMonths": 3,
    "startAmount": 500.0,
	"amountBeforeOperation": 0,
    "currentAmount": 200.0,
	"cashbackAmount": 0.0,
    "limitPerDay": 0.0,
    "initiated": true,
    "groupName": "Test",
    "forbiddenObjects": [],
	"cardName": "",
    "active": true,
    "rechargeable": false,
    "reusable": false,
    "used": true,
    "vname": "",
    "cashBack": false
	}

## **Withdraw amount from CARD**

**REQUEST METHOD : `Post`**/card/withdraw

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','CASHIER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `serial`   | String   |Card serial number. | YES | "10"|
| `amount`   | double   |Amount to withdraw from card. | YES | 5|

**RESPONSE**

{
    "transactionId": 14,
    "serialNumber": "321",
    "startDate": "2025-05-14",
    "endDate": "2026-05-14",
    "validMonths": 12,
    "startAmount": 10.00,
    "amountBeforeOperation": 30.00,
    "currentAmount": 25.00,
    "cashbackAmount": 0,
    "limitPerDay": 0.00,
    "initiated": true,
    "groupName": "Test1",
    "forbiddenObjects": [],
    "active": true,
    "rechargeable": true,
    "vname": "hmmm",
    "reusable": true,
    "cashBack": false,
    "used": false
}

# Report

## *Get Transactions report*

**REQUEST METHOD : `Post`**/report/getTransactions

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','OBSERVER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `branchList`   | List<Int>   | List of branhc ids to use as filter | NO | [2,11,23]|
| `groupList`   | List<Int>   | List of group ids to use as filter | NO | [5,6,7]|
| `operList`   | List<Int>   | List of operator ids to use as filter | No | [22,66,78]|
| `filterCardSerial`   | String   | Card serial to use as filter. Uses wild cards and will return every card with matching to input serial | NO | "123123"|
| `startDate`   | String   | Filter transactions between start and end Dates | NO | "2024-10-30"|
| `endDate`   | String   | Filter transactions between start and end Dates | NO | "2025-01-30"|

**RESPONSE**

	[
    {
        "transactionId": 36,
        "transactionTime": "05 декември 2024 14:31:52",
        "transactionAmount": 6.6,
        "transactionType": "S",
        "cardData": {
            "serialNumber": "99999",
            "startDate": "2024-12-05",
            "endDate": "2025-06-05",
            "validMonths": 6,
            "startAmount": 50.0,
            "amountBeforeOperation": 43.4,
            "currentAmount": 43.4,
            "cashbackAmount": 0.0,
            "limitPerDay": 0.0,
		    "initiated": true,
            "groupName": "Test",
            "forbiddenObjects": [],
            "active": true,
            "vname": "hmmm",
            "reusable": true,
            "cashBack": false,
            "rechargeable": true,
            "used": false
        },
        "branchData": {
            "objId": 9,
            "name": "Storage Shop ",
            "city": "Sofiq",
            "address": "Sofiq Center 11",
            "description": "My Storage",
            "active": true
        },
        "operatorData": {
            "name": "33",
            "defObjId": 9,
            "companyId": 3,
            "userRoles": [
                "MANAGER"
            ],
            "forbiddenObjList": [],
            "active": true
        }
    }
	]	

## *Get Transactions report records count*

**REQUEST METHOD : `Post`**/report/getTransactionsCount

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','OBSERVER' ROLE**

**Request body params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `branchList`   | List<Int>   | List of branhc ids to use as filter | NO | [2,11,23]|
| `groupList`   | List<Int>   | List of group ids to use as filter | NO | [5,6,7]|
| `operList`   | List<Int>   | List of operator ids to use as filter | YES | [22,66,78]|
| `filterCardSerial`   | String   | Card serial to use as filter. Uses wild cards and will return every card with matching to input serial | NO | "123123"|
| `startDate`   | String   | Filter transactions between start and end Dates | NO | "2024-10-30"|
| `endDate`   | String   | Filter transactions between start and end Dates | NO | "2025-01-30"|

**RESPONSE**

    {
    "count": 3
    }

## *Get Transactions bill item info*

**REQUEST METHOD : `Post`**/report/getBillData

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','OBSERVER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `transId`   | Int   | Id of the transaction | YES | 112231|

**RESPONSE with Data**

	{
    "itemStatusResponseDTO": {
        "type": "BillData",
        "status": "Available",
        "billResponseDTO": {
            "billId": 1,
            "billAmount": 6.6,
            "billDate": "2025-17-10",
            "billNumber": "123551233123",
            "billTransactionId": "1112233",
            "billPaymentType": "Card",
            "billUnp": "UNP1223AE11231231231",
            "billDateTime": "12 февруари 2025 10:38:03",
            "itemList": [
                {
                    "itemName": "Домат",
                    "itemBarcode": "",
                    "itemGroup": "Зеленчуци",
                    "itemDiscount": 0.0,
                    "itemPrice": 4.8,
                    "itemQuant": 2.3
                },
                {
                    "itemName": "Репички",
                    "itemBarcode": "",
                    "itemGroup": "Зеленчуци",
                    "itemDiscount": 0.0,
                    "itemPrice": 1.8,
                    "itemQuant": 1.0
                }
            ]
        }
    },
    "textStatusResponseDTO": {
        "type": "BillText",
        "status": "Available",
        "billText": "TESTTETETETETE"
    }
	}

**RESPONSE NO bill Data**

	{
    "itemStatusResponseDTO": {
        "type": "BillData",
        "status": "Unavailable",
        "billResponseDTO": null
    },
    "textStatusResponseDTO": {
        "type": "BillText",
        "status": "Unavailable",
        "billText": null
    }
	}

## *Get available transactions type*

**REQUEST METHOD : `Get`**/report/getTransTypes

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','OBSERVER' ROLE**

**RESPONSE**

	{
    "transList": [
        {
            "transName": "CREATE",
            "transCode": "C"
        },
        {
            "transName": "ACTIVATE",
            "transCode": "A"
        },
        {
            "transName": "DEACTIVATE",
            "transCode": "D"
        },
        {
            "transName": "RESTORE",
            "transCode": "R"
        },
        {
            "transName": "SPEND",
            "transCode": "S"
        },
        {
            "transName": "Deposit",
            "transCode": "De"
        },
        {
            "transName": "Withdraw",
            "transCode": "W"
        },
        {
            "transName": "INFO",
            "transCode": "I"
        },
        {
            "transName": "EDIT_LIMIT",
            "transCode": "EL"
        },
        {
            "transName": "EDIT_GROUP",
            "transCode": "EG"
        },
        {
            "transName": "EDIT_BRANCH",
            "transCode": "EB"
        },
        {
            "transName": "EDIT_DATE",
            "transCode": "ED"
        },
        {
            "transName": "REFUND",
            "transCode": "RF"
        },
        {
            "transName": "INITIATE",
            "transCode": "IN"
        }
    ]
	}

## *Get Revision version for Users*

**REQUEST METHOD : `Get`**/report/getUserRevisions

**Must include in header Authentication : Bearer ... with 'MANAGER','OWNER' ROLE**

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `userId`   | Int   | Id of the User | YES | 112231|
| `startDate`   | String   | Filter transactions between start and end Dates | NO | "2025-01-01"|
| `endDate`   | String   | Filter transactions between start and end Dates | NO | "2025-01-30"|


> **Start date and end date are replaced with a very early date and the last date available if not supplied.**

**RESPONSE**

	{
    "count": 3,
    "userRevisionDTO": [
        {
            "revId": 1,
            "revType": 1,
            "name": "cash",
            "defSubObjId": 2,
            "revDateTime": "09 януари 2025 16:17:05",
            "active": true,
            "cashier": true,
            "observer": true,
            "manager": false,
            "admin": false,
            "accountOwner": false
        },
        {
            "revId": 2,
            "revType": 1,
            "name": "cash",
            "defSubObjId": 4,
            "revDateTime": "09 януари 2025 16:20:11",
            "active": true,
            "cashier": true,
            "observer": true,
            "manager": false,
            "admin": false,
            "accountOwner": false
        },
        {
            "revId": 52,
            "revType": 1,
            "name": "cashNew",
            "defSubObjId": 4,
            "revDateTime": "13 януари 2025 11:20:03",
            "active": true,
            "cashier": true,
            "observer": false,
            "manager": false,
            "admin": false,
            "accountOwner": false
        }
    ]
	}

**REQUEST METHOD : `Get`**/report/getTransWithoutBillData

**Must include in header Authentication : Bearer ... with 'MANAGER','OWNER','CASHIER' ROLE**

    Important Notes
    
    - A filter is applied and transactions are retuned only for the requesting user.
    

**Request params** 

| Parameter  | Type   | Description           |Required | Example         |
| :---       | :---   | :---       			  | :---    | :---	          |
| `page`   | int   | Page number. | YES | 0|
| `size`   | int   | Records per page to show. | YES | 10|
| `startTime`   | String   | Filter transactions between start and end Dates | YES | "2025-02-12T08:59:11.000"|
| `endTime`   | String   | Filter transactions between start and end Dates | YES | "2025-02-12T14:20:11.000"|

**RESPONSE**

> "count" holds the total number of transactions without Bill data for the selected period

> Two transactions are available in this example since the call was made with page : 0 and size : 2

	{
    "count": 3,
    "transactionDataList": [
        {
            "transactionId": 39,
            "transactionTime": "12 февруари 2025 10:37:53",
            "transactionAmount": 5.00,
            "transactionType": "S",
            "cardData": {
                "transactionId": 0,
                "serialNumber": "212121",
                "startDate": "2025-02-11",
                "endDate": "2025-08-11",
                "validMonths": 6,
                "startAmount": 50.00,
                "amountBeforeOperation": 130.00,
                "currentAmount": 130.00,
                "cashbackAmount": 0,
                "limitPerDay": 0.00,
                "groupName": "Test",
                "forbiddenObjects": [],
                "active": true,
                "rechargeable": true,
                "reusable": true,
                "vname": "hmmm",
                "cashBack": false,
                "used": false
            },
            "branchData": {
                "objId": 2,
                "name": "Централен офис",
                "city": "Sofiq",
                "address": "Sofiq M.Luisa 100",
                "description": "Управление",
                "active": true
            },
            "operatorData": {
                "id": 2,
                "name": "1",
                "displayName": "",
                "defObjId": 2,
                "companyId": 2,
                "userRoles": [
                    "MANAGER",
                    "OWNER",
                    "CASHIER",
                    "OBSERVER"
                ],
                "forbiddenObjList": [],
                "active": true
            }
        },
        {
            "transactionId": 40,
            "transactionTime": "12 февруари 2025 11:59:14",
            "transactionAmount": 5.00,
            "transactionType": "S",
            "cardData": {
                "transactionId": 0,
                "serialNumber": "212121",
                "startDate": "2025-02-11",
                "endDate": "2025-08-11",
                "validMonths": 6,
                "startAmount": 50.00,
                "amountBeforeOperation": 130.00,
                "currentAmount": 130.00,
                "cashbackAmount": 0,
                "limitPerDay": 0.00,
                "groupName": "Test",
                "forbiddenObjects": [],
                "active": true,
                "rechargeable": true,
                "reusable": true,
                "vname": "hmmm",
                "cashBack": false,
                "used": false
            },
            "branchData": {
                "objId": 2,
                "name": "Централен офис",
                "city": "Sofiq",
                "address": "Sofiq M.Luisa 100",
                "description": "Управление",
                "active": true
            },
            "operatorData": {
                "id": 2,
                "name": "1",
                "displayName": "",
                "defObjId": 2,
                "companyId": 2,
                "userRoles": [
                    "MANAGER",
                    "OWNER",
                    "CASHIER",
                    "OBSERVER"
                ],
                "forbiddenObjList": [],
                "active": true
            }
        }
    ]
	}

# Error handling

When an error occurs, the API returns a structured JSON response. Below is an example of a typical error response:

    {
    "dateTime": "2024-10-18T16:31:26.1949867",
    "errMsg": "Card serial not found.",
    "userMsg": "Карта с такъв сериен номер не съществува.",
    "details": "uri=/card/activate",
    "errCode": "CARD_IS_NOT_FOUND"
    }

Error Response Fields:


- dateTime: The time when the error occurred.
- errMsg : A user-friendly description of the error.
- userMsg : Message to be displayed for the user.
- details : Additional information about the error, useful for debugging.
- errorCode: A unique identifier for the specific error.

## *Get Transactions report*

**REQUEST METHOD : `Post`**/report/getTransactionsCount

**Must include in header Authentication : Bearer ... with 'OWNER','MANAGER','OBSERVER' ROLE**

## Error description 


### General errors

| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `ACCESS_DENIED`   | 500   | Access Denied unauthorized user |
| `UNEXPECTED_EXCEPTION`   | 500   | Unexpected error |
| `JWT_TOKEN_EXPIRED`   | 500   | JWT Token expired.<br>Start a new session<br> |
| `INVALID_PAGE_INDEX_OR_SIZE`   | 400   | The number of the page or the size is invalid! |
| `INVALID_FIELD_INPUT`   | 400   | Input data is invalid.<br>Boolean data must be 0 or 1<br> |
| `RESOURCE_STATUS_UNCHANGED`   | 400   | Trying to change the status with the same status! |

----------	
### User Errors
----------

| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `NO_RIGHTS_TO_ALTER_USER`   | 403   | The user execiting the command<br>has no authorization to perform this action.<br>see errMsg for details<br> |
| `USERNAME_TAKEN`   | 403   | The supplied user name is already in use. |
| `NO_ACTIVE_COMPANY_ATTACHED`   | 403   | To perform this action the user<br>must be part of an active Company.<br> |
| `INVALID_USER`  | 403   | Supplied user is invalid see errMsg for details |
| `COMPANY_EXTRACTION_ERR`   | 400   | Supplied user has invalid company.<br>See errMsg for details<br> |
| `INVALID_USERNAME`   | 403   | User must contain min 8 symbols and one upper case letter or a special symbol!<br> |
| `INVALID_PASSWORD`   | 403   | Password must contain min 8 symbols a upper case letter and a special symbol!<br> |
| `USER_VALIDITY_PERIOD_EXPIRED`   | 403   | User validity period expired<br> |

### Groups Errors
----------
| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `INVALID_GROUP_NAME`   | 400   | Group name already in use! |

### Company Errors
----------

| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `INVALID_COMPANY`   | 400   | Faild while validating user's company.<br>See errMsg for details<br> |
| `COMPANY_EXPIRED`   | 403   | Payment renewal is required to extend the active period. |
| `COMPANY_ALREADY_EXISTS`   | 403   | Company with this EIK is already in use! |
| `INVALID_BRANCH`   | 400   | Fail while validating the branch status<br>See errMsg for details<br> |
| `INVALID_EMAIL`   | 403   | The supplied E-mail address is invalid!<br> |

### CARD Errors
----------

| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `CARD_SERIAL_NOT_UNIQUE`   | 400   | Cad serial number must be unique. |
| `CARD_IS_NOT_FOUND`   | 404   | Cad serial number not found. |
| `CARD_SUM_MUST_BE_POSITIVE`   | 403   | Invalid sum amount supplied.<br>See errMsg for details<br> |
| `NOT_ENOUGH_FUNDS`   | 403   | Not enough funds to cover the payment. |
| `CARD_BELONGS_TO_ANOTHER_COMPANY`   | 403   | Tis card belongs to another company. |
| `CARD_USE_FORBIDDEN`   | 403   | Card not allowed to be used in this branch. |
| `CARD_NOT_RECHARGEABLE`   | 403   | Deposits not allowed. |
| `CARD_IS_INACTIVE`   | 400   | This card is inactive and can not be used. |
| `CARD_VALIDITY_EXPIRED`   | 403   | Card is expired. |
| `CARD_LIMIT_EXCEEDED`   | 403   | Card day spend limit is exceeded. |
| `CARD_ALREADY_USED`   | 403   | This card can be used once!And it has already been used. |
| `PARTIAL_PAYMENT_NOT_ALLOWED`   | 403   | No partial payments allowed with this card. |
| `REFUND_EXCEEDING_AMOUNT`   | 403   | The amount being refunded is larger than the un refunded amount from the transaction. |

### Transaction Errors
----------
| Error Code | HTTP<br>Status<br>| Description |
| :---       | :---              | :---         |
| `MISSING_TRANSACTION_RESOURCE_ERROR`   | 400   | Missing data for selected resource or data belongs to another company. |