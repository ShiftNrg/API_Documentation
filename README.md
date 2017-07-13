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

GET /api/accounts/getBalance?address=address

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
    "amount" : /* Amount of transaction * 10^8. Example: to send 1.1234 SHIFT, use 112340000 as amount */,
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

###Get list of unconfirmed transactions

Gets a list of unconfirmed transactions.

GET /api/transactions/unconfirmed

**Response**

{
    "success" : true,
    "transactions" : [list of transaction objects]
}

**Example**

    curl -k -X GET http://localhost:9305/api/transactions/unconfirmed

##Peers

Peers API.

###Get peers list

Gets list of peers from provided filter parameters.

GET /api/peers?state=state&os=os&version=version&limit=limit&offset=offset&orderBy=orderBy

- state: State of peer. 1 - disconnected. 2 - connected. 0 - banned. (String)
- os: OS of peer. (String)
- version: Version of peer. String)
- limit: Limit to show. Max limit is 100. (Integer)
- offset: Offset to load. (Integer)
- orderBy: Name of column to order. After column name must go "desc" or "acs" to choose order type. (String)

All parameters joins by "OR".

**Example:**

    /api/peers?state=1&version=0.3.2 looks like: state=1 OR version=0.3.2

**Response**

    {
      "success": true,
      "peers": [
      "list of peers"
      ]
    }

**Example**
    
    curl -k -X GET http://localhost:9305/api/peers
    
###Get peer

Gets peer by IP address and port

GET /api/peers/get?ip=ip&port=port

- ip: Ip of peer. (String)
- port: Port of peer. (Integer)

