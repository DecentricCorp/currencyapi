FORMAT: 1A
HOST: https://api.lightrail.com/v1/

# Lightrail API

Welcome to the Lightrail API documentation.  Lightrail is an extremely powerful and flexible platform, designed to be rapidly adopted and built into.
This document is part of that experience; in addition to being a fully interactive REST reference, it is designed to minimize implementation time by elaborating on common
use cases, explaining the relevant portions of the Lightrail system and giving common API calls and examples for each.  

Contact us anytime at hello@lightrail.com - we are here to help you solidify your use case and implementation.

### A Note on the Documentation
This interactive documentation will provide full code samples in many popular languages and let you try out all functionality.

The API Blueprint file that produces this documentation is in GitHub at https://github.com/Giftbit/Lightrail-API-Docs. 
If you find a mistake or have a suggestion for improvement, please submit it directly via pull request.

### Common Use Cases
This documentation will explore in detail the most common use cases of the Lightrail API. 
Generally, our users are looking to implement either of the following programs:
<ol>
    <li>A gift card program</li>
    <li>A customer account credit or points program </li>
</ol>

Note that this is not an exhaustive list and some use cases will require a combination of the two. 
Lightrail is designed to power and enhance any use case of stored value.  We are here to work with you.

In each scenario, the power of Lightrail Promotions allows you to incentivize and motivate your users. Lightrail also provides a rich statistical interface which will give you more insight into your programs.

## Getting Started

### API Base URL

The base URL for Lightrail's API is <em>https://api.lightrail.com/v1/</em>.

### Lightrail User Creation and Authorization

You'll need your test or production `access_token` from your Lightrail user details page to use the API. If you don't already have a user account, you can <em><a href="https://www.lightrail.com/#gform" target="_blank">contact us to request a demo</a></em>. Be sure to store your production `access_token` extremely securely in your production configurations.

Note that in your user settings you can toggle to a test mode which provides you with a test `access_token`. This allows you to make calls to the API without affecting your Cards or the calculated statistics in live mode. Emails sent from test mode will be labeled as such.

You authenticate with your `access_token` by including your token prefixed by the word ‘Bearer’ (case sensitive) and a space in the Authorization HTTP header. For example `Authorization: Bearer {access_token}`.

## The Lightrail Object Model
The full API object model is here for your reference. Depending on your use case, you may not need to use every object/endpoint. Read on for use case elaboration

