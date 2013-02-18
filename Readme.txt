The Hamming Internet Payment System

Copyright (C) 2012 Michael Leonhard <michael206@gmail.com>

= Abstract =

This document describes an open electronic payment system to serve as
an alternative to credit cards, checks, and wire transfers.  The
system has online banks, automated brokers, users, and client
software.  Each online bank keeps a database of accounts.  Client
software can use single-use access codes to transfer value from an
account at a bank to another account at the same bank.  To transfer
value between banks, the system works like the informal Hawala value
transfer system: automated brokers trade value at one bank for value
at another bank.  Clients, banks, and brokers communicate using the
Hamming Internet Payment Protocol 1.0, which is defined here.


= Introduction =

== Motivation ==

Many important processes in modern society have been modernized with
Internet technology: communication, publishing, broadcasting,
shopping, and order processing.

Email in particular has increased the efficiency of communication in
many ways.  Sending and receiving email requires little human effort.
Email can be prepared and transmitted automatically.  Recipients also
benefit from automated processing and routing.  Email generates no
garbage.  Email delivery is fast and requires little energy.  Because
of its low cost, email has replaced most uses of faxes and letters.
It is also used for many valuable new types of communication that were
not practical with older technology.  In short, email has increased
the efficiency of our modern society and enabled new valuable
activities.

With the Hamming Internet Payment System, we aim to increase the
efficiency of payments and value transfers.  We aim to reduce the cost
of performing transfers by creating an alternative to credit card
transactions, traditional electronic funds transfers, wire transfers,
and checks.  By reducing cost, we hope to enable many new kinds of
business and economic activies that are not practical with current
technology.


== Goals of the Payment System ==

1. Facilitate the efficient transfer of monetary value from one entity to
   another
1. Decentralized. Anyone may participate in the system in any capacity.
   Prevent the rise of gatekeepers who would impose barriers to participation.
1. Employ only commonly available and easily understood technology.  No fancy
   crypto.
1. Facilitate the types of transfers that are commonplace today:
  * person -> business
  * business -> business
1. Facilitate transfers that are impractical or economically infeasible with
   current technology:
  * micro-payments
  * person -> person
  * anonymous transfers
1. Prevent theft, fraud, harassment, and inadvertent misuse
1. Protect the privacy of participants and their activities
1. Withstand malicious attacks
1. Assist participants in complying with their local laws


== Goals of this Document ==

1. Introduce the payment system
1. Describe common use cases
  * Individual purchasing a physical product from a website
  * Business paying another business for services rendered
  * Individual purchasing a physical product in a store using a mobile phone
  * Individual purchasing access to a blog post for the price of USD 0.01
  * Individual transferring value to another individual
1. Define the protocol and expected behavior of banks, brokers, and clients
  * Performing intra-bank transfers
  * Retrieving account balances
  * Retrieving account history
  * Managing access codes
  * Performing inter-bank transfers using brokers
  * Registering as a broker
  * Finding a broker
  * Reporting broker failure/fraud
  * Chaining brokers
  * Payer-initiated transfers
  * Payee-initiated transfers
  * Handling network and server failure
1. Describe how to use the payment system with other applications and services
  * Payment forms on websites
  * HTTP payments using the "402 Payment Required" status code
  * Refundable email postage as a solution to spam
  * Automated escrow services
  * Point of sale payments with mobile phones and NFC
1. Inspire readers to help develop the system and contribute software implementations


= Payment System Overview =

The payment system works like an online version of the informal Hawala
value transfer system.  The system has these actors:

 * Bank: A bank is a software program and a database of accounts.
   Each account has a set of associated access codes.  The bank
   receives connections and processes commands to transfer value
   between accounts, check status of transfers, query accounts, and
   manage codes.  The bank also keeps a list of registered brokers,
   responds to client broker searches, and automatically investigates
   client reports of broker failure/fraud.
 
 * Broker: A broker is a software program that controls accounts on
   two banks and provides cross-bank value transfer services.  The
   broker registers itself with both banks so it will appear in broker
   search results.
 
 * User: A user is a person or organization with an account at a bank.
   A user holds the list of codes that grant access to the functions
   of their account.  The user authorizes debits and credits on the
   account.
 
 * Client: A client is a software program that acts on behalf of a
   user.  The client communicates with the bank to manage the account
   and access codes.  It also finds and engages brokers to perform
   inter-bank transfers.  It uses segmented transfers minimize risk of
   theft of transferred value by malicious brokers.  It automatically
   reports failed transfers to the banks so they can penalize bad
   brokers.

