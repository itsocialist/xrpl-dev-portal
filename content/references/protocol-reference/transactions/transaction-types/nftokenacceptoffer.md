---
html: nftokenacceptoffer.html
parent: transaction-types.html
blurb: Accept an offer to buy or sell an NFToken.
labels:
  - NFTs, Non-fungible Tokens
status: not_enabled
---
# NFTokenAcceptOffer
{% include '_snippets/nfts-disclaimer.md' %}

The `NFTokenAcceptOffer` transaction is used to accept offers to `buy` or `sell` an `NFToken`. It can either:

* Allow one offer to be accepted. This is called _direct_ mode.
* Allow two distinct offers, one offering to buy a given `NFToken` and the other offering to sell the same `NFToken`, to be accepted in an atomic fashion. This is called _brokered_ mode.


## Brokered vs. Direct Mode

The mode in which the transaction operates depends on the presence of the `SellOffer` and `BuyOffer` fields of the transaction:

| `SellOffer` | `BuyOffer` | Mode     |
|:------------|:-----------|:---------|
| ✔️           | ✔️          | Brokered |
| ✔️           | X          | Direct   |
| X           | ✔️          | Direct   |


If neither of those fields is specified, the transaction is malformed and produces a `tem` class error.

The semantics of brokered mode are slightly different than one in direct mode: The account executing the transaction functions as a broker, bringing the two offers together and causing them to be matched, but does not acquire ownership of the involved NFT, which will, if the transaction is successful, be transferred directly from the seller to the buyer.


## Execution Details


### Direct Mode

In direct mode, `NFTokenAcceptOffer` transaction fails if:

* The `NFTokenOffer` against which `NFTokenAcceptOffer` transaction is placed is an offer to `buy` the `NFToken` and the account executing the `NFTokenAcceptOffer` is not, at the time of execution, the current owner of the corresponding `NFToken`.
* The `NFTokenOffer` against which `NFTokenAcceptOffer` transaction is placed is an offer to `sell` the `NFToken` and was placed by an account which is not, at the time of execution, the current owner of the `NFToken`
* The `NFTokenOffer` against which `NFTokenAcceptOffer` transaction is placed is an offer to `sell` the `NFToken` and was placed by an account which is not, at the time of execution, the `Account` in the recipient field of the `NFTokenOffer`, if there exist one.
* The `NFTokenOffer` against which `NFTokenAcceptOffer` transaction is placed specifies an `expiration` time and the close time field of the parent of the ledger in which the transaction would be included has already passed.
* The `NFTokenOffer` against which `NFTokenAcceptOffer` transaction is placed to buy or sell the `NFToken `is owned by the account executing the `NFTokenAcceptOffer`.

A side-effect of such failures is the removal of the `NFTokenOffer` object and the refund of the incremental reserve as if the offer had been cancelled. This necessitates the use of an appropriate `tec` class error.

If the transaction is executed successfully then:

* The relevant `NFTtoken` changes ownership, meaning that the token is removed from the `NFTokenPage` of the existing `owner` and added to the `NFTokenPage` of the new `owner`.
* Funds are transferred from the buyer to the seller, as specified in the `NFTokenOffer`. If the corresponding `NFToken` offer specifies a `TransferRate`, then the `issuer` receives the specified percentage, with the balance going to the seller of the `NFToken`.


### Brokered Mode

In brokered mode, the `NFTokenAcceptOffer` transaction fails if:

* The `buy` `NFTokenOffer` against which the  `NFTokenAcceptOffer` transaction is placed is owned by the account executing the transaction.
* The `sell` `NFTokenOffer` against which the `NFTokenAcceptOffer` transaction is placed is owned by the account executing the transaction.
* The account that placed the offer to sell the `NFToken` is not, at the time of execution, the current owner of the corresponding `NFToken`.
* Either offer (`buy` or `sell`) specifies an `expiration` time and the close time field of the parent of the ledger in which the transaction would be included has already passed.


## Fields


| Field           | JSON Type        | [Internal Type][] | Description         |
|:----------------|:-----------------|:------------------|:--------------------|
| `TransactionType` | String | UInt16 | Transaction type `NFTokenAcceptOffer`. The integer identifier is 29. |
| `SellOffer` | String | Hash256 | _(Optional)_ Identifies the `NFTokenOffer` that offers to sell the `NFToken`. |
| `BuyOffer` | String | Hash256 | _(Optional)_ Identifies the `NFTokenOffer` that offers to buy the `NFToken`. |
| `BrokerFee` | [Currency Amount][] | Amount | _(Optional)_ This field is only valid in brokered mode, and specifies the amount that the broker keeps as part of their fee for bringing the two offers together; the remaining amount is sent to the seller of the NFToken being bought. If specified, the fee must be such that, prior to accounting for the transfer fee charged by the issuer, the amount that the seller would receive is at least as much as the amount indicated in the sell offer. |

In direct mode, you must specify **either** the `SellOffer` or the `BuyOffer`. In brokered mode, you must specify **both** the `SellOffer` and the `BuyOffer`.

This functionality is intended to allow the `owner` of an `NFToken` to offer their token for sale to a third party broker, who may then attempt to sell the `NFToken` on for a larger amount, without the broker having to own the `NFToken` or custody funds.

If both offers are for the same asset, it is possible that the order in which funds are transferred might cause a transaction that would succeed to fail due to an apparent lack of funds. To ensure deterministic transaction execution and maximize the chances of successful execution, the account attempting to buy the `NFToken` is debited first. Funds due to the broker are credited _before_ crediting the seller.

In brokered mode, The offers referenced by `BuyOffer` and `SellOffer` must both specify the same `TokenID`; that is, both must be for the same `NFToken`.



<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
