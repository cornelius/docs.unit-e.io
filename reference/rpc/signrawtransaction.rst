.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

signrawtransaction
------------------

``signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )``

Sign inputs for raw transaction (serialized, hex-encoded).

The second optional argument (may be null) is an array of previous transaction outputs that
this transaction depends on but may not yet be in the block chain.

The third optional argument (may be null) is an array of base58-encoded private
keys that, if given, will be the only keys used to sign the transaction.

Argument #1 - hexstring
~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string, required

The transaction hex string

Argument #2 - prevtxs
~~~~~~~~~~~~~~~~~~~~~

**Type:** string, optional

An json array of previous dependent transaction outputs

::

     [               (json array of json objects, or 'null' if none provided)
       {
         "txid":"id",             (string, required) The transaction id
         "vout":n,                  (numeric, required) The output number
         "scriptPubKey": "hex",   (string, required) script key
         "redeemScript": "hex",   (string, required for P2SH or P2WSH) redeem script
         "amount": value            (numeric, required) The amount spent
       }
       ,...
    ]

Argument #3 - privkeys
~~~~~~~~~~~~~~~~~~~~~~

**Type:** string, optional

A json array of base58-encoded private keys for signing

::

    [                  (json array of strings, or 'null' if none provided)
      "privatekey"   (string) private key in base58-encoding
      ,...
    ]

Argument #4 - sighashtype
~~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string, optional, default=ALL

The signature hash type. Must be one of
       "ALL"
       "NONE"
       "SINGLE"
       "ALL|ANYONECANPAY"
       "NONE|ANYONECANPAY"
       "SINGLE|ANYONECANPAY"

Result
~~~~~~

::

  {
    "hex" : "value",           (string) The hex-encoded raw transaction with signature(s)
    "complete" : true|false,   (boolean) If the transaction has a complete set of signatures
    "errors" : [                 (json array of objects) Script verification errors (if there are any)
      {
        "txid" : "hash",           (string) The hash of the referenced, previous transaction
        "vout" : n,                (numeric) The index of the output to spent and used as input
        "scriptSig" : "hex",       (string) The hex-encoded signature script
        "sequence" : n,            (numeric) Script sequence number
        "error" : "text"           (string) Verification or signing error related to the input
      }
      ,...
    ]
  }

Examples
~~~~~~~~


.. highlight:: shell

::

  unit-e-cli signrawtransaction "myhex"

::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "signrawtransaction", "params": ["myhex"] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