The system is decentralized.  Any computer can act as any type of
entity.  There is no central authority that can give or revoke
permission to participate in the system.

All protocol connections are encrypted with TLS version 1.2 or newer.
This helps prevent the theft of access codes and private information.
Banks and brokers accept connections and provide server certificates.

The system is designed to preserve the privacy of its users while
allowing entities to comply with their local laws.


= Hamming Internet Payment Protocol 1.0 =

The Hamming Internet Payment Protocol 1.0 specifies how clients and
brokers communicate with banks.  Banks are software programs that
accept connections, receive and process requests, and send responses
over the connection stream.

The protocol is asynchronous.  This means that the client may send
multiple requests at a time.  Responses may be returned out of order.
Additionally, some requests may generate multiple responses.


== Request and Response Structure ==

Each request or response is a JSON object, encoded in UTF-8, followed
by a single newline character.

All attribute names in the JSON object should be lower-case.

The JSON object must not contain any newline or carriage return
characters.  Attribute names and string values in the JSON object must
quote those characters as '\r' and '\n'.

Example Request:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"ping",
 "requestid":"6Eo6",
 "timestamp":13447609524.104
}\n

Example Response:
{"resultcode":200,
 "explanation":"pong",
 "requestid":"6Eo6",
 "operationid":"VzJP",
 "timestamp":13447609524.748
}\n

Mandatory request attributes:

* "protocol" : "hipp/1.0"

* "host" : a string with the host name of the bank, eg "usd.bank.com"

* "command" : a lower-case string with a command name

* "requestid": a string with a random code, with a UTF-8 encoding of
  32 bytes or less

* "timestamp" : the time of the request, represented as the number of
  seconds since the Epoch; may contain decimals

Mandatory response attributes:

* "resultcode" : an integer encoding the result of the request

* "explanation" : a string with a human-readable description of the
  result of the request

* "requestid" : the string sent by the client in the request, or null
  if the request could not be parsed

* "operationid" : a string with a random code, with a UTF-8 encoding
  of 32 bytes or less

* "timestamp" : the time the command was processed, represented as the
  number of seconds since the Epoch; may contain decimals


== Status codes ==

1xx codes are informational messages.

* 102 Update Notification.  Notifies the client of an update to a
  segmented transfer.  The "transfer" value contains a JSON object
  with the current state of the transfer.

2xx codes mean that the request was successful.

* 200 OK.  The request succeeded.

4xx codes mean that an error occurred due to the client.  Requests
returning 4xx should not be retried as-is.

* 400 Bad Request.  The request was malformed in some way.

* 404 Not Found.  The specified object was not found.

* 405 Command Not Allowed.  The server does not support the specified
  command.

* 408 Request Timeout.  The client did not transmit a request within
  the time that the server was prepared to wait. The client MAY repeat
  the request without modifications at any later time.

* 409 Conflict.  The server is refusing to process the transfer update
  because the transfer state has changed.  For example, if the
  transfer status became 'timedout' then a subsequent 'updatetransfer'
  command will return 409 Conflict.

* 414 Request Line Too Long.  The server is refusing to service the
  request because the request line is longer than the server is
  willing to interpret.

* 419 Request ID Too Long.  The server is refusing to service the
  request because the request ID string is longer than 32 bytes when
  encoded in UTF-8.

* 420 Insufficient Value.  The server is refusing to process the
  request because the source access code does not have sufficient
  value available.

* 421 Invalid Access Code.  The provided access code does not exist or
  does not allow the requested command.  If a command specified
  'source' and 'destination' access codes then this error refers to
  the 'source' access code.

* 422 Invalid Destination Access Code.  The provided destination
  access code does not exist or does not allow the requested command.
  This refers to the value of the 'destination' attribute specified in
  the request.

* 423 Request Too Old.  The server is refusing to process the request
  because the provided timestamp is too far in the past.

* 424 Protocol Version Not Supported.  The server does not support, or
  refuses to support, the specified protocol version.

