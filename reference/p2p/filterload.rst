.. Copyright (c) 2014-2018 Bitcoin.org
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

filterload
----------

The filterload message tells the receiving peer to filter all relayed transactions and requested merkle blocks through the provided filter. Part of the connection bloom filtering mechanism as described in `BIP-37 <https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki>`__.

This allows clients to receive transactions relevant to their wallet plus a configurable rate of false positive transactions which can provide plausible-deniability privacy.

+------------+------------------+----------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Name       | Data Type        | Bytes    | Description                                                                                                                                                                |
+============+==================+==========+============================================================================================================================================================================+
| filter     | vector_\<uint8_> | *Varies* | A bit field of arbitrary byte-aligned size. The maximum size is 36,000 bytes.                                                                                              |
+------------+------------------+----------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| nHashFuncs | uint32_          | 4        | The number of hash functions to use in this filter. The maximum value allowed in this field is 50.                                                                         |
+------------+------------------+----------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| nTweak     | uint32_          | 4        | An arbitrary value to add to the seed value in the hash function used by the bloom filter.                                                                                 |
+------------+------------------+----------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| nFlags     | uint8_           | 1        | A set of flags that control how outpoints corresponding to a matched pubkey script are added to the filter. See the table in the Updating A Bloom Filter subsection below. |
+------------+------------------+----------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

The annotated hexdump below shows a "filterload" message. (The message header has been omitted.) For an example of how this payload was created, see the filterload example.

.. highlight:: text

::

   02 ......... Filter bytes: 2
   b50f ....... Filter: 1010 1101 1111 0000
   0b000000 ... nHashFuncs: 11
   00000000 ... nTweak: 0/none
   00 ......... nFlags: BLOOM_UPDATE_NONE

**Initializing A Bloom Filter**

Filters have two core parameters: the size of the bit field and the number of hash functions to run against each data element. The following formulas from `BIP-37 <https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki>`__ will allow you to automatically select appropriate values based on the number of elements you plan to insert into the filter (*n*) and the false positive rate (*p*) you desire to maintain plausible deniability.

-  Size of the bit field in bytes (*nFilterBytes*), up to a maximum of 36,000: ``(-1 / log(2)**2 * n * log(p)) / 8``

-  Hash functions to use (*nHashFuncs*), up to a maximum of 50: ``nFilterBytes * 8 / n * log(2)``

Note that the filter matches parts of transactions (transaction elements), so the false positive rate is relative to the number of elements checked—not the number of transactions checked. Each normal transaction has a minimum of four matchable elements (described in the comparison subsection below), so a filter with a false-positive rate of 1 percent will match about 4 percent of all transactions at a minimum.

According to `BIP-37 <https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki>`__, the formulas and limits described above provide support for bloom filters containing 20,000 items with a false positive rate of less than 0.1 percent or 10,000 items with a false positive rate of less than 0.0001 percent.

Once the size of the bit field is known, the bit field should be initialized as all zeroes.

**Populating A Bloom Filter**

The bloom filter is populated using between 1 and 50 unique hash functions (the number specified per filter by the *nHashFuncs* field). Instead of using up to 50 different hash function implementations, a single implementation is used with a unique seed value for each function.

The seed is ``nHashNum * 0xfba4c795 + nTweak`` as a *uint32_t*, where the values are:

-  **nHashNum** is the sequence number for this hash function, starting at 0 for the first hash iteration and increasing up to the value of the *nHashFuncs* field (minus one) for the last hash iteration.

-  **0xfba4c795** is a constant optimized to create large differences in the seed for different values of *nHashNum*.

-  **nTweak** is a per-filter constant set by the client to require the use of an arbitrary set of hash functions.

If the seed resulting from the formula above is larger than four bytes, it must be truncated to its four most significant bytes (for example, ``0x8967452301 & 0xffffffff → 0x67452301``).

The actual hash function implementation used is the `32-bit Murmur3 hash function <https://en.wikipedia.org/wiki/MurmurHash>`__.

**Warning:** the Murmur3 hash function has separate 32-bit and 64-bit versions that produce different results for the same input. Only the 32-bit Murmur3 version is used with Unit-e bloom filters.

The data to be hashed can be any transaction element which the bloom filter can match. See the next subsection for the list of transaction elements checked against the filter. The largest element which can be matched is a script data push of 520 bytes, so the data should never exceed 520 bytes.

The example below from unit-e `bloom.cpp <https://github.com/dtr-org/unit-e/blob/74834896848d6fd0fd0b377aed29fae451453097/src/bloom.cpp#L55>`__ combines all the steps above to create the hash function template. The seed is the first parameter; the data to be hashed is the second parameter. The result is a uint32_t modulo the size of the bit field in bits.

.. highlight:: c++

::

   MurmurHash3(nHashNum * 0xFBA4C795 + nTweak, vDataToHash) % (vData.size() * 8)

Each data element to be added to the filter is hashed by *nHashFuncs* number of hash functions. Each time a hash function is run, the result will be the index number (*nIndex*) of a bit in the bit field. That bit must be set to 1. For example if the filter bit field was ``00000000`` and the result is 5, the revised filter bit field is ``00000100`` (the first bit is bit 0).

It is expected that sometimes the same index number will be returned more than once when populating the bit field; this does not affect the algorithm—after a bit is set to 1, it is never changed back to 0.