![Lightrail Object Model](http://resources.giftbit.com/api/embeddedimages/Lightrail_Object_Model_(stacked).png)

### Lightrail Cards: The Core Object

The Card is a fundamental object in the Lightrail API and provides the functionality for managing branded currency. 
The concept of a Card can represent any sort of value that your business wishes to issue. 
Simply stated, a Card is a device for which value can be attached and transacted against. 

There are two types of Cards: Gift Cards and Account Cards. These are both represented as a Card but with a different `cardType` attribute. 

#### Gift Cards

Gift Cards, as you might guess, are used when implementing a gift card program. Gift Cards have a `fullcode` which is a unique and unguessable alpha-numeric code that can be distributed to a recipient. 
In a standard gift card program, the recipient would manually enter the `fullcode` during the checkout process.

#### Account Cards

Account Cards represent value that is attached to a Contact. 
They are used when implementing a customer account credit or points program and can be thought of as a customer's account.
Since Account Cards do not have a `fullcode`, their value is interacted with using the Card object. 

## Use Case Exploration

The following sections dive into Lightrail's most common use cases. Feel free to skip to the section relevant to you. If you're not sure which applies to you, please don't hesitate to ask!

Also note, the ability to incentivize your users or recipients through Lightrail Card Promotions is available for both Gift Cards and Account Cards. 
Detailed information for Card Promotions and how it can be applied to either use case can be found in the following [Create and Attach Promotions](#create-and-attach-promotions-anchor) section. 

Looking to import your existing codes into the Lightrail system? We support that. Please let us know to find out more. 

### Use Case 1: Powering Your Gift Card Program
 
A common use case for the Lightrail API is one in which Lightrail powers your gift cards. 

For example, suppose your business is an online e-commerce platform and you want to enable your customers to purchase gift cards.
When a customer purchases a gift card, you would create a Gift Card in the Lightrail API. You would retrieve the Gift Card's `fullcode` and deliver it to the customer.
Your store's checkout process would need to allow customers the opportunity to enter the `fullcode` of any gift cards they have.
When completing the purchase you would create a Transaction using the customer entered `fullcode`, redeeming the required value from the Gift Card, completing the transaction. 

Is your store is running on an established e-commerce platform? 
Lightrail supports popular e-commerce platforms and has dedicated integrations teams here to support you.

Lightrail enables fine-grained control of the rules for which Gift Cards can be issued using Gift Card Programs.

#### Lightrail Gift Card Programs

A Gift Card Program describes the rules and restrictions that apply to the Gift Cards created from that Program. 
For example, a Gift Card Program might declare that all Gift Cards are issued in USD, with an `initialValue` in the range of $5 - 100, and never expire. 

To create a Gift Card, it must be created using a Gift Card Program. 
You may create Gift Card Programs within the <a href="https://www.lightrail.com/app/#/programs" target="_blank">Gift Card</a> section of the Lightrail web application. 
Gift Card Programs are also used to structure and organize related Gift Cards, and also improve tracking of activity within your account. 

Gift Card Programs also provide the flexibility to add integrators. Adding integrators allows you to lend permission so another party can create and issue Gift Cards from your Gift Card Program. 
For more information on this functionality, please contact hello@lightrail.com.

#### Common Requests in Gift Card Integration

| Action                        | Type   | Body                                                                                                                | Endpoint                               | 
|:------------------------------|--------|---------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| Create Gift Card              | `POST` | `{'userSuppliedId':'gc1', 'cardType':'GIFT_CARD', 'programId':'program-123', 'initialValue':500, 'currency':'USD'}` | `/v1/cards`                         | 
| Retrieve `fullcode`           | `GET`  |                                                                                                                     | `/v1/cards/<cardId>/fullcode`       |
| Check Balance                 | `GET`  |                                                                                                                     | `/v1/codes/<fullcode>/card/balance`      |
| Create Transaction            | `POST` | `{'userSuppliedId':'tx1', 'value':-10, 'currency':'USD'}`                                                           | `/v1/codes/<fullcode>/transactions` | 
| Retrieve Transaction History  | `GET`  |                                                                                                                     | `/v1/codes/<fullcode>/transactions` | 

Note, there may be additional endpoints you'd need to use depending on your use-case. 

### Use Case 2: Power an Account Credits or Points Program

Another common use case is one in which Lightrail manages value attached to your customers. 

For example, suppose you'd like to enable your users to earn points within your platform.
Once a user has earned enough points perhaps you'd like to allow them to spend their points and choose a reward.
In this use case, you would create a Contact in the Lightrail API followed by creating an Account Card for that Contact.
This Account Card would be used to track the customer's points. 
As a customer earns or spends value, you would add or subtract value by creating Transactions against the Account Card in the Lightrail API.

#### Common Requests in Account Credits or Points Program

| Action                             | Type   | Body                                                                                               | Endpoint                                                                 | 
|:-----------------------------------|--------|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| Create Contact                     | `POST` | `{'userSuppliedId':'ct1', 'email':'name@example.com'}`                                             | `/v1/contacts`                                                           | 
| Create Account Card                | `POST` | `{'userSuppliedId':'ac1', 'cardType':'ACCOUNT_CARD', 'contactId':'contact-123', 'currency':'XXX'}` | `/v1/cards`                                                              | 
| Check Balance                      | `GET`  |                                                                                                    | `/v1/cards/<cardId>/balance`                                             |
| Create Transaction                 | `POST` | `{'userSuppliedId':'tx1', 'value':-10, 'currency':'XXX'}`                                          | `/v1/cards/<cardId>/transactions`                                        | 
| Retrieve Transaction History       | `GET`  |                                                                                                    | `/v1/cards/<cardId>/transactions`                                        | 
| Retrieve Account Card from Contact | `GET`  |                                                                                                    | `/v1/cards?contactId=<contactId>&currency=<currency>&cardType=ACCOUNT_CARD`|

Note, in these example calls, the currency is set to `XXX` to represent points although `USD` or any other valid currency code could be used if you wanted to track a dollar figure.
Also, in this sort of use case, we recommend storing the Account Card's `cardId` with your customer's account within your application. 
This will remove an unnecessary lookup each time you want to update a customer's points.

If this use case fits your need for points, a great way to reward your customers is offer them gift cards from top brands through <a href="https://www.giftbit.com" target="_blank">Giftbit’s Rewards API.</a>

<a name="create-and-attach-promotions-anchor"></a>
## Create and Attach Promotions

In addition to the many capabilities Lightrail provides out of the box, Lightrail gives you the ability to attach Promotions to Cards, allowing you to incentivize and motivate your users or recipients. 
Promotions work by adding temporary value to your cards with special redemption rules. 
Promotions can be used in both the gift card and account credit use cases. 

Promotions are created from a Promotion Program which can be created from the <a href="https://www.lightrail.com/app/#/promotions" target="_blank">Promotions</a> section of the Lightrail web application. 
Similar to a Gift Card Program, they enable you to restrict promotion properties such the allowed value ranges, currency, and custom expiry. 
A Card is always created with a `PRINCIPAL` ValueStore. When promotions are attached, they are represented as `ATTACHED` ValueStores.

### How Promotions Fit Into the Lightrail Object Model

For example, suppose you've created a $10 USD Card. It could be either a Gift Card or an Account Card. Below is a model of the newly created objects.

![Promotion Object Model Part1](http://resources.giftbit.com/api/embeddedimages/Lightrail_Promitions_Object_Model_p1.png)

Perhaps you want to add a $5 promotion that will be valid for the next 10 days to incentivize the recipient to spend their value. As you can see below, a new ValueStore is attached to Card. 

![Promotion Object Model Part2](http://resources.giftbit.com/api/embeddedimages/Lightrail_Promitions_Object_Model_p2.png)

Suppose the recipient spends $8 USD within the 10-day limit. Below you can see how the balances of the ValueStore and in turn the balance of the Card is affected.

![Promotion Object Model Part3](http://resources.giftbit.com/api/embeddedimages/Lightrail_Promitions_Object_Model_p3.png)

This example illustrates that when Transactions are created against Cards that have multiple ValueStores, any valid ValueStore will be transacted against in order of expiry. 
As you can see, Lightrail automatically handles the complexity of taking value from the correct ValueStore.

### Redemption Rules

Redemption rules are a powerful feature of Lightrail which enable setting extra conditions on promotions. 
When transacting against a card, redemption rules determine whether or not a promotion can be used for the transaction. 
The rules operate on the transaction details including `metadata`. 
The `metadata` field is a flexible JSON structure that can store any additional information about a transaction such as its payment method or cart items. 
Redemption rules, therefore, can unlock powerful marketing promotions such as: "$5 off when you buy a red hat," or "$20 off if you spend more than $100."

Check out the extended <a href="https://github.com/Giftbit/Lightrail-API-Docs/blob/master/feature-deep-dive/RedemptionRules.md" target="_blank">redemption rules documentation</a> for more information.

## Implementation Details

### Coding for Idempotency and the `userSuppliedId` Field

The API is fully idempotent on the `userSuppliedId` field, which is a required parameter for all endpoint operations that result in a change of state on the Lightrail side (POST, PUT, PATCH).  
Therefore, in the case of a non-received response from the API or other unknown condition, it is safe and recommended to retry the request with the same `userSuppliedId`.

For example, your customer is using a `fullcode` in your checkout.  
You POST to the _/codes/{fullcode}/transactions_ endpoint to mark the use, but get a network timeout and are unable to process the response. 
As a result, your system doesn't know if Lightrail successfully received your POST and recorded the transaction. 
With idempotency, you do not need any lookups or other complicated error handling to see if the original API call succeeded. 

You can simply retry the same call (as many times as needed) with the same `userSuppliedId`, and get the same 200 response upon success whether or not a previous request went through. 
Our server will do the operation once and only once for that `userSuppliedId`.

Given the above, it is important you think about how to structure your `userSuppliedId`s and error handling so that you are always sending a unique value for different logical requests, but the same one for any retries.  

Attempting to reuse a `userSuppliedId` but providing different request parameters will result in a 409 error.

### About Currency Value, Currency Type, and No-Currency Use (Such as Points)
Where currency type (eg. USD, CDN, AUD) is required or returned, the API expects/uses 3 character uppercase codes conforming to the ISO-4217 standard. 
Lightrail does not do any currency conversion nor does currency influence internal behavior; rather, currency allows you to issue and track cards in different currencies as you choose.

In all cases where value is concerned, you need to provide the amount in the smallest currency unit. 
For most, this is the amount in cents (or pence, penny, or similarly named unit). For example, to create a Card for USD1.00, you would set the initialValue=100 (100 cents).

For zero-decimal currencies or use with non-currency applications such as points, use the regular whole denomination. 
For example, for ¥1, you should set initialValue=1 (1 JPY), since ¥1 is the smallest currency unit.

### About Dates
The API expects all dates in request parameters to conform to the ISO-8601 format, specifically "yyyy-MM-dd'T'HH:mm:ss.SSSZ".  You can see examples in this documentation.

This allows you to control things such as Code expiry in fine granularity to the time zone of your choosing. 
All responses will always be given in this same format.

### Handling Error Responses

Clients should always check the HTTP status code of the response and act accordingly if the response is not a 200.

Error response JSON will be in the following format:
- status: (number) - will match the HTTP response code.    
- message: (string, optional) - a descriptive error message if one is available
- code: (string, optional) - a code that can be provided to Lightrail support if troubleshooting help is needed

#### Example error responses:
| Status | Description         | Message                                                                                                                                          | 
|:-------|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `400`  | bad request         | Failed to create card due to error during code creation. Response from code creation: Bad Request. Missing Required Parameter 'initialValue'.    | 
| `401`  | unauthorized        | Unauthorized.                                                                                                                                    |
| `409`  | idempotency failure | A different transaction with the same userSuppliedId already exists.                                                                             |
| `429`  | throttled failure   | Too many requests.                                                              

### Legal Responsibilities
The Lightrail API provides flexibility to implement multiple currency solutions (gift cards, unique promo codes, credit refunds, etc). 
It is the responsibility of the API user and their organization to understand and follow the jurisdictions and laws that govern all aspects of their implementation.

## Ping and Health Check [/ping]
Use the /ping endpoint to check that your authorization is working correctly or to health check the API. 

### Ping [GET]
+ Request (application/json)
    + Headers

            Authorization: Bearer <YOUR_ACCESS_TOKEN>

+ Response 200
    + Attributes
        + username (string) - The email address associated with the account.
        + mode (string) - String indicating whether the credentials provided are for TEST or LIVE mode.
        + scopes - A list of scopes associated with the credentials.
        + roles - A list of roles associated with the credentials.
        + effectiveScopes - A list of the effective scopes as a result of combining roles and scopes.
        
    + Body
    
            {
              "user":{
                "username":"tim+apidocresfresh@giftbit.com",
                "mode":"TEST",
                "scopes":[
                ],
                "roles":[
                  "accountManager",
                  "contactManager",
                  ...
                ],
                "effectiveScopes":[
                  {
                    "deny":false,
                    "scopePath":[
                      "lightrailV1",
                      "account"
                    ]
                  },
                  {
                    "deny":false,
                    "scopePath":[
                      "lightrailV1",
                      "payments"
                    ]
                  },
                  ...
                ]
              }
            }
        
+ Response 401

        {
            "status": 401,
            "message": "Unauthorized",
            "code": "CREDENTIALS_INVALID"
        }

## Cards [/cards/]
The _/cards_ endpoint is for use by your system to create and manage your Gift Cards and Account Cards.
The `cardId` should be stored as it's used in many subsequent requests - such as transaction against a Card. 
In use cases where the recipient requires a gift code, the `fullcode` should be supplied. 

---
### List Cards [GET /cards/{?limit}{?offset}{?categoryKey}{?categoryValue}{?contactId}{?cardType}{?currency}{?userSuppliedId}]
Retrieve a paginated list of Cards.

---
+ Request (application/json)
    + Headers
    
             Authorization: Bearer <YOUR_ACCESS_TOKEN>
   
+ Parameter
    + limit (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
    + offset (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
    + categoryKey (string, optional) - A key of a Category. 
    + categoryValue (string, optional) - A value of a Category. 
    + contactId (string, optional) - A contactId to filter by.
    + cardType (string, optional) - `ACCOUNT_CARD`, `GIFT_CARD`.
    + currency (string, optional) - The three-character ISO-4217 currency.
    + userSuppliedId (string, optional) - Endpoint-unique idempotency ID provided by the client.

+ Response 200
    + Attributes 
        + cards (array[Card])
        + pagination (Pagination)

    + Body
    
            {
              "cards":[
                {
                  "cardId":"card-a063cbe39ab1402ebf997cf088e81b82",
                  "userSuppliedId":"anonymous-giftcard28",
                  "contactId":null,
                  "dateCreated":"2017-07-28T21:21:00.606Z",
                  "categories":[
                    {
                      "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                      "key":"giftbit_order",
                      "value":"2017-07-28"
                    },
                    {
                      "categoryId":"category-d2318a92c00f411987076a64243a0b57",
                      "key":"giftbit_program",
                      "value":"program-93f8264813b44877a08e8b7d9ee205e3"
                    }
                  ],
                  "cardType":"GIFT_CARD",
                  "currency":"USD"
                },
                {
                  "cardId":"card-15b464bb6c164751b394f1a99df373bd",
                  "userSuppliedId":"anonymous-giftcard3",
                  "contactId":null,
                  "dateCreated":"2017-07-28T21:21:00.599Z",
                  "categories":[
                    {
                      "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                      "key":"giftbit_order",
                      "value":"2017-07-28"
                    },
                    {
                      "categoryId":"category-d2318a92c00f411987076a64243a0b57",
                      "key":"giftbit_program",
                      "value":"program-93f8264813b44877a08e8b7d9ee205e3"
                    }
                  ],
                  "cardType":"GIFT_CARD",
                  "currency":"USD"
                }
              ],
              "pagination":{
                "count":2,
                "limit":2,
                "maxLimit":1000,
                "offset":0,
                "totalCount":94
              }
            }


### Retrieve Card by cardId [GET /cards/{cardId}]
+ Parameters 
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

+ Response 200

    + Attributes 
        + card (Card)

    + Body
        
            {
              "card":{
                "cardId":"card-a063cbe39ab1402ebf997cf088e81b82",
                "userSuppliedId":"anonymous-giftcard28",
                "contactId":null,
                "dateCreated":"2017-07-28T21:21:00.606Z",
                "categories":[
                  {
                    "categoryId":"category-d2318a92c00f411987076a64243a0b57",
                    "key":"giftbit_program",
                    "value":"program-93f8264813b44877a08e8b7d9ee205e3"
                  },
                  {
                    "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                    "key":"giftbit_order",
                    "value":"2017-07-28"
                  }
                ],
                "cardType":"GIFT_CARD",
                "currency":"USD"
              }
            }


### Create Account Card [POST /cards]
Create a Card of type `ACCOUNT_CARD` which is associated with an existing Contact.

---
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Attributes 
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
        + cardType (string, required) - ACCOUNT_CARD
        + currency (string, required) - The three-character ISO-4217 currency.
        + initialValue (number, optional) - If not provided, will default to 0.
        + categories (object, optional) - An object of key-value pairs. For example: `'categories': {'city':'Seattle', 'country':'USA'}`.
        + contactId (string, required) - Note, the Contact must be created before the request to create the card.
        + expires (string, optional) - Defaults to never expires.
        + startDate (string, optional) - The date for which the associated ValueStore will become usable.
        + inactive (boolean, optional) - If `true` the Card's `PRINCIPAL` ValueStore will have an `INACTIVE` balance.

    + Body
    
            {
              "userSuppliedId":"accountCard1",
              "cardType":"ACCOUNT_CARD",
              "contactId":"contact-0cfafcd94b2d42a698771a7b63e37c86",
              "currency":"USD",
              "initialValue":500,
              "categories": {
                "city":"seattle"
              }
            }
        
+ Response 200
    + Attributes
        + card (Card)
        
    + Body

            {
              "card":{
                "cardId":"card-d4e04d050acf48e1ae63657f4c710abf",
                "userSuppliedId":"giftcard1",
                "contactId":"contact-0cfafcd94b2d42a698771a7b63e37c86",
                "dateCreated":"2017-07-28T21:59:15.325Z",
                "categories":[
                  {
                    "categoryId":"category-597f1f7f98b64285b7763548f83fff67",
                    "key":"city",
                    "value":"seattle"
                  },
                  {
                    "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                    "key":"giftbit_order",
                    "value":"2017-07-28"
                  }
                ],
                "cardType":"ACCOUNT_CARD",
                "currency":"USD"
              }
            }
            

### Create Gift Card [POST /cards]
Create a Card of type `GIFT_CARD`.

---
+ Request (application/json)

    + Headers

            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Attributes 
        + userSuppliedId (string, required) - Unique idempotency ID for the request.
        + cardType (string, required) - GIFT_CARD
        + initialValue (number, optional) - If not provided, will default to 0.
        + currency (string) - The three-character ISO-4217 currency.
        + programId (string, required) - The Lightrail Program ID.
        + categories (object, optional) - An object of key-value pairs. For example: `'categories': {'city':'Seattle', 'country':'USA'}`.
        + contactId (string, optional) - Note, the Contact must be created before the request to create the card.
        + expires (string, optional) - Defaults to never expires.
        + startDate (string, optional) - The date for which the ValueStore will become useable.
        + inactive (boolean, optional) - If `true` the Card's `PRINCIPAL` ValueStore will have an `INACTIVE` balance.

    + Body
    
            {
              "userSuppliedId":"giftcard1",
              "cardType":"GIFT_CARD",
              "currency":"USD",
              "initialValue":500,
              "categories": {
                "city":"seattle"
              }
            }
        
+ Response 200
    + Attributes
        + card (Card)
        
    + Body

            {
              "card":{
                "cardId":"card-d4e04d050acf48e1ae63657f4c710abf",
                "userSuppliedId":"giftcard1",
                "contactId":null,
                "dateCreated":"2017-07-28T21:59:15.325Z",
                "categories":[
                  {
                    "categoryId":"category-597f1f7f98b64285b7763548f83fff67",
                    "key":"city",
                    "value":"seattle"
                  },
                  {
                    "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                    "key":"giftbit_order",
                    "value":"2017-07-28"
                  }
                ],
                "cardType":"GIFT_CARD",
                "currency":"USD"
              }
            }


### Retrieve fullcode for Gift Card [GET /cards/{cardId}/fullcode]
Retrieves the `fullcode` associated with a Gift Card.

---
+ Parameters 
    + cardId (string) - The Lightrail Card ID. Note, Card must have cardType = `GIFT_CARD`.
    
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

+ Response 200
    + Attributes 
        + fullcode (Fullcode)

    + Body
    
            {
                "fullcode":{
                    "code" : "5JBUR-9V922-5EDL8-RP5R5-YYEVY"
                }
            }


### Update Contact on Card [PATCH /cards/{cardId}]
Update the Contact associated with a Card.

---
+ Parameters 
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
    + Attributes
        + contactId (string, required) - The Contact's unique identifier which you'd like to update the Card with.
            
    + Body
    
            {
                "contactId": "contact-592941d2335d402e9dc3d6a981bc3896"
            }

+ Response 200

    + Attributes 
        + card (Card)

    + Body
        
            {
              "card": {
                "cardId": "card-2235b727f65a46648138429b95fdaa7b",
                "userSuppliedId": "testCategoryCreation10-19",
                "contactId": "contact-592941d2335d402e9dc3d6a981bc3896",
                "dateCreated": "2017-05-16T16:49:33.027Z",
                "categories": [
                  {
                    "categoryId": "category-48ecc24b16944b40a1218e319facd24a",
                    "key": "giftbit_order",
                    "value": "testCategoryCreationId9"
                  }
                ],
                "cardType": "GIFT_CARD",
                "currency": "USD"
              }
            }

### Activate Card [POST /cards/{cardId}/activate]
Activate Cards that were created as inactive.

---
+ Parameters
    + cardId (string, required) - The Card must have been created with `"inactive":true`.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required) - Unique idempotency ID for the request.
        
    + Body 
    
            {
                "userSuppliedId":"activate-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body
    
            {
              "transaction":{
                "transactionId":"transaction-7ad3325fa13d4d53a193d8c2d75ecb2f",
                "value":500,
                "userSuppliedId":"activate-1",
                "dateCreated":"2017-07-28T22:12:19.924Z",
                "transactionType":"ACTIVATE",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":500,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-8b2a4c9cb4b745fcb4ddc286038bc4a9",
                "currency":"USD",
                "codeLastFour":"99SY"
              }
            }

### Freeze Card [POST /cards/{cardId}/freeze]
Freeze a Card, preventing all transactions until unfrozen. 

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required)
    + Body 
    
            {
                "userSuppliedId":"freeze-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
              "transaction":{
                "transactionId":"transaction-d2b9ded0e836471d8ea054865140ee06",
                "value":-500,
                "userSuppliedId":"freeze-1",
                "dateCreated":"2017-07-28T22:15:51.265Z",
                "transactionType":"FREEZE",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":0,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-8b2a4c9cb4b745fcb4ddc286038bc4a9",
                "currency":"USD",
                "codeLastFour":"99SY"
              }
            }        


### Unfreeze Card [POST /cards/{cardId}/unfreeze]
Unfreeze a frozen Card, re-enabling the creation of transactions.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required)
    + Body 
    
            {
                "userSuppliedId":"unfreeze-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
              "transaction":{
                "transactionId":"transaction-a9506ed18c9b4b78afb98f6b31956ffa",
                "value":500,
                "userSuppliedId":"unfreeze-1",
                "dateCreated":"2017-07-28T22:16:38.956Z",
                "transactionType":"UNFREEZE",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":500,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-8b2a4c9cb4b745fcb4ddc286038bc4a9",
                "currency":"USD",
                "codeLastFour":"99SY"
              }
            }

### Cancel Card [POST /cards/{cardId}/cancel]
Permanently cancels a Card.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
    + Attributes
        + userSuppliedId (string, required) - Unique idempotency ID for the request.
    
    + Body
            
            {
                "userSuppliedId": "cancel-1"
            }
    
+ Response 200

    + Body

            {
              "card":{
                "cardId":"card-8b2a4c9cb4b745fcb4ddc286038bc4a9",
                "userSuppliedId":"giftcard2",
                "contactId":null,
                "dateCreated":"2017-07-28T22:11:09.431Z",
                "categories":[
                  {
                    "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                    "key":"giftbit_order",
                    "value":"2017-07-28"
                  },
                  {
                    "categoryId":"category-597f1f7f98b64285b7763548f83fff67",
                    "key":"city",
                    "value":"seattle"
                  },
                  {
                    "categoryId":"category-660c0b26726d4615bb8b4bbd52ae17d2",
                    "key":"giftbit_status",
                    "value":"CANCELLED"
                  }
                ],
                "cardType":"GIFT_CARD",
                "currency":"USD"
              }
            }

## Contacts [/contacts]
Contacts can be attached to Cards to help you keep track of who is interacting with your Cards.

---
### List Contacts [GET /contacts{?limit}{?offset}{?email}{?firstName}{?lastName}{?userSuppliedId}]
Retrieve a paginated list of Contacts.

---
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
+ Parameters 
    + limit (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
    + offset (number, optional) - For pagination. The offset of the first results in the total results. Default 0.
    + email (string, optional) - The contact's email.
    + firstName (string, optional) - The contact's first name.
    + lastName (string, optional) - The contact's last name.
    + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
    
+ Response 200
    + Attributes
       + contact (Contact)
       + pagination (Pagination)

    + Body
    
            {
              "contacts":[
                {
                  "contactId":"contact-0cfafcd94b2d42a698771a7b63e37c86",
                  "userSuppliedId":"contact3",
                  "email":"johnsmom@example.com",
                  "firstName":"Sarah",
                  "lastName":"Connor",
                  "dateCreated":"2017-07-28T21:21:04.000Z"
                },
                {
                  "contactId":"contact-5e5633f937894404901e3f5cbecf8e2c",
                  "userSuppliedId":"contact4",
                  "email":"whatshappening@example.com",
                  "firstName":"Bill",
                  "lastName":"Lumbergh",
                  "dateCreated":"2017-07-28T21:21:04.000Z"
                }
              ],
              "pagination":{
                "count":61,
                "limit":100,
                "maxLimit":1000,
                "offset":0,
                "totalCount":61
              }
            }


### Show Contact [GET /contacts/{contactId}]
+ Parameters
    + contactId (string, required) - The Lightrail Contact ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
    
+ Response 200
    + Attributes
        + contact (Contact)

    + Body
    
            {
              "contact":{
                "contactId":"contact-0cfafcd94b2d42a698771a7b63e37c86",
                "userSuppliedId":"contact3",
                "email":"johnsmom@example.com",
                "firstName":"Sarah",
                "lastName":"Connor",
                "dateCreated":"2017-07-28T21:21:04.000Z"
              }
            }


### Create Contact [POST /contacts]
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Attributes 
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
        + email (string, optional) - The contact's email.
        + firstName (string, optional) - The contact's first name.
        + lastName (string, optional) - The contact's last name.
        
            
    + Body
    
            {
                "userSuppliedId": "12345678",
                "email": "noreply@giftbit.com"
            }
        

    
+ Response 200
    + Attributes 
        + contact (Contact)

    + Body
    
            {
                "contact":
                {
                    "contactId": "contact-0f06b54b9fa44727a7ba7043de5365cf",
                    "userSuppliedId": "12345678",
                    "email": "noreply@giftbit.com",
                    "firstName": null,
                    "lastName": null,
                    "dateCreated": "2016-12-09T00:06:00.000Z"
                }
            }


### Update Contact [PATCH /contacts/{contactId}/]
+ Parameters
    + contactId (string, required)
    
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
    + Attributes
        + email (string, optional) 
        + firstName (string, optional)
        + lastName (string, optional) 
            
    + Body
    
            {
                "firstName": "Jane",
                "lastName": "Smith"
            }
    
+ Response 200
    + Attributes 
        + contact (Contact)

    + Body
    
            {
                "contact":
                {
                    "contactId": "contact-0f06b54b9fa44727a7ba7043de5365cf",
                    "userSuppliedId": "12345678",
                    "email": "noreply@giftbit.com",
                    "firstName": "Jane",
                    "lastName": "Smith",
                    "dateCreated": "2016-12-09T00:06:00.000Z"
                }
            }


### Retrieve Account Cards or Gift Cards for Contact [GET /cards{?contactId}{?cardType}{?currency}]
Retrieve a paginated list of a Contact's Cards.

---
+ Parameters 
    + contactId (string, required) - The Lightrail Contact ID.
    + cardType (string, required) - `ACCOUNT_CARD`, `GIFT_CARD`.
    + currency (string, optional) - The three-character ISO-4217 currency. Only needed if your Contact has Account Cards in multiple currencies.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + cards (array[Card])
        + pagination (Pagination)

    + Body
    
            {
              "cards":[
                {
                  "cardId":"card-f35b7fb3f7c5435385b709fc7b2bc7a5",
                  "userSuppliedId":"contact2-account",
                  "contactId":"contact-e56b00c7bb8e4f45ac61f941d3f557ba",
                  "dateCreated":"2017-07-03T21:21:00.565Z",
                  "categories":[
                    {
                      "categoryId":"category-3d4ff98bf941489e8216c9ffb9d4efc3",
                      "key":"giftbit_order",
                      "value":"2017-07-28"
                    },
                    {
                      "categoryId":"category-bf9bfbac78004199b2d544d6c830d3c2",
                      "key":"giftbit_program",
                      "value":"program-account-USD-user-1dfd10e5cb3c451382bc79774ab33232-TEST"
                    }
                  ],
                  "cardType":"ACCOUNT_CARD",
                  "currency":"USD"
                }
              ],
              "pagination":{
                "count":1,
                "limit":100,
                "maxLimit":1000,
                "offset":0,
                "totalCount":1
              }
            }

## Transactions [/cards/{cardId}/transactions]
A feature the Lightrail API supports is the ability to create Pending Transactions. 
This means the value required for the Pending Transaction will be unavailable to be used for other Transactions. 
A Pending Transaction can either be captured, which confirms the Transaction, or it can be voided, freeing up their associated value. 

### List Card Transactions [GET /cards/{cardId}/transactions]
Retrieve a paginated list of a Card's Transactions.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + limit (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
        + offset (number, optional) - For pagination. The offset of the first results in the total results. Default 0.
    
+ Response 200
    + Attributes
        + transactions (array[Transaction])
        + pagination (Pagination)

    + Body
    
            {
              "transactions":[
                {
                  "transactionId":"transaction-622de8c7b5604808b7576985db59fe70",
                  "value":-500,
                  "userSuppliedId":"example2",
                  "dateCreated":"2017-07-31T18:38:02.449Z",
                  "transactionType":"DRAWDOWN",
                  "transactionAccessMethod":"CARDID",
                  "valueAvailableAfterTransaction":1500,
                  "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                  "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                  "currency":"USD",
                  "metadata":{
                    "checkout-cart":{
                      "items":[
                        {
                          "id":"1"
                        },
                        {
                          "id":"2"
                        }
                      ]
                    }
                  }
                },
                {
                  "transactionId":"transaction-0094bdcc1be34b0ebc9e957d5dd924a5",
                  "value":2000,
                  "userSuppliedId":"anonymous-giftcard10",
                  "dateCreated":"2017-07-13T21:21:00.601Z",
                  "transactionType":"INITIAL_VALUE",
                  "transactionAccessMethod":"CARDID",
                  "valueAvailableAfterTransaction":2000,
                  "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                  "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                  "currency":"USD",
                  "metadata":{
                    "giftbit_override_dateCreated":"2017-07-13T21:21:00.601Z"
                  }
                }
              ],
              "pagination":{
                "count":2,
                "limit":100,
                "maxLimit":1000,
                "offset":0,
                "totalCount":2
              }
            }


### Show Transaction [GET /cards/{cardId}/transactions/{transactionId}]
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + transactionId (string, required) - The Lightrail Transaction ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            { "transaction": {
                {
                  "transactionId":"transaction-622de8c7b5604808b7576985db59fe70",
                  "value":-500,
                  "userSuppliedId":"example2",
                  "dateCreated":"2017-07-31T18:38:02.449Z",
                  "transactionType":"DRAWDOWN",
                  "transactionAccessMethod":"CARDID",
                  "valueAvailableAfterTransaction":1500,
                  "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                  "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                  "currency":"USD",
                  "metadata":{
                    "checkout-cart":{
                      "items":[
                        {
                          "id":"1"
                        },
                        {
                          "id":"2"
                        }
                      ]
                    }
                  }
                }
            }


### Create Transaction [POST /cards/{cardId}/transactions]
Creates a transaction against a Card. Transactions can be created as pending which locks the value required for the Transaction until it is either captured or voided. 

---
+ Parameters
    + cardId (string, required)

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + value (number) - The value of the transaction. Can be negative or positive. Note, value is in the smallest unit for the currency. If USD, then value represents cents. If the Transaction represents a $50 USD redemption value = -5000.
        + currency (required) - The three-character ISO-4217 currency.
        + metadata (Metadata, optional) - A key-value object to store additional information about the transaction. Note Lightrail's [Redemption Rules](https://giftbit.github.io/Lightrail-API-Docs/RedemptionRules) operate on Transaction metadata to determine whether a particular promotion can be spent. Example: `"metadata":{"checkout-cart":{"items":[{"id":"1"},{"id":"2"}]}}`. Also note, the `giftbit_*` namespace for keys is reserved.
        + pending (boolean, optional) - If `true`, the required value will be locked, however in order to complete the transaction the pending transaction must be captured.
        
    + Body 
    
            {
              "userSuppliedId":"example2",
              "value":-500,
              "currency":"USD",
              "metadata":{
                "checkout-cart":{
                  "items":[
                    {
                      "id":"1"
                    },
                    {
                      "id":"2"
                    }
                  ]
                }
              }
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
              "transaction":{
                "transactionId":"transaction-622de8c7b5604808b7576985db59fe70",
                "value":-500,
                "userSuppliedId":"exmaple2",
                "dateCreated":"2017-07-31T18:38:02.449Z",
                "transactionType":"DRAWDOWN",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":1500,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                "currency":"USD",
                "metadata":{
                  "checkout-cart":{
                    "items":[
                      {
                        "id":"1"
                      },
                      {
                        "id":"2"
                      }
                    ]
                  }
                }
              }
            }


### Capture Pending Transaction [POST /cards/{cardId}/transactions/{transactionId}/capture]
Captures a pending Transaction. This locks in the value of the pending Transaction.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card Id.
    + transactionId (string, required) - The Lightrail Transaction ID. Must be a pending Transaction.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
            
    + Attributes
        + userSuppliedId (string, required) - Unique idempotency ID for the request.

+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
              "transaction":{
                "transactionId":"transaction-38b58e7a8ea4478c997e66d80239e052",
                "value":-50,
                "userSuppliedId":"transaction-901c9043830d4b96af2982d3f2b103f6-capture",
                "dateCreated":"2017-07-31T18:50:25.357Z",
                "transactionType":"DRAWDOWN",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":1349,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                "currency":"USD",
                "parentTransactionId":"transaction-901c9043830d4b96af2982d3f2b103f6",
                "metadata":{
                  "giftbit_initial_transaction_id":"transaction-901c9043830d4b96af2982d3f2b103f6"
                }
              }
            }


### Void Pending Transaction [POST /cards/{cardId}/transactions/{transactionId}/void]
Voids a pending Transaction. This makes the value of the pending Transaction available to be used by other Transactions.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + transactionId (string, required) - The Lightrail Transaction ID. Must be a pending Transaction.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Attributes
        + userSuppliedId (string, required) - Unique idempotency ID for the request. 
    
+ Response 200
    + Attributes
        + transaction (Transaction)
        
    + Body

            {
              "transaction":{
                "transactionId":"transaction-7dfa13635e0a468ca6257ff02bc4023b",
                "value":50,
                "userSuppliedId":"transaction-fb3d3a0c76ac4d809dfbd4be3d77aa5a-reverse",
                "dateCreated":"2017-07-31T18:54:58.141Z",
                "transactionType":"PENDING_VOID",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":1299,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                "currency":"USD",
                "parentTransactionId":"transaction-fb3d3a0c76ac4d809dfbd4be3d77aa5a",
                "metadata":{
                  "giftbit_initial_transaction_id":"transaction-fb3d3a0c76ac4d809dfbd4be3d77aa5a"
                }
              }
            }


### Refund Transaction [POST /cards/{cardId}/transactions/{transactionId}/refund]
Refund a Transaction, reverting its affects.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + transactionId (string, required) -The Lightrail Transaction ID. Must be an existing Transaction with transactionType `DRAWDOWN`.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Attributes
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
    
+ Response 200
    + Attributes
        + transaction (Transaction)
        
    + Body

            {
              "transaction":{
                "transactionId":"transaction-77e01a30d919402eafdf1b62a13b9634",
                "value":50,
                "userSuppliedId":"transaction-9efd0613b8514bfc8e983b626b72b3a3-reverse",
                "dateCreated":"2017-07-31T18:57:44.844Z",
                "transactionType":"DRAWDOWN_REFUND",
                "transactionAccessMethod":"CARDID",
                "valueAvailableAfterTransaction":1299,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                "currency":"USD",
                "parentTransactionId":"transaction-9efd0613b8514bfc8e983b626b72b3a3",
                "metadata":{
                  "giftbit_initial_transaction_id":"transaction-9efd0613b8514bfc8e983b626b72b3a3"
                }
              }
            }
            

## Codes [/codes/{fullcode}/transactions]
These endpoints are used when interacting directly with the `fullcode` of a Gift Card.

---
### List Transactions [GET /codes/{fullcode}/transactions{?pin}]
Retrieve a paginated list of a Card's Transactions using the Gift Card's `fullcode`.

---
+ Parameters
    + fullcode (string, required) - The unique gift code.
    + pin (string, optional) - This is required if the fullcode has a pin.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + transactions (array[Transaction])
        + pagination (Pagination)

    + Body

            {
              "transactions":[
                {
                  "transactionId":"transaction-e099b4fc262b4c98a65d79e42fd6f4f5",
                  "value":-599,
                  "userSuppliedId":"anonymous-giftcard10-transaction1",
                  "dateCreated":"2017-07-28T21:21:10.009Z",
                  "transactionType":"DRAWDOWN",
                  "transactionAccessMethod":"CARDID",
                  "valueAvailableAfterTransaction":1401,
                  "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                  "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                  "currency":"USD",
                  "metadata":{
                    "giftbit_override_dateCreated":"2017-07-28T21:21:10.009Z"
                  }
                },
                {
                  "transactionId":"transaction-0094bdcc1be34b0ebc9e957d5dd924a5",
                  "value":2000,
                  "userSuppliedId":"anonymous-giftcard10",
                  "dateCreated":"2017-07-13T21:21:00.601Z",
                  "transactionType":"INITIAL_VALUE",
                  "transactionAccessMethod":"CARDID",
                  "valueAvailableAfterTransaction":2000,
                  "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                  "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                  "currency":"USD",
                  "metadata":{
                    "giftbit_override_dateCreated":"2017-07-13T21:21:00.601Z"
                  }
                }
              ],
              "pagination":{
                "count":2,
                "limit":100,
                "maxLimit":1000,
                "offset":0,
                "totalCount":2
              }
            }


### Show Transaction [GET /codes/{fullcode}/transactions/{transactionId}{?pin}]
+ Parameters
    + transactionId (string, required) - The Lightrail Transaction ID.
    + fullcode (string, required) - The unique unguessable gift code.
    + pin (string, optional) - This is required if the gift code has a pin.
    

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + transaction (Transaction)
        
    + Body 
    
            {
                transaction":{
                    "transactionId": "transaction-3ec8ad2afea14c04906159ee8f1ad735",
                    "value": -50,
                    "userSuppliedId": "gift_card_4-inactive",
                    "dateCreated": "2017-06-05T15:39:25.411Z",
                    "transactionType": "INACTIVATE",
                    "transactionAccessMethod": "CARDID",
                    "valueAvailableAfterTransaction": 0,
                    "giftbitUserId": "user-5022fccf827647ee9cfb63b779d62193-TEST",
                    "codeLastFour": "S9X5",
                    "cardId": "card-9991e39b4526401f85a889cb20b1a465",
                    "currency": "XXX"
                }
            }


### Create Transaction [POST /codes/{fullcode}/transactions{?pin}]
Creates a transaction against a Card using the Gift Card's `fullcode`.

---
+ Parameters
    + fullcode (string, required) - The unique gift code.
    + pin (string, optional) - This is required if the fullcode has a pin.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + value (number) - The value of the transaction. Must be negative when transacting against the fullcode. Note, value is in the smallest unit for the currency. If USD, then value represents cents. If the Transaction represents a $50 USD redemption value = -5000.
        + currency (required) - The three-character ISO-4217 currency.
        + metadata (Metadata, optional) - A key-value object to store additional information about the transaction. Note Lightrail's [Redemption Rules](https://giftbit.github.io/Lightrail-API-Docs/RedemptionRules) operate on Transaction metadata to determine whether a particular promotion can be spent. Example: `"metadata":{"checkout-cart":{"items":[{"id":"1"},{"id":"2"}]}}`. Also note, the `giftbit_*` namespace for keys is reserved.
        + pending (boolean, optional) - If `true`, the required value will be locked, however in order to complete the transaction the pending transaction must be captured.
        
    + Body 
    
            {
              "userSuppliedId":"example2",
              "value":-500,
              "currency":"USD",
              "metadata":{
                "checkout-cart":{
                  "items":[
                    {
                      "id":"1"
                    },
                    {
                      "id":"2"
                    }
                  ]
                }
              }
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
              "transaction":{
                "transactionId":"transaction-622de8c7b5604808b7576985db59fe70",
                "value":-500,
                "userSuppliedId":"exmaple2",
                "dateCreated":"2017-07-31T18:38:02.449Z",
                "transactionType":"DRAWDOWN",
                "transactionAccessMethod":"RAWCODE",
                "valueAvailableAfterTransaction":1500,
                "giftbitUserId":"user-1dfd10e5cb3c451382bc79774ab33232-TEST",
                "cardId":"card-76f0f09b52df40f29252d1abd886cbab",
                "currency":"USD",
                "metadata":{
                  "checkout-cart":{
                    "items":[
                      {
                        "id":"1"
                      },
                      {
                        "id":"2"
                      }
                    ]
                  }
                }
              }
            }


## Balances [/cards/{cardId}/balance/]
These endpoints are used to get the available balance associated with a Card.

---
### Get Balance by cardId [GET /cards/{cardId}/balance{?asAtDate}]
Retrieve a Card's balance.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + asAtDate (string, optional) - The date and time you'd like to retrieve the balance at.
+ Request (application/json)
    + Headers

            Authorization: Bearer <YOUR_ACCESS_TOKEN>

+ Response 200
    + Attributes (Balance)

    + Body

            {
                "balance":{
                "principal":{
                "currentValue": 150,
                "state": "ACTIVE",
                "expires": null,
                "startDate": null,
                "programId": "program-dedccd921d074f1eb6c0f75fc5ebee72",
                "valueStoreId": "value-18638d4486ad4feaab125a61f5df3079"
                },
                "attached":[
                ],
                "currency": "USD",
                "cardType":"GIFT_CARD",
                "balanceDate": "2017-06-05T17:11:36.999Z"
                }
            }


### Get Balance by fullcode [GET /codes/{fullcode}/card/balance{?asAtDate}{?pin}]
Retrieve a Card's balance using the Gift Card's `fullcode`.

---
+ Parameters
    + fullcode (string, required) - The unique unguessable gift code.
    + pin (string, optional) - This is required if the fullcode has a pin.
    + asAtDate (string, optional) - The date and time you'd like to retrieve the balance at.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes (Balance)

    + Body

            {
                "balance":{
                "principal":{
                "currentValue": 150,
                "state": "ACTIVE",
                "expires": null,
                "startDate": null,
                "programId": "program-dedccd921d074f1eb6c0f75fc5ebee72",
                "valueStoreId": "value-18638d4486ad4feaab125a61f5df3079"
                },
                "attached":[
                ],
                "currency": "USD",
                "cardType":"GIFT_CARD",
                "balanceDate": "2017-06-05T17:11:36.999Z"
                }
            }


### Get Balance by fullcode (Deprecated) [GET /codes/{fullcode}/balance{?asAtDate}{?pin}]
Note, this endpoint is deprecated and should no longer be used. 

---
+ Parameters
    + fullcode (string, required)
    + pin (string, optional) - This is required if the fullcode has a pin.
    + asAtDate (string, optional) - The date and time you'd like to retrieve the balance at.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Body

            {
                balance":{
                "currentValue": 150,
                "fundedValue": 0,
                "drawdownValue": 0,
                "cancellationValue": 0,
                "initialValue": 150,
                "frozenValue": 0,
                "inactiveValue": 0,
                "pendingValue": 0,
                "currency": "USD"
                }
            }

## ValueStores: Managing Promotions [/cards/{cardId}/valueStores/]
These endpoints are used for managing Promotion Programs.

---
### List ValueStores [GET /cards/{cardId}/valueStores]
Retrieve a paginated list of a Card's ValueStores.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

+ Response 200
    + Attributes
        + valueStores (array[ValueStore])
        + pagination (Pagination)

    + Body

            {
                "valueStores":[
                {
                    "cardId": "card-9991e39b4526401f85a889cb20b1a465",
                    "valueStoreId": "value-0ff47ca176eb4708b08a8bb010584704",
                    "valueStoreType": "PRINCIPAL",
                    "currency": "XXX",
                    "dateCreated": "2017-06-05T15:39:25.392Z",
                    "programId": "program-dedccd921d074f1eb6c0f75fc5ebee72"
                }
                ],
                "pagination":{
                "count": 1,
                "limit": 100,
                "maxLimit": 1000,
                "offset": 0,
                "totalCount": 1
                }          
            }


### Show ValueStore [GET /cards/{cardId}/valueStores/{valueStoreId}]
+ Parameters
    + cardId (string, required)
    + valueStoreId (string, required)

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    
+ Response 200
    + valueStore (ValueStore)

    + Body

            {
                "valueStore":{
                    "cardId": "card-9991e39b4526401f85a889cb20b1a465",
                    "valueStoreId": "value-0ff47ca176eb4708b08a8bb010584704",
                    "valueStoreType": "PRINCIPAL",
                    "currency": "XXX",
                    "dateCreated": "2017-06-05T15:39:25.392Z",
                    "programId": "program-dedccd921d074f1eb6c0f75fc5ebee72"
                }
            }


### Add ValueStore [POST /cards/{cardId}/valueStores]
Adds a ValueStore from a Promotion Program to a Card. 

---
+ Parameters
    + cardId (string, required)

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
        + currency (string, required) - The three-character ISO-4217 currency.
        + programId (string, required) - The unique id of the Promotion Program. Note the Program's `ValueStoreType` must be of type `ATTACHED`.
        + expires (string, optional) - Defaults to lifespan set by program.
        + startDate (string, optional) - The date for which the ValueStore will become useable.
        + initialValue (number, optional) - The initial value of the ValueStore.
        
    + Body 
    
            {
            "userSuppliedId":"promo1",
            "currency": "XXX",
            "programId":"program-d0c6328cacde469ba3b28105e9d89c04",
            "initialValue":150
            }
    
+ Response 200
    + Attributes
        + valueStore (ValueStore)

    + Body

            {
                "valueStore":{
                    "cardId": "card-9991e39b4526401f85a889cb20b1a465",
                    "valueStoreId": "value-806116e736854136867d17fbfb959382",
                    "valueStoreType": "ATTACHED",
                    "currency": "XXX",
                    "dateCreated": "2017-06-05T16:14:58.314Z",
                    "programId": "program-d0c6328cacde469ba3b28105e9d89c04",
                    "expires": "2017-06-13T06:59:59.000Z"
                }   
            }


### Freeze ValueStore [POST /cards/{cardId}/valueStores/{valueStoreId}/freeze]
Freeze a Card's ValueStore, preventing all transactions against that ValueStore until unfrozen. 

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + valueStoreId (string, required) - The Lightrail ValueStore ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required) - Unique idempotency ID for the request.
        
    + Body 
    
            {
                "userSuppliedId":"freeze-valuestore-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)


    + Body

            {
                "transaction":{
                    "transactionId": "transaction-3f96b36c9b2845e2b3e61e3d74d955e1",
                    "value": -50,
                    "userSuppliedId": "freeze-valueStore-1",
                    "dateCreated": "2017-06-05T16:36:56.321Z",
                    "transactionType": "FREEZE",
                    "transactionAccessMethod": "CARDID",
                    "valueAvailableAfterTransaction": 0,
                    "giftbitUserId": "user-5022fccf827647ee9cfb63b779d62193-TEST",
                    "codeLastFour": "NKNA",
                    "cardId": "card-fb8e53946e784280a2bc8273dcbb0fda",
                    "currency": "XXX"
                }
            }


### Unfreeze ValueStore [POST /cards/{cardId}/valueStores/{valueStoreId}/unfreeze]
Unfreeze a Card's frozen ValueStore, re-enabling the creation of transactions against that ValueStore.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + valueStoreId (string, required) - The Lightrail ValueStore ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
        
    + Body 
    
            {
                "userSuppliedId":"unfreeze-valuestore-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
                "transaction":{
                "transactionId": "transaction-f7726f6b6d52412980544ee217894c82",
                "value": 50,
                "userSuppliedId": "unfreeze-valueStore-1",
                "dateCreated": "2017-06-05T16:37:57.218Z",
                "transactionType": "UNFREEZE",
                "transactionAccessMethod": "CARDID",
                "valueAvailableAfterTransaction": 50,
                "giftbitUserId": "user-5022fccf827647ee9cfb63b779d62193-TEST",
                "codeLastFour": "NKNA",
                "cardId": "card-fb8e53946e784280a2bc8273dcbb0fda",
                "currency": "XXX"
                }
            }


### Cancel ValueStore [POST /cards/{cardId}/valueStores/{valueStoreId}/cancel]
Permanently cancels a Card's ValueStore.

---
+ Parameters
    + cardId (string, required) - The Lightrail Card ID.
    + valueStoreId (string, required) - The Lightrail ValueStore ID.

+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>

    + Attributes
        + userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
        
    + Body 
    
            {
                "userSuppliedId":"cancel-valuestore-1"
            }
    
+ Response 200
    + Attributes
        + transaction (Transaction)

    + Body

            {
                "transaction":{
                "transactionId": "transaction-7f9c5dea0be24f57a902568a23a11c89",
                "value": -50,
                "userSuppliedId": "cancel-valueStore-1",
                "dateCreated": "2017-06-05T16:39:06.679Z",
                "transactionType": "CANCELLATION",
                "transactionAccessMethod": "CARDID",
                "valueAvailableAfterTransaction": 0,
                "giftbitUserId": "user-5022fccf827647ee9cfb63b779d62193-TEST",
                "codeLastFour": "NKNA",
                "cardId": "card-fb8e53946e784280a2bc8273dcbb0fda",
                "currency": "XXX"
                }
            }


## Categories [/categories/]
These endpoints are used to retrieve information about your existing Categories.

---

### List Categories [GET /categories]
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + categories (array[Category])

    + Body

            {
                "categories":[
                {
                "categoryId": "category-46f7808dffa54c8ba0da83d448ead415",
                "key": "giftbit_program",
                "value": "program-dedccd921d074f1eb6c0f75fc5ebee72"
                },
                {
                "categoryId": "category-da55afb0c007420e9417e38bc47ca8d9",
                "key": "giftbit_order",
                "value": "2017-06-05"
                }
                ],
                "pagination":{
                "count": 2,
                "limit": 100,
                "maxLimit": 1000,
                "offset": 0,
                "totalCount": 2
                }
            }


### List Categories on Card [GET /cards/{cardId}/categories]
+ Parameters
    + cardId (string, required)
    
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    
+ Response 200
    + Attributes
        + categories (array[Category])

    + Body

            {
                "categories":[
                    {
                    "categoryId": "category-46f7808dffa54c8ba0da83d448ead415",
                    "key": "giftbit_program",
                    "value": "program-dedccd921d074f1eb6c0f75fc5ebee72"
                    },
                    {
                    "categoryId": "category-da55afb0c007420e9417e38bc47ca8d9",
                    "key": "giftbit_order",
                    "value": "2017-06-05"
                    }
            }


## Programs [/programs/]
This endpoint can be used to retrieve information about your existing Gift Card and Promotion Programs.

---
### List Programs [GET /programs{?limit}{?offset}]
+ Request (application/json)
    + Headers
    
            Authorization: Bearer <YOUR_ACCESS_TOKEN>
    + Parameter
        + limit (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
        + offset (number, optional) - For pagination. The maximum number of results to return at once. Default 100.
+ Response 200

    + Body

            {
                "programs":[
                {
                "programId": "program-dedccd921d074f1eb6c0f75fc5ebee72",
                "userSuppliedId": "568fa33b-5cb34b3c-5c2bccf2",
                "name": "Gift Card Program 2017",
                "active": true,
                "currency": "USD",
                "dateCreated": "2017-06-05T17:09:10.000Z",
                "programExpiresDate": null,
                "programStartDate": "2017-06-05T15:51:35.000Z",
                "codeActivePeriodInDays": null,
                "codeValueMin": 100,
                "codeValueMax": 50000,
                "fixedCodeValues": null,
                "codeEngine": "SIMPLE_STORED_VALUE",
                "codeConfig": "GFBT",
                "valueStoreType": "PRINCIPAL",
                "metadata": null,
                "timeZone": "PST",
                "cardType": null
                },
                {
                "programId": "program-d0c6328cacde469ba3b28105e9d89c04",
                "userSuppliedId": "527650f4-6b903ce9-4c792cf6",
                "name": "Spring Promotion 2017",
                "active": true,
                "currency": "XXX",
                "dateCreated": "2017-06-05T16:14:46.000Z",
                "programExpiresDate": null,
                "programStartDate": "2017-06-05T15:51:35.000Z",
                "codeActivePeriodInDays": 7,
                "codeValueMin": 1,
                "codeValueMax": 5000,
                "fixedCodeValues": null,
                "codeEngine": "SIMPLE_STORED_VALUE",
                "codeConfig": "GFBT",
                "valueStoreType": "ATTACHED",
                "metadata": null,
                "timeZone": "PST",
                "cardType": null
                }
                ],
                "pagination":{
                "count": 2,
                "limit": 100,
                "maxLimit": 1000,
                "offset": 0,
                "totalCount": 2
                }
            }


# Data Structures

## Card (object)
+ cardId (string) - The Lightrail Card ID.
+ userSuppliedId (string) - Endpoint-unique idempotency ID provided by the client.
+ contactId (string) - Lightrail assigned Contact identifier.
+ dateCreated (string) - Lightrail system time of creation in ISO-8601 format.
+ categories (Category) - A key-value object to store additional information about the Card. For example: `"categories": {"city": "san francisco", "special": "earlybird"}` 
+ cardType (string) - `ACCOUNT_CARD`, `GIFT_CARD`.
+ currency (string) - The three-character ISO-4217 currency.

## ValueStore (object)
+ cardId (string) - The Lightrail Card ID.
+ valueStoreId (string) - The Lightrail ValueStore ID.
+ valueStoreType (string) - `PRINCIPAL`, `ATTACHED`
+ currency (string) - The three-character ISO-4217 currency.
+ dateCreated (string) - Lightrail system time of the creation in ISO-8601 format.
+ programId (string) - The Lightrail Program ID.
+ expires (string) - The date when the ValueStore expires in ISO-8601 format.
+ startDate (string, optional) - The date for which the ValueStore will become useable.

## Fullcode (object)
+ code (string, required) - The unique unguessable gift code.
+ pin (string, optional) - Card Programs can be configured to generate a pin to be used with the code.

## Transaction (object)
+ transactionId (string) - The Lightrail Transaction ID.
+ value (number) - The value of the transaction. Can be negative or positive. Note, value is in the smallest unit for the currency. If USD, then value represents cents.
+ userSuppliedId (string) - Endpoint-unique idempotency ID provided by the client.
+ dateCreated (string) - Lightrail system time of the creation in ISO-8601 format.
+ transactionType (string) - The type of the Transaciton: `DRAWDOWN, FUND, INITIAL_VALUE, CANCELLATION, INACTIVATE, ACTIVATE, FREEZE, UNFREEZE, PENDING_CREATE, PENDING_VOID, PENDING_CAPTURE, DRAWDOWN_REFUND`.
+ transactionAccessMethod (string) - Indicates how the transaction was created. Either through the cardId or the fullcode. Possible values: ["CARDID", "RAWCODE"].
+ valueAvailableAfterTransaction (string) - Deprecated. Indicates the value available on the Card after the Transaction. 
+ giftbitUserId (string) - Deprecated.  
+ cardId (string) - The Lightrail Card ID.
+ currency (string) - The three-character ISO-4217 currency.
+ metadata (object) - A key-value object to store additional information about the transaction. Note Lightrail's Redemption Rules operate on Transaction Metadata to determine whether a particular promotion can be spent. Example: `"metadata":{"checkout-cart":{"items":[{"id":"1"},{"id":"2"}]}}`. Also note, the `giftbit_*` namespace for keys is reserved.

## Pagination (object)
+ count (number) - The number of results.
+ offset (number) - The offset of the first results in the total results. Default 0.
+ limit (number) - The maximum number of results to return at once. Default 100.
+ maxLimit (number) - The maximum allowed limit that can be returned in a single page.
+ totalCount (number) - The total number of objects that matched the query.

## Code (object)
+ currency (string, required) - The three-character ISO-4217 currency.
+ initialValue (number, required) - The value of the code at creation in the smallest currency unit 
  (such as cents).
+ programId (string, required) - The id of the Program for which the code will be created from.
+ expires (string, optional) - The code's expiry date and time.  This field will be ommitted if the 
  code was not created with an expiry.  ISO-8601 format.
+ startDate (string, optional) - The code's start date and time.  This field will be ommitted if 
  the code was not created with an start date.  ISO-8601 format.
+ metadata (object) - A collection of key/value pairs of additional information about the code. 
  The `giftbit_*` namespace for keys is reserved. 

## ValueStoreBalance (object)
+ currentValue (number) - The current value of the ValueStore.
+ state (string) - `ACTIVE`, `INACTIVE`, `NOT_STARTED`, `EXPIRED`, `FROZEN`, `CANCELLED`
+ expires (string, optional) - The date when the ValueStore expires in ISO-8601 format.
+ startDate (string, optional) - The date for which the ValueStore will become useable in ISO-8601 format.
+ programId (string) - The Lightrail Program ID.
+ valueStoreId (string) - The Lightrail ValueStore ID.

## Balance (object)
+ principal (ValueStoreBalance)
+ attached (array[ValueStoreBalance])
+ currency (string) - The three-character ISO-4217 currency.
+ cardType (string) - `ACCOUNT_CARD`, `GIFT_CARD`.
+ balanceDate (string) - The time the balance was checked.
+ cardId (string) - The Lightrail Card ID.

## Category (object)
+ categoryKey (string) - The key of the category. Examples: "city", "special"
+ categoryValue (string) - The value of the category. Examples: "san francisco", "earlybird"

## Contact (object)
+ contactId (string, required) - The Lightrail Contact ID.
+ userSuppliedId (string, required) - Endpoint-unique idempotency ID provided by the client.
+ dateCreated (string) - Lightrail system time of creation in ISO-8601 format.
+ email (string, optional) - The contact's email.
+ firstName (string, optional) - The contact's first name.
+ lastName (string, optional) - The contact's last name.

## Metadata (object)
+ A collection of key/value pairs of additional information about the transaction. The `giftbit_*` namespace for keys is reserved.
