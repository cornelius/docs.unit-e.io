.. Copyright (c) 2018 The Unit-e developers
   Distributed under the MIT software license, see the accompanying
   file LICENSE or https://opensource.org/licenses/MIT.

encryptwallet
-------------

``encryptwallet "passphrase"``

Encrypts the wallet with 'passphrase'. This is for first time encryption.

After this, any calls that interact with private keys such as sending or signing
will require the passphrase to be set prior the making these calls.

Use the walletpassphrase call for this, and then walletlock call.

If the wallet is already encrypted, use the walletpassphrasechange call.

Note that this will shutdown the server.

Argument #1 - passphrase
~~~~~~~~~~~~~~~~~~~~~~~~

**Type:** string

The pass phrase to encrypt the wallet with. It must be at least 1 character, but should be long.

Examples
~~~~~~~~

Encrypt your wallet::

  unite-cli encryptwallet "my pass phrase"

Now set the passphrase to use the wallet, such as for signing or sending unite::

  unite-cli walletpassphrase "my pass phrase"

Now we can do something like sign::

  unite-cli signmessage "address" "test message"

Now lock the wallet again by removing the passphrase::

  unite-cli walletlock

As a json rpc call::

  curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "encryptwallet", "params": ["my pass phrase"] }' -H 'content-type: text/plain;' http://127.0.0.1:7181/

