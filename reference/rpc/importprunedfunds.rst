.. Copyright (c) 2018-2019 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

importprunedfunds
-----------------

``importprunedfunds``

Imports funds without rescan. Corresponding address or script must previously be included in wallet. Aimed towards pruned wallets. The end-user is responsible to import additional transactions that subsequently spend the imported outputs or rescan after the point in the blockchain the transaction is included.

Argument #1 - rawtransaction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string, required

A raw transaction in hex funding an already-existing address in wallet

Argument #2 - txoutproof
~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string, required

The hex output from gettxoutproof that contains the transaction