* 425 Unsupported Pattern.  The server does not support, or refuses to
  support, the specified transfer pattern.

5xx codes mean the server encountered an error and is incapable of
completing the request.  The client may retry the request.

* 500 Internal Server Error.  The server encountered an unexpected
  condition which prevented it from fulfilling the request.

* 503 Service Unavailable.  The server is currently unable to handle
  the request due to a temporary overloading or maintenance of the
  server.


== Nullipotent and Idempotent Commands ==

All commands in HIPP 1.0 are either nullipotent or idempotent.
Nullipotent commands have no side-effects.  After executing a
nullipotent command, the state of the server and accounts is the same
as before the command was issued.  Idempotent commands have
side-effects.  Issuing an idempotent command multiple times has the
same effect as issuing it once.  Because all HIPP 1.0 commands are
either idempotent or nullipotent, any command may be retried with no
unwanted side-effects.  If the client fails to receive a response to a
command, it can simply issue the command again.  There is no danger of
performing a transfer twice.  The server simply returns a copy of the
result from the previous run of the command.

The protocol achieves idempotency with the 'requestid' attribute.  The
server must remember the requestid and result of each idempotent
command and return the same response if the client retries the
request.  The server may store this information forever or as long as
server resources permit.  The client must send an identical timestamp
with a retried request.  If the server receives a request with a
timestamp far in the past and it may have processed the command
and already discarded the result, then it should return a "423 Request
Too Old" response.


== Begin Transfer Command ==

HIPP/1.0 supports segmented transfers.  In a segmented transfer, value
is transferred in segments rather than all at once.  The payer
releases funds to the recipient in small chunks.  The recipient can
receive realtime notifications when each segment is released.

Segmented transfers help to lower risk when moving value using a
broker or a chain of brokers.  Each value segment makes its way
through the chain of brokers to the destination account.  If any
broker in the chain fails or cheats, the user loses only the value of
the current segment, rather than the value of the whole transfer.

Use the idempotent 'begintransfer' command to initiate a segmented
transfer of value from one account to another.

Request attributes:

* "command" : "begintransfer"

* "source" : a string with an access code that allows debits

* "destination" : a string with an access code that allows deposits

* "amount" : an integer specifying the total amount of value to move
  from the source account to the destination account in this transfer

* "releasedamount" : an integer specifying the amount of value to
  initially release to the destination account.  Must be less than or
  equal to "amount".

* "for" : a string containing a description of the transfer, with a
  UTF-8 encoding of 200 bytes or less

Response attributes:

* "transfer" : a JSON object containing these values:

  - "transferid" : a string with a random code, with a UTF-8 encoding
    of 32 bytes or less

  - "source" : the access code releasing value

  - "destination" : the access code receiving value

  - "amount" : the total amount of value to move in this transfer

  - "releasedamount" : an integer specifying the amount of value that
    has been released to the destination account
  
  - "for" : a string containing a description of the transfer, with a
    UTF-8 encoding of 200 bytes or less

  - "status" : one of the strings "inprogress", "completed",
    "stoppedbyinitiator", or "timedout"

  - "begintimestamp" : the time the command was processed, represented
    as the number of seconds since the Epoch; may contain decimals

  - "updatetimestamp" : the time of the most recent update to the
    transfer, represented as the number of seconds since the Epoch;
    may contain decimals

* "updateauthcode" : a string with a random code, with a UTF-8
  encoding of 32 bytes or less

Example request:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"begintransfer",
 "source":"123SRC",
 "destination":"789DST",
 "amount":100,
 "releasedamount":0,
 "for":"http://www.acme.com/invoice/14177",
 "requestid":"2a2E",
 "timestamp":1344760952.204,
}\n

Successful response:
{"resultcode":200,
 "explanation":"transfer initiated",
 "requestid":"2a2E",
 "operationid":"iOEq90sKGL",
 "timestamp":1344760978.934,
 "transfer":{
   "transferid":"JSoQqIOBEf",
   "source":"123SRC",
   "destination":"789DST",
   "amount":100,
   "releasedamount":0,
   "status":"inprogress",
   "begintimestamp":1344760978.934,
   "updatetimestamp":1344760978.934
 },
 "updateauthcode":"Q31Lz41SjI"
}\n

Result codes relevant to this command:

* 200 OK
  * The transfer was successfully created
