.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

getmempoolinfo
--------------

``getmempoolinfo``

Returns details on the active state of the TX memory pool.

Result
~~~~~~

::

  {
    "size": xxxxx,               (numeric) Current tx count
    "bytes": xxxxx,              (numeric) Sum of all virtual transaction sizes as defined in BIP 141. Differs from actual serialized size because witness data is discounted
    "usage": xxxxx,              (numeric) Total memory usage for the mempool
    "maxmempool": xxxxx,         (numeric) Maximum memory usage for the mempool
    "mempoolminfee": xxxxx       (numeric) Minimum fee rate in UTE/kB for tx to be accepted. Is the maximum of minrelaytxfee and minimum mempool fee
    "minrelaytxfee": xxxxx       (numeric) Current minimum relay fee for transactions
  }

Examples
~~~~~~~~


.. highlight:: shell

::

  unit-e-cli getmempoolinfo

::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmempoolinfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

