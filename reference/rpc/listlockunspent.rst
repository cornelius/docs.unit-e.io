.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

listlockunspent
---------------

``listlockunspent``

Returns list of temporarily unspendable outputs.

See the lockunspent call to lock and unlock transactions for spending.

Result
~~~~~~

::

  [
    {
      "txid" : "transactionid",     (string) The transaction id locked
      "vout" : n                      (numeric) The vout value
    }
    ,...
  ]

Examples
~~~~~~~~


.. highlight:: shell

List the unspent transactions::

  unit-e-cli listunspent

Lock an unspent transaction::

  unit-e-cli lockunspent false "[{\"txid\":\"a08e6907dbbd3d809776dbfc5d82e371b764ed838b5655e72f463568df1aadf0\",\"vout\":1}]"

List the locked transactions::

  unit-e-cli listlockunspent

Unlock the transaction again::

  unit-e-cli lockunspent true "[{\"txid\":\"a08e6907dbbd3d809776dbfc5d82e371b764ed838b5655e72f463568df1aadf0\",\"vout\":1}]"

As a json rpc call::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "listlockunspent", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

