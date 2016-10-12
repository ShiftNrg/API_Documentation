# API_Documentation

##Introduction

Shift client API. All API endpoints are relative to the /api prefix.

All endpoints will return:

- Success parameter. true or false dependent on success.
- Error parameter. Provided when response is unsuccessful.

The API is only available after the client has successfully loaded,<br /> otherwise all endpoints will return:

    {
      "success" : false,
      "error" : "loading"
    }


In the case the client is not fully synced all routes may return intermediate/old values.

Each API entry contains an example call to help provide understanding of how to use the call.
These examples rely on curl being installed and Shift running on the localhost.
The examples also include ```<field>;``` use this for easy identification of what needs to be changed for the call to function.

##Accounts

API calls related to Account functionality.

###Open account

Request information about an account.

POST /api/accounts/open

**Request**

    {
      "secret": "secret key of account"
    }
**Response**

    {
      "success": true,
      "account": {
      "address": "Address of account. String",
      "unconfirmedBalance": "Unconfirmed balance of account. Integer",
      "balance": "Balance of account. Integer",
      "publicKey": "Public key of account. Hex",
      "unconfirmedSignature": "If account enabled second signature, but it's still not confirmed. Boolean: true or false",
      "secondSignature": "If account enabled second signature. Boolean: true or false",
      "secondPublicKey": "Second signature public key. Hex",
      "username": "Username of account."
      }
    }
**Example**
    
    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"secret":"<INSERT SECRET HERE>"}' \
    http://localhost:9305/api/accounts/open

###Get balance

Request the balance of an account.

GET /api/accounts/getBalance?address=<address>

address: wallet address of the account

**Response**
    
    {
      "success": true,
      "balance": "Balance of account",
      "unconfirmedBalance": "Unconfirmed balance of account"
    }
**Example**
    
    curl -k -X GET http://localhost:9305/api/accounts/getBalance?address=<address>

###Get account public key

Get the public key of an account. If the account does not exist the API call will return an error.

GET /api/accounts/getPublicKey?address=address

address: wallet address of the account

**Response**
    
    {
      "success": true,
      "publicKey": "Public key of account. Hex"
    }

**Example**
    
    curl -k -X GET http://localhost:9305/api/accounts/getPublicKey?address=<address>

###Generate public key

Returns the public key of the provided secret key.

POST /api/accounts/generatePublicKey

**Request**
    
    {
      "secret": "secret key of account"
    }
**Response**
    
    {
      "success": true,
      "publicKey": "Public key of account. Hex"
    }
**Example**
    
    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"secret":"<INSERT SECRET HERE>"}' \
    http://localhost:9305/api/accounts/generatePublicKey

###Get account

Returns account information of an address.

GET /api/accounts?address=address

address: wallet address of an account

**Response**
    
    {
      "success": true,
      "account": {
      "address": "Address of account. String",
      "unconfirmedBalance": "Unconfirmed balance of account. Integer",
      "balance": "Balance of account. Integer",
      "publicKey": "Public key of account. Hex",
      "unconfirmedSignature": "If account enabled second signature, but it's still not confirmed. Boolean: true or false",
      "secondSignature": "If account enabled second signature. Boolean: true or false",
      "secondPublicKey": "Second signature public key. Hex"
      }
    }

**Example**
    
    curl -k -X GET http://localhost:9305/api/accounts?address=<address>

###Get delegates

Returns delegate accounts by address.

GET /api/accounts/delegates?address=address

address: wallet address of account

**Response**

    {
      "success": true,
      "delegates": [array]
    }
