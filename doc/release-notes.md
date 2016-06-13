(note:這是一個暫時性的文件，可任意更改，然後會在發布時被移到release-notes)

Bitcoin Core version *version* is now available from:

  <https://bitcoin.org/bin/bitcoin-core-*version*/>

這是一個先釋出的版本，包含新的features,各種bug修改，performance improvements和更新的翻譯

如果有發現bug請用github的issue tracker回報

  <https://github.com/bitcoin/bitcoin/issues>

要收到安全性或更新通知,請訂閱以下

  <https://bitcoincore.org/en/list/announcements/join/>

值得注意的修改
===============

Example item
----------------


bitcoin-cli: arguments隱私
--------------------------------

RPC command line client新增了一項argument, `-stdin`
來從標準的輸入中讀入額外的arguments,一行一個直到 EOF/Ctrl-D.
For example:

    $ echo -e "mysecretcode\n120" | src/bitcoin-cli -stdin walletpassphrase

這個推薦可以給易變的資料來做使用,像wallet passphrases,同時command-line arguments可以頻繁的自process table中供所有系統的使用者讀取

RPC 底層更動
----------------------

- `gettxoutsetinfo` UTXO hash (`hash_serialized`) 做了更動. 原本32-bit和64-bit平台間有歧異,且hashed data中的txids遺失了.這已經被修正了,但這意味著輸出會和前一版本不同


C++11 and Python 3
-------------------

一些code modernizations(現代化)被完成,比特幣核心code base已經開始使用C++11.這意味著現在要build需要一個C++11相容的compiler,簡單來說就是GCC 4.7 or higher,或Clang 3.3 or higher.

若要cross-compiling for一個沒有C++11的目標,請裝配
`./configure --enable-glibc-back-compat ... LDFLAGS=-static-libstdc++`.

如果要跑 `qa/rpc-tests`中的函數測試,現在需要 Python3.4 or higher

0.13.0 Change log
=================

以下是較Detailed的release notes.此概述包含影響運轉狀態的更動,而非code的移動,refactors及字串更動
為了locating的方便,code的更及伴隨的討論,包含pull request和git merge commit都有提到

### RPC and REST

Asm script的輸出現在包含 OP_CHECKLOCKTIMEVERIFY 來代替 OP_NOP2
-------------------------------------------------------------------------

OP_NOP2 has been renamed to OP_CHECKLOCKTIMEVERIFY by [BIP 
65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki)

以下輸出因上述更動而有所改變:
- RPC `getrawtransaction` (in verbose mode)
- RPC `decoderawtransaction`
- RPC `decodescript`
- REST `/rest/tx/` (JSON format)
- REST `/rest/block/` (JSON format when including extended tx details)
- `bitcoin-tx -json`

New mempool information RPC calls
---------------------------------

RPC calls的詳細統計資料被增加到輸出中個別的mempool entries中,為了計算交易的in-mempool祖先或子孫
see `getmempoolentry`, `getmempoolancestors`, `getmempooldescendants`.

### ZMQ

每個ZMQ 通知現在都包含一個up-counting的序列數字讓listeners可以偵測到遺失的通知
序列數字永遠都是一個多部份的ZMQ通知中的最後一個元素,因此也支援向後相容
每個massage type都有自己的counter
(https://github.com/bitcoin/bitcoin/pull/7762)

### Configuration and command-line options

### Block and transaction handling

### P2P protocol and network code

The p2p alert system has been removed in #7692 and the 'alert' message is no longer supported.


Fee filtering of invs (BIP 133)
------------------------------------

The optional new p2p message "feefilter" is implemented and the protocol
version is bumped to 70013. Upon receiving a feefilter message from a peer,
a node will not send invs for any transactions which do not meet the filter
feerate. [BIP 133](https://github.com/bitcoin/bips/blob/master/bip-0133.mediawiki)

### Validation

### Build system

### Wallet

### GUI

### Tests

### Miscellaneous

