Unauthenticated REST Interface
==============================

The REST API can be enabled with the `-rest` option.
REST(具象狀態傳輸)是英文 REpresentational State Transfer 的縮寫，是近年來迅速興起的，一種基於 HTTP，URI，以及 XML 這些現有協議與標準的，針對網路應用的設計和開發方式。它可以降低開發的複雜度，提高系統的可伸縮性。 
REST 的核心是可編輯的資源及其集合，每個資源或者集合有一個惟一的 URI。系統以資源為中心，構建並提供一系列的 Web 服務。
REST 的基本概念和原則包括：系統上的所有事物都被抽象為資源；每個資源對應唯一的資源標識；對資源的操作不會改變資源標識本身；所有的操作都是無狀態的；

使用方式:
在 REST 中，開發人員顯式地使用 HTTP 方法，對系統資源進行建立、讀取、更新和刪除的操作： 
使用 POST 方法在伺服器上建立資源 
使用 GET 方法從伺服器檢索某個資源或者資源集合 
使用 PUT 方法對伺服器的現有資源進行更新 
使用 DELETE 方法刪除伺服器的某個資源

Supported API 支援交易、區塊、區塊頭、鏈資訊的API
-------------

####Transactions
`GET /rest/tx/<TX-HASH>.<bin|hex|json>`
傳入transaction hash回傳的transaction可以以binary, hex-encoded binary, 或是 JSON formats形式回傳
Given a transaction hash: returns a transaction in binary, hex-encoded binary, or JSON formats.

For full TX query capability, one must enable the transaction index via "txindex=1" command line / configuration option.

####Blocks
`GET /rest/block/<BLOCK-HASH>.<bin|hex|json>`
`GET /rest/block/notxdetails/<BLOCK-HASH>.<bin|hex|json>`

傳入block hash回傳block可以以binary, hex-encoded binary, 或是 JSON formats形式回傳
Given a block hash: returns a block, in binary, hex-encoded binary or JSON formats.

若使用HTTP方式的request and response則會handled entirely in-memory
The HTTP request and response are both handled entirely in-memory, thus making maximum memory usage at least 2.66MB (1 MB max block, plus hex encoding) per request.

如果使用/notxdetails/這種JSON的方式，回傳的值只會包含交易的hash值而不是整個交易訊息
With the /notxdetails/ option JSON response will only contain the transaction hash instead of the complete transaction details. The option only affects the JSON response.

####Blockheaders
`GET /rest/headers/<COUNT>/<BLOCK-HASH>.<bin|hex|json>`
傳入block hash會以<COUNT>回傳上層的blockheaders，以binary, hex-encoded binary, 或是 JSON formats形式回傳
Given a block hash: returns <COUNT> amount of blockheaders in upward direction.

####Chaininfos
`GET /rest/chaininfo.json`
只支援JOSN的形式，可以回傳多種block chain processing狀態
Returns various state info regarding block chain processing.
Only supports JSON as output format.
* chain : (string) current network name as defined in BIP70 (main, test, regtest)
* blocks : (numeric) the current number of blocks processed in the server
* headers : (numeric) the current number of headers we have validated
* bestblockhash : (string) the hash of the currently best block
* difficulty : (numeric) the current difficulty
* verificationprogress : (numeric) estimate of verification progress [0..1]
* chainwork : (string) total amount of work in active chain, in hexadecimal
* pruned : (boolean) if the blocks are subject to pruning
* pruneheight : (numeric) heighest block available
* softforks : (array) status of softforks in progress

####Query UTXO set
`GET /rest/getutxos/<checkmempool>/<txid>-<n>/<txid>-<n>/.../<txid>-<n>.<bin|hex|json>`

The getutxo command allows querying of the UTXO set given a set of outpoints.
See BIP64 for input and output serialisation:
https://github.com/bitcoin/bips/blob/master/bip-0064.mediawiki

Example:
```
$ curl localhost:18332/rest/getutxos/checkmempool/b2cdfd7b89def827ff8af7cd9bff7627ff72e5e8b0f71210f92ea7a4000c5d75-0.json 2>/dev/null | json_pp
{
   "chaintipHash" : "00000000fb01a7f3745a717f8caebee056c484e6e0bfe4a9591c235bb70506fb",
   "chainHeight" : 325347,
   "utxos" : [
      {
         "scriptPubKey" : {
            "addresses" : [
               "mi7as51dvLJsizWnTMurtRmrP8hG2m1XvD"
            ],
            "type" : "pubkeyhash",
            "hex" : "76a9141c7cebb529b86a04c683dfa87be49de35bcf589e88ac",
            "reqSigs" : 1,
            "asm" : "OP_DUP OP_HASH160 1c7cebb529b86a04c683dfa87be49de35bcf589e OP_EQUALVERIFY OP_CHECKSIG"
         },
         "value" : 8.8687,
         "height" : 2147483647,
         "txvers" : 1
      }
   ],
   "bitmap" : "1"
}
```

####Memory pool
`GET /rest/mempool/info.json`

回傳TX mempool的資訊，包括size、bytes、usage
tx(tx describes a bitcoin transaction)
連結:https://en.bitcoin.it/wiki/Protocol_documentation#tx

Returns various information about the TX mempool.
Only supports JSON as output format.
* size : (numeric) the number of transactions in the TX mempool
* bytes : (numeric) size of the TX mempool in bytes
* usage : (numeric) total TX mempool memory usage

`GET /rest/mempool/contents.json`
回傳TX mempool的交易
Returns transactions in the TX mempool.
Only supports JSON as output format.

Risks
-------------
Running a web browser on the same node with a REST enabled bitcoind can be a risk. Accessing prepared XSS websites could read out tx/block data of your node by placing links like `<script src="http://127.0.0.1:8332/rest/tx/1234567890.json">` which might break the nodes privacy.
