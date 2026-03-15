# Native Transfer Fees for URITokens

**Author:** <your name>  
**Created:** 2026-03-15  
**Status:** Proposal  
**Category:** Amendment  

## Abstract

This proposal adds an optional immutable `TransferFee` field to URITokens on Xahau.

The `TransferFee` is specified at mint time, stored on the URIToken ledger object, and automatically applied during qualifying native secondary sales. This proposal also adds an optional `TransferFeeRecipient` field, allowing the minter to direct transfer fees to a wallet other than the issuer account.

The design intentionally borrows the bounded mint-time transfer-fee concept familiar from XLS-20 while preserving the lighter and separate architecture of URITokens.

This proposal does not attempt to make URITokens equivalent to XLS-20 NFTs, nor does it require Hooks-based royalty enforcement. Instead, it introduces a minimal native transfer-fee primitive for common creator and marketplace use cases.

## Motivation

URITokens were designed as lightweight first-class non-fungible objects. That simplicity is valuable, but it also means creator fee behavior is not natively standardized.

A native transfer-fee mechanism would provide:

1. A standardized transfer fee declared at mint time.
2. Immutable fee terms after issuance.
3. Automatic fee payment on qualifying native secondary sales.
4. A standard on-ledger representation visible to wallets, explorers, and marketplaces.
5. Optional direction of transfer fees to a wallet other than the issuer.

This would give creators and marketplaces a simple native option without requiring Hooks-based enforcement stacks for common use cases.

## Rationale

URITokens and XLS-20 NFTs are different systems and should remain different systems.

This proposal borrows only the transfer-fee concept from XLS-20, not its full object model, offer system, ID packing, or related semantics.

The design is intentionally minimal:

- optional
- immutable
- bounded
- applied only on qualifying native sale execution paths
- optionally payable to a designated recipient wallet

This keeps URITokens simple while supporting a common creator and issuer requirement.

## Specification

### Amendment

This proposal introduces a new amendment to the Xahau protocol.

When enabled, the amendment adds:

- a new optional `TransferFee` field to `URITokenMint`
- a new optional `TransferFeeRecipient` field to `URITokenMint`
- corresponding optional fields on the `URIToken` ledger object
- transfer-fee-aware processing for qualifying `URITokenBuy` transactions

## New Serialized Fields

Add the following optional serialized fields:

- `sfTransferFee`, `UInt16`
- `sfTransferFeeRecipient`, `AccountID`

### `TransferFee` Semantics

`TransferFee` represents a fee rate in tenths of a basis point.

This proposal adopts the same numeric convention commonly used for transfer-fee style fields:

- minimum: `0`
- maximum: `50000`
- precision: `0.001%`
- `1000` = `1.000%`
- `2500` = `2.500%`
- `10000` = `10.000%`
- `50000` = `50.000%`

If absent, the transfer fee is considered `0`.

Values above `50000` MUST cause the mint transaction to fail.

### `TransferFeeRecipient` Semantics

`TransferFeeRecipient` identifies the account that receives transfer-fee proceeds during qualifying royalty-bearing secondary sales.

Rules:

1. If `TransferFeeRecipient` is absent, the fee recipient defaults to the `Issuer`.
2. If `TransferFeeRecipient` is present, it MUST be a valid Xahau account address.
3. `TransferFeeRecipient` MUST NOT be changed after minting.
4. `TransferFeeRecipient` MAY be the issuer or a different wallet.

This allows creators, DAOs, treasury wallets, or managed revenue wallets to receive secondary-sale fees without forcing the issuer account itself to collect them.

## Modified Transaction: `URITokenMint`

Add optional fields:

- `TransferFee`
- `TransferFeeRecipient`

### Mint Rules

When a `URITokenMint` transaction includes `TransferFee`:

1. The value MUST be between `0` and `50000` inclusive.
2. The value MUST be copied into the created `URIToken` ledger object.
3. The value becomes immutable after minting.

When a `URITokenMint` transaction includes `TransferFeeRecipient`:

1. The account MUST be a valid address.
2. The value MUST be copied into the created `URIToken` ledger object.
3. The value becomes immutable after minting.

If `TransferFee` is absent, the token has no transfer-fee obligation.

If `TransferFeeRecipient` is absent, the recipient defaults to the issuer.

If `TransferFee` is `0`, implementations MAY omit `TransferFeeRecipient` from the created ledger object.

## Modified Ledger Object: `URIToken`

Add optional fields:

- `TransferFee`
- `TransferFeeRecipient`

Example object:

```json
{
  "LedgerEntryType": "URIToken",
  "Flags": 0,
  "Issuer": "rIssuerExample...",
  "Owner": "rOwnerExample...",
  "URI": "697066733A2F2F62616679...",
  "TransferFee": 2500,
  "TransferFeeRecipient": "rRoyaltyWalletExample..."
}