After all data elements have been added to the filter, each set of eight bits is converted into a little-endian byte. These bytes are the value of the *filter* field.

**Comparing Transaction Elements To A Bloom Filter**

To compare an arbitrary data element against the bloom filter, it is hashed using the same parameters used to create the bloom filter. Specifically, it is hashed *nHashFuncs* times, each time using the same *nTweak* provided in the filter, and the resulting output is modulo the size of the bit field provided in the *filter* field. After each hash is performed, the filter is checked to see if the bit at that indexed location is set. For example if the result of a hash is ``5`` and the filter is ``01001110``, the bit is considered set.

If the result of every hash points to a set bit, the filter matches. If any of the results points to an unset bit, the filter does not match.

The following transaction elements are compared against bloom filters. All elements will be hashed in the byte order used in blocks (for example, TXIDs will be in internal byte order).

-  **TXIDs:** the transaction’s SHA256(SHA256()) hash.

-  **Outpoints:** each 36-byte outpoint used this transaction’s input section is individually compared to the filter.

-  **Signature Script Data:** each element pushed onto the stack by a data-pushing opcode in a signature script from this transaction is individually compared to the filter. This includes data elements present in P2SH redeem scripts when they are being spent.

-  **PubKey Script Data:** each element pushed onto the the stack by a data-pushing opcode in any pubkey script from this transaction is individually compared to the filter. (If a pubkey script element matches the filter, the filter will be immediately updated if the ``BLOOM_UPDATE_ALL`` flag was set; if the pubkey script is in the P2PKH format and matches the filter, the filter will be immediately updated if the ``BLOOM_UPDATE_P2PUBKEY_ONLY`` flag was set. See the subsection below for details.)

The following annotated hexdump of a transaction is from the `raw transaction format section <types/Transaction.html>`__; the elements which would be checked are the Outpoint TXID, the Outpoint index number, the Secp256k1 signature, and the PubKey hash. Note that this transaction’s TXID (**``01000000017b1eab[...]``**) would also be checked, and that the outpoint TXID and index number below would be checked as a single 36-byte element.

.. highlight:: text

::

   01000000 ................................... Version

   01 ......................................... Number of inputs
   |
   | 7b1eabe0209b1fe794124575ef807057
   | c77ada2138ae4fa8d6c4de0398a14f3f ......... Outpoint TXID
   | 00000000 ................................. Outpoint index number
   |
   | 49 ....................................... Bytes in sig. script: 73
   | | 48 ..................................... Push 72 bytes as data
   | | | 30450221008949f0cb400094ad2b5eb3
   | | | 99d59d01c14d73d8fe6e96df1a7150de
   | | | b388ab8935022079656090d7f6bac4c9
   | | | a94e0aad311a4268e082a725f8aeae05
   | | | 73fb12ff866a5f01 ..................... Secp256k1 signature
   |
   | ffffffff ................................. Sequence number: UINT32_MAX

   01 ......................................... Number of outputs
   | f0ca052a01000000 ......................... Satoshis (49.99990000 UTE)
   |
   | 19 ....................................... Bytes in pubkey script: 25
   | | 76 ..................................... OP_DUP
   | | a9 ..................................... OP_HASH160
   | | 14 ..................................... Push 20 bytes as data
   | | | cbc20a7664f2f69e5355aa427045bc15
   | | | e7c6c772 ............................. PubKey hash
   | | 88 ..................................... OP_EQUALVERIFY
   | | ac ..................................... OP_CHECKSIG

   00000000 ................................... locktime: 0 (a block height)

**Updating A Bloom Filter**

Clients will often want to track inputs that spend outputs (outpoints) relevant to their wallet, so the filterload field *nFlags* can be set to allow the filtering node to update the filter when a match is found. When the filtering node sees a pubkey script that pays a pubkey, address, or other data element matching the filter, the filtering node immediately updates the filter with the outpoint corresponding to that pubkey script.

.. figure:: /img/dev/en-bloom-update.svg
   :alt: Automatically Updating Bloom Filters

   Automatically Updating Bloom Filters

If an input later spends that outpoint, the filter will match it, allowing the filtering node to tell the client that one of its transaction outputs has been spent.

The *nFlags* field has three allowed values:

+-------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Value | Name                       | Description                                                                                                                                                                        |
+=======+============================+====================================================================================================================================================================================+
| 0     | BLOOM_UPDATE_NONE          | The filtering node should not update the filter.                                                                                                                                   |
+-------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | BLOOM_UPDATE_ALL           | If the filter matches any data element in a pubkey script, the corresponding outpoint is added to the filter.                                                                      |
+-------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2     | BLOOM_UPDATE_P2PUBKEY_ONLY | If the filter matches any data element in a pubkey script and that script is either a P2PKH or non-P2SH pay-to-multisig script, the corresponding outpoint is added to the filter. |
+-------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

In addition, because the filter size stays the same even though additional elements are being added to it, the false positive rate increases. Each false positive can result in another element being added to the filter, creating a feedback loop that can (after a certain point) make the filter useless. For this reason, clients using automatic filter updates need to monitor the actual false positive rate and send a new filter when the rate gets too high.

.. _uint32: types/Integers.html
.. _uint8: types/Integers.html
.. _vector: types/vector.html

.. Content originally imported from https://github.com/bitcoin-dot-org/bitcoin.org/blob/master/_data/devdocs/en/references/