**Response**

    {
      "success": true,
      "peer": "peer object"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/peers/get?ip=<ip>&port=<port>

###Get peer version, build time

Gets a list peer versions and build times


    GET /api/peers/version

**Response**

    {
      "success": true,
      "version": "version of Shift",
      "build": "time of build"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/peers/version
    
##Blocks

Blocks management API.

###Get block

Gets block by provided id.


    GET /api/blocks/get?id=id

- id: Id of block.

**Response**

    {
    "success": true,
    "block": {
      "id": "Id of block. String",
      "version": "Version of block. Integer",
      "timestamp": "Timestamp of block. Integer",
      "height": "Height of block. Integer",
      "previousBlock": "Previous block id. String",
      "numberOfRequests": "Not using now. Will be removed in 0.2.0",
      "numberOfTransactions": "Number of transactions. Integer",
      "numberOfConfirmations": "Not using now.",
      "totalAmount": "Total amount of block. Integer",
      "totalFee": "Total fee of block. Integer",
      "payloadLength": "Payload length of block. Integer",
      "requestsLength": "Not using now. Will be removed in 0.2.0",
      "confirmationsLength": "Not using now.,
      "payloadHash": "Payload hash. Hex",
      "generatorPublicKey": "Generator public key. Hex",
      "generatorId": "Generator id. String.",
      "generationSignature": "Generation signature. Not using. Will be removed in 0.2.0",
      "blockSignature": "Block signature. Hex"
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/get?id=<id>

###Get blocks

Gets all blocks by provided filter(s).

    GET /api/blocks?generatorPublicKey=generatorPublicKey&height=height&previousBlock=previousBlock&totalAmount=totalAmount&totalFee=totalFee&limit=limit&offset=offset&orderBy=orderBy

All parameters joins by OR.

**Example:**

    /api/blocks?height=100&totalAmount=10000 looks like: height=100 OR totalAmount=10000

- totalFee: total fee of block. (Integer)
- totalAmount: total amount of block. (Integer)
- previousBlock: previous block of need block. (String)
- height: height of block. (Integer)
- generatorPublicKey: generator id of block in hex. (String)
- limit: limit of blocks to add to response. Default to 20. (Integer)
- offset: offset to load blocks. (Integer)
- orderBy: field name to order by. Format: fieldname:orderType. Example: height:desc, ----  timestamp:asc (String)

**Response**

    {
      "success": true,
      "blocks": [
      "array of blocks"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks?generatorPublicKey=<generatorPublicKey>

###Get blockchain fee

Get transaction fee for sending "normal" transactions.

    GET /api/blocks/getFee

**Response**

    {
      "success": true,
      "fee": Integer
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getFee

###Get blockchain fees schedule

Get transaction fee for all types of transactions.

    GET /api/blocks/getFees

**Response**
    
    {
      "success": true,
      "fees":{
        "send": Integer,
        "vote": Integer,
        "secondsignature": Integer,
        "delegate": Integer,
        "multisignature": Integer,
        "dapp": Integer
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getFees

###Get blockchain reward schedule

Gets the forging reward for blocks.

    GET /api/blocks/getReward

**Response**

    {
      "success": true,
      "reward": Integer
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getReward

###Get supply of available Shift

Gets the total amount of Shift in circulation

    GET /api/blocks/getSupply

**Response**
    
    {
      "success": true,
      "supply": Integer
    }
    
**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getSupply

###Get blockchain height

Gets the blockchain height of the client.

    GET /api/blocks/getHeight

**Response**

    {
      "success": true,
      "height": "Height of blockchain. Integer"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getHeight

###Gets status of height, fee, milestone, blockreward and supply

Gets status of height, fee, milestone, blockreward and supply

    GET /api/blocks/getStatus

**Response**

    {
      "success": true,
      "height": Integer
      "fee": Integer
      "milestone": Integer
      "reward": Integer
      "supply": Integer
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getStatus

###Get blockchain nethash

Gets the nethash of the blockchain on a client.

    GET /api/blocks/getNethash

**Response**

    {
      "success": true,
      "nethash": "Nethash of the Blockchain. String"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getNethash

###Get blockchain milestone

Gets the milestone of the blockchain on a client.

    GET /api/blocks/getMilestone

**Response**

    {
      "success": true,
      "milestone": Integer
    }

**Example**

    curl -k -X GET http://localhost:9305/api/blocks/getMilestone

##Signatures

Signature management API.

###Get signature

Gets the second signature status of an account.

    GET /api/signatures/get?id=id

- id: Id of signature. (String)

**Response**

    {
      "success" : true,
      "signature" : {
        "id" : "Id. String",
        "timestamp" : "TimeStamp. Integer",
        "publicKey" : "Public key of signature. hex",
        "generatorPublicKey" : "Public Key of Generator. hex",
        "signature" : [array],
        "generationSignature" : "Generation Signature"
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/signatures/get?id=<id>

###Add second signature

Add a second signature to an account.

    PUT /api/signatures

**Request**

    {
      "secret": "secret key of account",
      "secondsecret": "second secret key of account",
      "publicKey": "optional, to verify valid secret key and account"
    }

**Response**

    {
      "success": true,
      "transactionId": "id of transaction with new signature",
      "publicKey": "Public key of signature. hex"
    }

**Example**
    
    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","secondSecret":"<INSERT SECOND SECRET HERE>","publicKey":"<INSERT PUBLIC KEY HERE>" }' \
    http://localhost:9305/api/signatures

##Delegates

Delegates API.

###Enable delegate on account

Puts request to create a delegate.

    PUT /api/delegates

**Request**

    {
      "secret": "Secret key of account",
      "secondSecret": "Second secret of account",
      "username": "Username of delegate. String from 1 to 20 characters."
    }

**Response**

    {
      "success": true,
      "transaction": "transaction object"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","secondSecret":"<INSERT SECOND SECRET HERE>","username":"<INSERT USERNAME HERE>" }' \
    http://localhost:9305/api/delegates

###Get delegates list

Gets list of delegates by provided filter.

    GET /api/delegates?limit=limit&offset=offset&orderBy=orderBy

- limit: Limit to show. Integer. Maximum is 100. (Integer)
- offset: Offset (Integer)
- orderBy: Order by field (String)

**Response**

    {
      "success": true,
      "delegates": "delegates objects array"
    }

- Delegates Array includes: delegateId, address, publicKey, vote (# of votes), producedBlocks, missedBlocks, rate, productivity

**Example**

    curl -k -X GET http://localhost:9305/api/delegates?limit=<limit>

###Get delegate

Gets delegate by public key or username.

    GET /api/delegates/get?publicKey=publicKey

- publicKey: Public key of delegate account (String)


	GET /api/delegates/get?username=username


- username: Username of delegate account (String)

**Response**

    {
      "success": true,
      "delegate": {
       "username": "Username. String",
       "address": "Address. String",
       "publicKey": "Public key. String",
       "vote": "Total votes. Integer",
       "producedblocks": "Produced blocks. Integer",
       "missedblocks": "Missed blocks. Integer",
       "rate": "Ranking. Integer",
       "approval": "Approval percentage. Float",
       "productivity": "Productivity percentage. Float"
      }
    }

**Example**

    curl -k -X GET http://localhost:9305/api/delegates/get?publicKey=publicKey

###Get delegates count

Get total count of registered delegates.

    GET /api/delegates/count

**Response**

    {
      "success": true,
      "count": 101
    }

**Example**

    curl -k -X GET http://localhost:9305/api/delegates/count

###Get votes of account

Get votes by account wallet address.

    GET /api/accounts/delegates/?address=address

- address: Address of account. (String)

**Response**

    {
      "success": true,
      "delegates": "array of delegates"
    }

- Delegates Array includes: delegateId, address, publicKey, vote (# of votes), producedBlocks, missedBlocks, rate, productivity

**Example**

    curl -k -X GET http://localhost:9305/api/accounts/delegates/?address=<address>

###Get voters

Get voters of delegate.

    GET /api/delegates/voters?publicKey=publicKey

- publicKey: Public key of registered delegate account. (String)

**Response**

    {
      "success": true,
      "accounts": [
        "array of accounts who vote for delegate"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/delegates/voters?publicKey=<publicKey>

###Enable forging on delegate

Enables forging for a delegate on the client node.

    POST /api/delegates/forging/enable

**Request**

    {
      "secret": "secret key of delegate account"
    }

**Response**

    {
      "success": true,
      "address": "address"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"secret":"<INSERT SECRET HERE>"}' \
    http://localhost:9305/api/delegates/forging/enable

###Disable forging on delegate

Disables forging for a delegate on the client node.

    POST /api/delegates/forging/disable

**Request**

    {
      "secret": "secret key of delegate account"
    }

**Response**

    {
      "success": true,
      "address": "address"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"secret":"<INSERT SECRET HERE>"}' \
    http://localhost:9305/api/delegates/forging/disable

###Get forged by account

Get amount of Shift forged by an account.

    GET /api/delegates/forging/getForgedByAccount?generatorPublicKey=generatorPublicKey

- generatorPublicKey: generator id of block in hex. (String) [Required]
- start: Sets the start time of the search - timestamp UNIX time. (String) [Optional]
- end: Sets the endtime of the search - timestamp UNIX time. (String) [Optional]

**Response**

    {
      "success": true,
      "sum": "Forged amount. Integer"
    }

**Example**
  
    curl -k -X GET http://localhost:9305/api/delegates/forging/getForgedByAccount?generatorPublicKey=<generatorPublicKey>
    

##Apps

Blockchain Applications API.

##Apps

Registers a Blockchain Application.

    PUT /api/dapps

**Request**

    {
      "secret": "Secret of account. String. Required",
      "secondSecret": "Second secret of account. String. Optional",
      "publicKey": "Public key to verify sender secret key. Hex. Optional",
      "category": "DApp category. Integer. Required",
      "name": "DApp name. String. Required",
      "description": "DApp description. String. Optional",
      "tags": "DApp tags. String. Optional",
      "type": "DApp type. Integer. Required (Only type 0 is currently supported)",
      "link": "Link to DApp file. ZIP supported. String. Required",
      "icon": "Link to icon file. PNG and JPG/JPEG supported. String. Optional"
    }

**Response**

    {
      "success": true,
      "transactionId": "transaction id"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","secondSecret":"<INSERT SECOND SECRET HERE>"
    "category":<INSERT INTEGER HERE>,"name":"<INSERT APPLICATION NAME HERE>",
    "type":0,"link":"<INSERT LINK TO APPLICATION HERE>"}' \
    http://localhost:9305/api/dapps

###Get apps

Gets a list of Blockchain Applications registered on the network.


    GET /api/dapps?category=category&name=name&type=type&link=link&limit=limit&offset=offset&orderBy=orderBy

- category: DApp category. (Integer)
- name: DApp name. (String)
- type: DApp type. (Integer)
- link: DApp link. (String)
- limit: Query limit. Maximum is 100. (Integer)
- offset: Query offset. (Integer)
- orderBy: Order by field. (String)

**Response**

    {
      "success": true,
      "dapps": "array of dapps"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps?name=<INSERT APPLICATION NAME HERE>

###Get app

Gets a specific Blockchain Application by registered id.

    GET /api/dapps/get?id=id

- id: Id of app

**Response**

    {
      "success": true,
      "dapp": "dapp object"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/get?id=<id>

###Search for apps

Searches for Blockchain Applications by filter(s) on a node.

    GET /api/dapps/search?q=q&category=category&installed=installed

- q: Search criteria. (String)
- category: Category to search within. (Integer)
- installed: Search installed apps only. 1 or 0. (Integer)

**Response**

    {
      "success": true,
      "dapps": [
        "array of dapps"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/search?installed=1

###Install app

Installs a app by id on the node.

    POST /api/dapps/install

**Request**

    {
      "id": "Blockchain Application id to install"
    }

**Response**

    {
      "success": true,
      "path": "dppp install path"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"id":"<INSERT ID HERE>"}' \
    http://localhost:9305/api/dapps/install

###Installed apps

Returns a list of installed apps on the requested node.

    GET /api/dapps/installed

**Response**

    {
      "success": "true",
      "dapps": [
        "array of dapps"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/installed

###Installed apps Ids

Returns a list of installed app ids on the requested node.

    GET /api/dapps/installedIds

**Response**

    {
      "success": true,
      "ids": [
        "array of dapp ids"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/installedIds

###Uninstall apps

Uninstalls a app by id from the requested node.

    POST /api/dapps/uninstall

**Request**

    {
      "id": "Blockchain Application id to install"
    }

**Response**

    {
      "success": true
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"id":"<INSERT ID HERE>"}' \
    http://localhost:9305/api/dapps/uninstall

###Launch app

Launches a app by id on the requested node.

    POST /api/dapps/launch

**Request**

    {
      "id": "dapp id to launch",
      "params": "dapp launch params, not required, array"
    }

**Response**

    {
      "success": true
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"id":"<INSERT ID HERE>"}' \
    http://localhost:9305/api/dapps/launch

###Installing

Returns a list of app ids currently being installed on the requested node.

    GET /api/dapps/installing

**Response**

    {
      "success": true,
      "installing": [
         "array of dapp ids"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/installing

###Uninstalling

Returns a list of app ids currently being uninstalled on the client node.

    GET /api/dapps/uninstalling

**Response**

    {
      "success": true,
      "uninstalling": [
         "array of dapp ids"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/uninstalling

###Launched

Returns a list of app ids which are currently launched on the client node.

    GET /api/dapps/launched

**Response**

    {
      "success": true,
      "launched": [
         "array of dapp ids"
      ]
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/launched

###Categories

Returns a full list of app categories.

    GET /api/dapps/categories

**Response**

    {
      "success": true,
      "category": "object containing category names and ids"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/dapps/categories

###Stop app

Stops a app by id on the requested node.

    POST /api/dapps/stop

**Request**

    {
      "id": "dapp id to stop"
    }

**Response**

    {
      "success": true
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"id":"<INSERT ID HERE>"}' \
    http://localhost:9305/api/dapps/stop

##Multi-Signature

Multi-signature API.

###Get pending multi-signature transactions

Returns a list of multi-signature transactions that waiting for signature by publicKey.

    GET /api/multisignatures/pending?publicKey=publicKey

- publicKey: Public key of account (String)

**Response**

    {
      "success": true,
      "transactions": ['array of transactions to sign']
    }

**Example**

    curl -k -X GET http://localhost:9305/api/multisignatures/pending?publicKey=<publicKey>

###Create multi-signature account

Create a multi-signature account.

    PUT /api/multisignatures

**Request**

    {
    "secret": "your secret. string. required.",
    "lifetime": "request lifetime in hours (1-24). required.",
    "min": "minimum signatures needed to approve a tx or a change (1-15). integer. required",
    "keysgroup": [array of public keys strings]. add '+' before publicKey to add an account or '-' to remove. required.
    }

**Response**

    {
      "success": true,
      "transactionId": "transaction id"
    }

**Example**

    curl -k -H "Content-Type: application/json" \
    -X PUT -d '{"secret":"<INSERT SECRET HERE>","lifetime":<INSERT NUMBER HERE>,"min":<INSERT NUMBER OF SIGNATURES HERE>,"keysgroup":["+<INSERT PUBLIC KEY HERE>","+<INSERT PUBLIC KEY HERE>"] }' \
    http://localhost:9305/api/multisignatures

###Sign transaction

Signs a transaction that is awaiting signature.

    POST /api/multisignatures/sign

**Request**

    {
      "secret": "your secret. string. required.",
      "publicKey": "public key of your account. string. optional.",
      "transactionId": "id of transaction to sign. REQUIRED"
    }

**Response**

    {
      "success": true,
      "transactionId": "transaction id"
    }

**Example**
    
    curl -k -H "Content-Type: application/json" \
    -X POST -d '{"secret":"<INSERT SECRET HERE>","transactionId":"<INSERT TRANSACTION ID HERE>"}' \
    http://localhost:9305/api/multisignatures/sign
    
###Get accounts of multi-signature

Gets a list of accounts that are part of a multi-signature account.

    GET /api/multisignatures/accounts?publicKey=publicKey

- publicKey: Public key of multi-signature account (String)

**Response**

    {
      "success": true,
      "accounts": "array of accounts"
    }

**Example**

    curl -k -X GET http://localhost:9305/api/multisignatures/accounts?publicKey=<publicKey>



##Credits

This API is based on the LISK API, the syntax is nearly the same.

###Informations
Initial Written by List Team<br />
Ported to Shift-Github Repo by LeChuckDE
