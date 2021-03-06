.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

gettxoutsetinfo
---------------

``gettxoutsetinfo``

Returns statistics about the unspent transaction output set.

Note this call may take some time.

Result
~~~~~~

::

  {
    "height":n,     (numeric) The current block height (index)
    "bestblock": "hex",   (string) The hash of the block at the tip of the chain
    "transactions": n,      (numeric) The number of transactions with unspent outputs
    "txouts": n,            (numeric) The number of unspent transaction outputs
    "bogosize": n,          (numeric) A meaningless metric for UTXO set size
    "hash_serialized_2": "hash", (string) The serialized hash
    "disk_size": n,         (numeric) The estimated size of the chainstate on disk
    "total_amount": x.xxx          (numeric) The total amount
  }

Examples
~~~~~~~~


.. highlight:: shell

::

  unit-e-cli gettxoutsetinfo

::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "gettxoutsetinfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