* 420 Insufficient Value
  * The source account does not have enough value to satisfy the initial 'releasedamount'
* 421 Invalid Access Code
* 422 Invalid Destination Access Code


== Update Transfer Command ==

Use the idempotent 'updatetransfer' command to release funds or
permanently cancel a transfer.  Specify either 'releasedamount' or
'status'.

Request attributes:

* "command" : "updatetransfer"

* "transferid" : the string returned in a previous 'begintransfer'
  command
  
* "updateauthcode" : the string returned in the response to
  the previous 'begintransfer' command

* "releasedamount" : an integer specifying the amount of value that
  should be released to the destination account.  New 'releasedamount'
  must be greater or equal to the current 'releasedamount' and less
  than or equal to 'amount'.

* "status" : "stoppedbyinitiator".  Allowed only when 'releasedamount'
  is not present.

Response attributes:

* "transfer" : a JSON object showing the current state of the transfer

Example request to release funds:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"updatetransfer",
 "releasedamount":5,
 "updateauthcode":"Q31Lz41SjI"
 "requestid":"UNFb",
 "timestamp":1345437868.210,
}\n

Successful response:
{"resultcode":200,
 "explanation":"updated transfer",
 "requestid":"UNFb",
 "operationid":"4daZ",
 "timestamp":1345437881.350,
 "transfer":{
   "transferid":"JSoQqIOBEf",
   "source":"123SRC",
   "destination":"789DST",
   "amount":100,
   "releasedamount":5,
   "for":"http://www.acme.com/invoice/14177",
   "status":"inprogress",
   "begintimestamp":1344760978.934,
   "updatetimestamp":1345437881.350
 }
}\n

Example request to stop transfer:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"updatetransfer",
 "status":"stoppedbyinitiator",
 "updateauthcode":"Q31Lz41SjI"
 "requestid":"lnYY",
 "timestamp":1361177033.317,
}\n

Successful response:
{"resultcode":200,
 "explanation":"updated transfer",
 "requestid":"lnYY",
 "operationid":"AP7W",
 "timestamp":1345437881.350,
 "transfer":{
   "transferid":"JSoQqIOBEf",
   "source":"123SRC",
   "destination":"789DST",
   "amount":100,
   "releasedamount":5,
   "for":"http://www.acme.com/invoice/14177",
   "status":"stoppedbyinitiator",
   "begintimestamp":1344760978.934,
   "updatetimestamp":1361177033.317
 }
}\n


Result codes relevant to this command:

* 200 OK
  * The transfer was successfully updated
* 400 Bad Request
  * Both 'releasedamount' and 'status' attributes were specified
  * The specified 'releasedamount' value is greater than the transfer 'amount'
  * The specified 'status' value was not 'stoppedbyinitiator'
* 404 Not Found
  * No transfer found with the specified transfer ID
* 409 Conflict
  * Update refused because transfer status is 'timedout' or 'stoppedbyinitiator'
  * Update refused because it would reduce the 'releasedamount'
* 420 Insufficient Value
  * The source account does not have enough value to satisfy the increase in 'releasedamount'


== Subscribe Transfer Updates ==

Use the nullipotent 'subscribeupdates' command to request that the
server send "102 Update Notification" messages over the current
connection whenever the specified transfer is updated.

Request attributes:

* "transferid" : the string returned in a previous 'begintransfer'
  command

Response attributes:

* "transfer" : a JSON object showing the current state of the transfer

Example request:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"subscribeupdates",
 "transferid":"JSoQqIOBEf",
 "requestid":"e6qU",
 "timestamp":1345438891.510,
}\n

Successful response:
{"resultcode":200,
 "explanation":"subscribed to updates",
 "requestid":"e6qU",
 "operationid":"eiuA",
 "timestamp":1345438905.770,
 "transfer":{
   "transferid":"JSoQqIOBEf",
   "source":"123SRC",
   "destination":"789DST",
   "amount":100,
   "releasedamount":5,
   "for":"http://www.acme.com/invoice/14177",
   "status":"inprogress",
   "begintimestamp":1344760978.934,
   "updatetimestamp":1345437881.350
 }
}\n

Result codes relevant to this command:

* 200 OK
  * Server will send "102 Update Notification" messages when this transfer changes
