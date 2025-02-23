---
html: nftokenmint.html
parent: transaction-types.html
blurb: Use TokenMint to issue new NFTs.
labels:
  - Non-fungible Tokens, NFTs
status: not_enabled
---
# NFTokenMint
[[Source]](https://github.com/ripple/rippled/blob/xls20/src/ripple/app/tx/impl/NFTokenMint.cpp)
{% include '_snippets/nfts-disclaimer.md' %}

The `NFTokenMint` transaction creates an non-fungible token and adds it to the relevant [NFTokenPage object][] of the `minter` as an [NFToken][] object. A required parameter to this transaction is the `Token` field specifying the actual token. This transaction is the only opportunity the `minter` has to specify any token fields that are defined as immutable (for example, the `TokenFlags`).

If the transaction is successful, the newly minted token is owned by the account (the `minter` account) that executed the transaction. If needed, the server creates a new `NFTokenPage`  for the account and applies a reserve charge.


## Example {{currentpage.name}} JSON


```json
{
  "TransactionType": "NFTokenMint",
  "Account": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B",
  "Issuer": "rNCFjv8Ek5oDrNiMJ3pw6eLLFtMjZLJnf2",
  "TransferFee": 314,
  "TokenTaxon": 0,
  "Flags": 2147483659,
  "Fee": 10,
  "URI": "ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf4dfuylqabf3oclgtqy55fbzdi",
  "Memos": [
        {
            "Memo": {
                "MemoType":
                  "687474703A2F2F6578616D706C652E636F6D2F6D656D6F2F67656E65726963",
                "MemoData": "72656E74"
            }
        }
    ]
}
```


This transaction assumes that the issuer, `rNCFjv8Ek5oDrNiMJ3pw6eLLFtMjZLJnf2`, has set the `MintAccount` field in its `AccountRoot` to `rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B`, thereby authorizing that account to mint tokens on its behalf.



{% include '_snippets/tx-fields-intro.md' %}

| Field         | JSON Type           | [Internal Type][] | Description        |
|:--------------|:--------------------|:------------------|:-------------------|
| `TokenTaxon` | Number | UInt32 | The taxon associated with the token. The taxon is generally a value chosen by the minter of the token. A given taxon can be used for multiple tokens. Taxon identifiers greater than or equal to `0x80000000` are disallowed. |
| `Issuer` | String | AccountID | _(Optional)_ The issuer of the token, if the sender of the account is issuing it on behalf of another account. This field must be omitted if the account sending the transaction is the issuer of the `NFToken`. If provided, the issuer's [AccountRoot object][] must have the `MintAccount` field set to sender of this transaction (this transaction's `Account` field). |
| `TransferFee` | Number | UInt16 | _(Optional)_ The value specifies the fee charged by the issuer for secondary sales of the Token, if such sales are allowed. Valid values for this field are between 0 and 9999 inclusive, allowing transfer rates of between 0.00% and 99.99% in increments of 0.01. If this field is provided, the transaction MUST have the [`tfTransferable` flag](#nftokenmint-flags) enabled. |
| `URI` | String | Blob | _(Optional)_ A URI that points to the data or metadata associated with the NFT. This field need not be an HTTP or HTTPS URL; it could be an IPFS URI, a magnet link, immediate data encoded as an [RFC2379 "data" URL](https://datatracker.ietf.org/doc/html/rfc2397), or even an issuer-specific encoding. The URI is NOT checked for validity, but the field is limited to a maximum length of 256 bytes.


## NFTokenMint Flags

Transactions of the NFTokenMint type support additional values in the [`Flags` field](transaction-common-fields.html#flags-field), as follows:

| Flag Name     | Hex Value    | Decimal Value | Description                   |
|:--------------|:-------------|:--------------|:------------------------------|
| `tfBurnable` | `0x00000001` | 1 | Allow the issuer (or an entity authorized by the issuer) to destroy the minted `NFToken`. (The `NFToken`'s owner can _always_ do so.) |
| `tfOnlyXRP` | `0x00000002` | 2 | The minted `NFToken` can only be bought or sold for XRP. This may be desirable if the token has a transfer fee and the issuer does not want to receive fees in non-XRP currencies. |
| `tfTrustLine` | `0x00000004` | 4 | Automatically create [trust lines](trust-lines-and-issuing.html) from the issuer to hold transfer fees received from transferring the minted `NFToken`. |
| `tfTransferable` | `0x00000008` | 8 | The minted `NFToken` can be transferred to others. If this flag is _not_ enabled, the token can still be transferred _from_ or _to_ the issuer. |


## Embedding additional information

If you need to specify additional information during minting (for example, details identifying a property by referencing a particular [plat](https://en.wikipedia.org/wiki/Plat), a vehicle by specifying a [VIN](https://en.wikipedia.org/wiki/Vehicle_identification_number), or other object-specific descriptions) you can use a [transaction memo](transaction-common-fields.html#memos-field). Memos are a part of the signed transaction and are available from historical archives, but are not stored in the ledger's state data.

## Error Cases

- If the `TransferFee` field is not within the acceptable range (0 to 9999 inclusive) the transaction fails with `temBAD_TRANSFER_RATE`.
- If the `URI` field is longer than 256 bytes, the transaction fails with `temMALFORMED`.
- If the `Issuer` field refers to an account that does not exist, the transaction fails with `tecNO_ISSUER`.
- If account referenced by the `Issuer` field has not authorized this transaction's sender (using the `MintAccount` setting) to mint `NFToken`s on their behalf, the transaction fails with `tecNO_PERMISSION`.
- If the owner would not meet the updated [reserve requirement](reserves.html) after minting the token, the transaction fails with `tecINSUFFICIENT_RESERVE`. Note that new `NFToken`s only increase the owner's reserve if it requires a new [NFTokenPage object][], which can each hold up to 32 NFTs.
- If the `Issuer`'s `MintedTokens` field maxes out, the transaction fails with `tecMAX_SEQUENCE_REACHED`. This is only possible if 2<sup>32</sup>-1 `NFToken`s have been minted in total by the issuer or on their behalf.

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
