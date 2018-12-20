.. Copyright (c) 2018 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

decodescript
------------

``decodescript "hexstring"``

Decode a hex-encoded script.

Argument #1 - hexstring
~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string

the hex encoded script

Result
~~~~~~

::

  {
    "asm":"asm",   (string) Script public key
    "hex":"hex",   (string) hex encoded public key
    "type":"type", (string) The output type
    "reqSigs": n,    (numeric) The required signatures
    "addresses": [   (json array of string)
       "address"     (string) unite address
       ,...
    ],
    "p2sh","address" (string) address of P2SH script wrapping this redeem script (not returned if the script is already a P2SH).
  }

Examples
~~~~~~~~

::

  unite-cli decodescript "hexstring"

::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "decodescript", "params": ["hexstring"] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