* 404 Not Found
  * No transfer found with the specified transfer ID

Once this command succeeds, any subsequent updates to the object will
trigger a 102 Update Notification message to be sent over this
connection.  The message will contain the 'requestid' value from the
'subscribeupdates' command.

102 Update Notification message:
{"resultcode":102,
 "explanation":"update notification",
 "requestid":"e6qU",
 "timestamp":1345439315.488,
 "transfer":{
   "transferid":"JSoQqIOBEf",
   "source":"123SRC",
   "destination":"789DST",
   "amount":100,
   "releasedamount":100,
   "for":"http://www.acme.com/invoice/14177",
   "status":"complete",
   "begintimestamp":1344760978.934,
   "updatetimestamp":1345440394.610
  }
}\n


== List Transfers Command ==

Use the nullipotent 'listtransfers' command request to query the
server for transfers with specific parameters.  Since there may be
many matching transfers, the server returns a portion of the matching
transfers and includes a continuation token in the response.  Repeat
the request and include this continuation token to get the next
portion of matching transfers.  The server returns the last portion
with no continuation token.

Request attributes:

* "transferpattern" : a JSON object with transfer parameters.
  Matching transfers will have all of the values specified in the
  pattern.  For example, specifying "source":"123SRC" will ensure that
  only transfers with "source":"123SRC" are returned.
  
  Servers MUST support transfer patterns where both "source" and
  "destination" are specified.  Servers may refuse other patterns with
  425 Unsupported Pattern.

* "continuationtoken" : optional, a string that was returned in the
  response of a previous 'listtransfers' command

Response attributes:

* "transfers" : a list of JSON objects representing the transfers
  matching the specified parameters.  This list may be empty if there
  are no matching transfers.

* "continuationtoken" : optional, a string with a UTF-8 encoding of
  256 bytes or less.  Repeat the request and include this token to
  retrieve the next portion of matching transfers.

Example request:
{"protocol":"hipp/1.0",
 "host":"usd.bank.com",
 "command":"listtransfers",
 "transferpattern":{
   "source":"123SRC",
   "destination":"789DST"
 },
 "requestid":"0912951772",
 "timestamp":1345439837.100,
}\n

Successful response:
{"resultcode":200,
 "explanation":"subscribed to updates",
 "requestid":"0912951772",
 "operationid":"7UO679K7qp",
 "timestamp":1345439873.141,
 "transfers":[
   {"transferid":"JSoQqIOBEf",
    "source":"123SRC",
    "destination":"789DST",
    "amount":100,
    "releasedamount":100,
    "for":"http://www.acme.com/invoice/14177",
    "status":"complete",
    "begintimestamp":1344760978.934,
    "updatetimestamp":1345440394.610
   },
   {"transferid":"r6Gm9Sjg41",
    "source":"123SRC",
    "destination":"789DST",
    "amount":20,
    "releasedamount":0,
    "for":"http://www.acme.com/invoice/14043",
    "status":"inprogress",
    "begintimestamp":1345440524.462,
    "updatetimestamp":1345440524.462
   }
 ],
 "continuationtoken":"SJJZ4od6qV0KNQsLxAakp9QBf5u3gpGWu80AALXfDKj2gK9k5usfkZvavzlL5fzh"
}\n

Result codes relevant to this command:

* 200 OK
* 425 Unsupported Pattern


= Acknowledgements =

Thanks to Brian Chiu for suggesting the term 'segmented transfer'.


= Remaining Questions =

An open-ended segmented transfer may be useful to some applications.
Such a transfer would not specify an amount.


= Related Information =

Bitcoin P2P Digital Currency (2009)
http://bitcoin.org/

U.S. Financial Crimes Enforcement Network
http://www.fincen.gov/

Internet Secure Payments Protocol (ispp) Working Group (1997)
http://www.ietf.org/wg/concluded/ispp.html

Internet Keyed Payment Protocols (iKP)
http://www.zurich.ibm.com/security/past-projects/ecommerce/iKP_overview.html

Secure Electronic Transaction (SET) (1997)

Visa 3-D Secure (Verified by Visa)

Internet Open Trading Protocol (trade) Working Group (1998-2005)
http://datatracker.ietf.org/wg/trade/charter/

QuickPay Online Payment Protocol
(Central trusted host)
