#### Notaries Rpc Tools

This repo provides a few useful RPC methods for Komodo notary nodes, which was not yet implemented in daemon.
For example, `utxo spliting`. Before, it was implemented in iguana or various scripts, but splitting directly
from daemon was not possible. Not is time to fix it.

First of all, start your daemon as notary node with corresponding `-pubkey=%your_compressed_public_key%'. 
Then, start daemon and make a call to `nn_getwalletinfo`. It haven't any params, so, it could be called simply
like: `./src/komodo-cli -ac_name=DECKER nn_getwalletinfo`.

The result will be following:
```
{
  "currentSeason": 7,
  "nn_index": 33,
  "nn_name": "decker_EU",
  "pubkey": "036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39",
  "pubkey_address": "1E4gmPV7D23pYjxKAYpbw4MhfRBaYp1HtD",
  "ismine": true,
  "transactions_count": 104,
  "available_coins_count": 5,
  "notaryvins_utxos_count": 0,
  "others_utxos_count": 5
}
```

It means, that your wallet contains 104 txes in total (including those that not in the mainchain), totally you have 5 (`available_coins_count`) utxos and among them you have 0 notaryvins and 5 other utxos. Let's create 10 notaryvins, it can be done simple with:

```
./src/komodo-cli -ac_name=DECKER nn_split
```
Or the full form of this command:
```
./src/komodo-cli -ac_name=DECKER nn_split 10 0.0001 false true true
```
It will produce following result:
```
{
  "tx": "bd5003fc5f3223d80d83c308fefe09576db58f98bf0172ca1d957931634d7c25",
  "input_utxos_value": 1.00000000,
  "input_utxos_count": 1,
  "out_notaryvins_count": 10,
  "out_utxos_value": 0.99890000,
  "out_utxos_count": 1,
  "estimated_tx_size": 348,
  "real_tx_size": 598
}
```
It means, that 1 utxo (`input_utxos_count`) was used as `vin` and 10 notaryvins was created, also 1 utxo was used as a change in `vout` (`out_utxos_count`). In other words, we took 1 utxo containing 1 COIN, made `10 * 0.0001 COIN` notary vins utxos from it, and used 0.0001 COIN as a transaction fee. The result tx will look like a:
```
0100000001636797556ecb993e1412a48af3ad578a23f510259792ceb4109d688704404c6f0000000049483045022100d20aaaac8058ec8756f4d7ea95a82d9fe008b8d3762f74c2b4b31d1ac842199d0220124623fed733b207248ea1f01b88f8a761dcc52dcda3134451e0a9c968c6966801ffffffff0b5033f405000000001976a9148f4c13d5ab46b533a3e208a419799990f8c693d888ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac10270000000000002321036b65cf6ba7160ec22012dc11f28280b3c79cfdd5f85f172140069eac0bb42b39ac00000000
```
Use `decoderawtransaction` to see it's fields, etc. Now let's time to learn the params of `nn_split` RPC. Let's read help:
```
./src/komodo-cli -ac_name=DECKER help nn_split
```

As you are see you can merge all your utxos in the wallet with splitting at the same time (in one transaction), buy using `fmergeallutxos=true` and `fskipnotaryvins=false`. It will merge all existing utxos including notaryvins and will create `countnotaryvintocreate` notaryvins in the wallet. This form of command is very useful in combine with `cleanwallettransactions`, when you'll need to clean the wallet from the txes and have some splitted utxos at once.

Arguments of `nn_split` are following:
```
1. countnotaryvintocreate (numeric, default=10) - number of notary vin P2PK utxos to be created.
2. fee (numeric, default=0.0001) - the fee amount to attach to this transaction.
3. fmergeallutxos (boolean, default=false) - if true will merge all utxos, else merge only needed vins to match needed amount of notaryvins output.
4. fskipnotaryvins (boolean, default=true) - if true - will skip (don't merge) existing notaryvins, else will merge existing notaryvins.
5. fsendtransaction (boolean, default=true) - if true - will broadcast tx immediatelly, false - just show raw hex tx.
```

Instructions of how to instert these sources in your komodo daemon you can find in the end of `notaries.cpp` file.

#### Dev notes

Utxo splitting (create of resulting transaction) implemented in two different ways:
- using `CMutableTransaction` (method, compatible almost with all utxo coins)
- using `TransactionBuilder` (method, suitable for ZCash / ZEC based coins)

You can choose which method is actually used by `fUseTxBuilder` in the sources, or you can comment `TransactionBuilder` method at all, if your coin doesn't support it.

