.. Copyright (c) 2018 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

bumpfee
-------

``bumpfee "txid" ( options )``

Bumps the fee of an opt-in-RBF transaction T, replacing it with a new transaction B.

An opt-in RBF transaction with the given txid must be in the wallet.

The command will pay the additional fee by decreasing (or perhaps removing) its change output.

If the change output is not big enough to cover the increased fee, the command will currently fail
instead of adding new inputs to compensate. (A future implementation could improve this.)
The command will fail if the wallet or mempool contains a transaction that spends one of T's outputs.

By default, the new fee will be calculated automatically using estimatefee.

The user can specify a confirmation target for estimatefee.

Alternatively, the user can specify totalFee, or use RPC settxfee to set a higher fee rate.

At a minimum, the new fee rate must be high enough to pay an additional new relay fee (incrementalfee
returned by getnetworkinfo) to enter the node's mempool.

Argument #1 - txid
~~~~~~~~~~~~~~~~~~

**Type:** string, required

The txid to be bumped

Argument #2 - options
~~~~~~~~~~~~~~~~~~~~~

**Type:** object, optional

::

   {
     "confTarget"        (numeric, optional) Confirmation target (in blocks)
     "totalFee"          (numeric, optional) Total fee (NOT feerate) to pay, in satoshis.
                         In rare cases, the actual fee paid might be slightly higher than the specified
                         totalFee if the tx change output has to be removed because it is too close to
                         the dust threshold.
     "replaceable"       (boolean, optional, default true) Whether the new transaction should still be
                         marked bip-125 replaceable. If true, the sequence numbers in the transaction will
                         be left unchanged from the original. If false, any input sequence numbers in the
                         original transaction that were less than 0xfffffffe will be increased to 0xfffffffe
                         so the new transaction will not be explicitly bip-125 replaceable (though it may
                         still be replaceable in practice, for example if it has unconfirmed ancestors which
                         are replaceable).
     "estimate_mode"     (string, optional, default=UNSET) The fee estimate mode, must be one of:
         "UNSET"
         "ECONOMICAL"
         "CONSERVATIVE"
   }

Argument #3 - test_fee
~~~~~~~~~~~~~~~~~~~~~~

**Type:** bool, optional, default=false

Only return the fee it would cost to send, txn is discarded.

Result
~~~~~~

::

  {
    "txid":    "value",   (string)  The id of the new transaction
    "origfee":  n,         (numeric) Fee of the replaced transaction
    "fee":      n,         (numeric) Fee of the new transaction
    "errors":  [ str... ] (json array of strings) Errors encountered during processing (may be empty)
  }

Examples
~~~~~~~~

Bump the fee, get the new transaction's txid::

  unite-cli bumpfee <txid>