- Delegates Array includes: delegateId, address, publicKey, vote (# of votes), producedBlocks, missedBlocks, rate, productivity

**Example**

    curl -k -X GET http://localhost:9305/api/accounts/delegates?address=<address>

###Put delegates

Vote for the selected delegates. Maximum of 33 delegates at once.

PUT /api/accounts/delegates

**Request**

    {
      "secret" : "Secret key of account",
      "publicKey" : "Public key of sender account, to verify secret passphrase in wallet. Optional, only for UI",
      "secondSecret" : "Secret key from second transaction, required if user uses second signature",
      "delegates" : "Array of string in the following format: ["+DelegatePublicKey"] OR ["-DelegatePublicKey"]. Use + to UPvote, - to DOWNvote"
    }

**Response**

    {
      "success": true,
      "transaction": {object}
    }

**Example - No Second Secret**
    
    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","publicKey"="<INSERT PUBLICKEY HERE>","delegates":["<INSERT DELEGATE PUBLICKEY HERE>"]}' \
    http://localhost:9305/api/accounts/delegates
    
**Example - With Second Secret**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","publicKey"="<INSERT PUBLICKEY HERE>",secondSecret"="<INSERT SECONDSECRET HERE>,"delegates":["<INSERT DELEGATE PUBLICKEY HERE>"]}' \
    http://localhost:9305/api/accounts/delegates
    
**Example - Multiple Votes**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","publicKey"="<INSERT PUBLICKEY HERE>","delegates":["<INSERT DELEGATE PUBLICKEY HERE>","<INSERT DELEGATE PUBLICKEY HERE>"]}' \
    http://localhost:9305/api/accounts/delegates
    
##Loader

Provides the synchronization and loading information of a client. These API calls will only work if the client is syncing or loading.

###Get loading status

Returns the status of the blockchain

GET /api/loader/status

**Response**

    {
       "success": true,
       "loaded": "Is blockchain loaded? Boolean: true or false",
       "now": "Last block loaded during loading time. Integer",
       "blocksCount": "Total blocks count in blockchain at loading time. Integer"
    }

**Example**
    
    curl -k -X GET http://localhost:9305/api/loader/status/
    
###Get synchronization status

Get the synchronization status of the client.

GET /api/loader/status/sync

**Response**

    {
       "success": true,
       "syncing": "Is wallet is syncing with another peers? Boolean: true or false",
       "blocks": "Number of blocks remaining to sync. Integer",
       "height": "Total blocks in blockchain. Integer"
    }

**Example**
    
    curl -k -X GET http://localhost:9305/api/loader/status/sync
    
##Transactions

API calls related to transactions.

###Get list of transactions

List of transactions matched by provided parameters.

    GET /api/transactions?blockId=blockId&senderId=senderId&recipientId=recipientId&limit=limit&offset=offset&orderBy=field
    
- blockId: Block id of transaction. (String)
- senderId: Sender address of transaction. (String)
- recipientId: Recipient of transaction. (String)
- limit: Limit of transaction to send in response. Default is 20. (Number)
- offset: Offset to load. (Integer number)
- orderBy: Name of column to order. After column name must go "desc" or "acs" to choose -- - order type, prefix for column name is t_. Example: orderBy=t_timestamp:desc (String)

All parameters join by "OR".

**Example:**
    /api/transactions?blockId=10910396031294105665&senderId=6881298120989278452C&orderBy=timestamp:desc looks like: blockId=10910396031294105665 OR senderId=6881298120989278452C
    
**Response**

{
  "success": true,
  "transactions": [
  "list of transactions objects"
  ]
}

**Example - blockId**

    curl -k -X GET http://localhost:9305/api/transactions?blockId=<blockId>
    
**Example - senderId**

    curl -k -X GET http://localhost:9305/api/transactions?senderId=<senderId>
    
**Example - senderId**

    curl -k -X GET http://localhost:9305/api/transactions?recipientId=<recipientId>
    
###Send transaction

Send transaction to broadcast network.

PUT /api/transactions

**Request**

    {
    "secret" : "Secret key of account",
    "amount" : /* Amount of transaction * 10^8. Example: to send 1.1234 LISK, use 112340000 as amount */,
    "recipientId" : "Recipient of transaction. Address or username.",
    "publicKey" : "Public key of sender account, to verify secret passphrase in wallet. Optional, only for UI",
    "secondSecret" : "Secret key from second transaction, required if user uses second signature"
    }

**Response**

    {
      "success": true,
      "transactionId": "id of added transaction"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","amount":<INSERT AMOUNT HERE>,"recipientId":"<INSERT WALLET ADDRESS HERE>"}' \
    http://localhost:9305/api/transactions

**Example - Second Secret**
    
    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","secondSecret":"<INSERT SECOND SECRET HERE>",
    "amount":<INSERT AMOUNT HERE>,"recipientId":"<INSERT WALLET ADDRESS HERE>"}' \
    http://localhost:9305/api/transactions
    
###Get transaction

Get transaction that matches the provided id.

GET /api/transactions/get?id=id

id: String of transaction (String)

**Response**

    {
      "success": true,
      "transaction": {
      "id": "Id of transaction. String",
      "type": "Type of transaction. Integer",
      "subtype": "Subtype of transaction. Integer",
      "timestamp": "Timestamp of transaction. Integer",
      "senderPublicKey": "Sender public key of transaction. Hex",
      "senderId": "Address of transaction sender. String",
      "recipientId": "Recipient id of transaction. String",
      "amount": "Amount. Integer",
      "fee": "Fee. Integer",
      "signature": "Signature. Hex",
      "signSignature": "Second signature. Hex",
      "companyGeneratorPublicKey": "If transaction was sent to merchant, provided comapny generator public key to find company. Hex",
      "confirmations": "Number of confirmations. Integer"
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/transactions/get?id=<id>

###Get unconfirmed transaction

Get unconfirmed transaction that matches the provided id.

GET /api/transactions/unconfirmed/get?id=id

id: String of transaction (String)

**Response**

    {
      "success": true,
      "transaction": {
      "id": "Id of transaction. String",
      "type": "Type of transaction. Integer",
      "subtype": "Subtype of transaction. Integer",
      "timestamp": "Timestamp of transaction. Integer",
      "senderPublicKey": "Sender public key of transaction. Hex",
      "senderId": "Address of transaction sender. String",
      "recipientId": "Recipient id of transaction. String",
      "amount": "Amount. Integer",
      "fee": "Fee. Integer",
      "signature": "Signature. Hex",
      "signSignature": "Second signature. Hex",
      "confirmations": "Number of confirmations. Integer"
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/transactions/unconfirmed/get?id=<id>
