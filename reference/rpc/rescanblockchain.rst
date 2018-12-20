.. Copyright (c) 2018 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

rescanblockchain
----------------

``rescanblockchain ("start_height") ("stop_height")``

Rescan the local blockchain for wallet related transactions.

Argument #1 - start_height
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** numeric, optional

block height where the rescan should start

Argument #2 - stop_height
~~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** numeric, optional

the last block height that should be scanned

Result
~~~~~~

::

  {
    "start_height"     (numeric) The block height where the rescan has started. If omitted, rescan started from the genesis block.
    "stop_height"      (numeric) The height of the last rescanned block. If omitted, rescan stopped at the chain tip.
  }

Examples
~~~~~~~~

::

  unite-cli rescanblockchain 100000 120000

::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "rescanblockchain", "params": [100000, 120000] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

