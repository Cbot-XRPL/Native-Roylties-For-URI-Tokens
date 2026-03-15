---
title: Native Royalties for URITokens
description: Adds an optional immutable royalty field to URITokens and applies it to qualifying native secondary sales.
author: Cbot Labs LLC
status: Proposal
category: Amendment
created: 2026-03-15
---

# Native Royalties for URITokens

## Abstract

This proposal adds an optional immutable royalty field to URITokens on Xahau.

The royalty field is specified at mint time, stored on the URIToken ledger object, and automatically applied during qualifying native secondary sales. The design intentionally follows the simple, bounded, mint-time royalty model used by XLS-20 while preserving the separate and lighter architecture of URITokens.

This proposal does not attempt to make URITokens equivalent to XLS-20 NFTs, nor does it require Hooks-based royalty enforcement. Instead, it introduces a minimal native royalty primitive for common marketplace and creator use cases.

## Motivation

URITokens were designed as lightweight first-class non-fungible objects. This simplicity is a major advantage, but it also means that creator royalty behavior is not natively standardized.

Although royalty and brokerage behavior has been explored for URITokens through hook-centric approaches, many use cases would benefit from a simpler native mechanism that wallets, explorers, issuers, and marketplaces can implement consistently without requiring audited hook stacks, issuer blackholing patterns, or specialized brokerage infrastructure.

Creators and marketplaces commonly want the following:

1. A royalty percentage declared at mint time.
2. Immutable royalty terms after issuance.
3. Automatic royalty payment on qualifying native secondary sales.
4. A standard on-ledger representation visible to wallets and indexers.

This amendment addresses those needs while keeping URIToken transaction semantics as simple as possible.

## Rationale

XLS-20 and URITokens solve different problems and should remain distinct designs.

XLS-20 NFTs include more complex object and transfer semantics. URITokens instead prioritize first-class ownership, simpler object handling, and lighter transaction flow. For this reason, this proposal borrows only the royalty concept from XLS-20, not its full object model, ID packing, offer semantics, or trustline-related behavior.

The royalty mechanism proposed here is intentionally minimal:

- optional
- immutable
- bounded
- applied only on qualifying native sale execution paths

This is sufficient to support many common creator-economy use cases without changing the core character of URITokens.

## Specification

### Amendment

This proposal introduces a new amendment to the Xahau protocol.

When enabled, the amendment adds:

- a new optional serialized field on `URITokenMint`
- a corresponding optional field on the `URIToken` ledger object
- royalty-aware processing for qualifying `URITokenBuy` transactions

### New Serialized Field

Add a new optional serialized field:

- `sfRoyalty`, `UInt16`

#### Semantics

`sfRoyalty` represents a royalty rate in tenths of a basis point.

This proposal adopts the same numeric convention used by XLS-20 transfer fees:

- minimum: `0`
- maximum: `50000`
- unit precision: `0.001%`
- `1000` = `1.000%`
- `2500` = `2.500%`
- `10000` = `10.000%`
- `50000` = `50.000%`

If absent, the royalty is considered `0`.

Values above `50000` MUST cause the transaction to fail.

### Modified Transaction: `URITokenMint`

Add optional field:

- `sfRoyalty`, `UInt16`, optional

#### Mint Rules

When a `URITokenMint` transaction includes `sfRoyalty`:

1. The value MUST be between `0` and `50000` inclusive.
2. The value MUST be copied into the created `URIToken` ledger object.
3. The value becomes immutable after minting.

If `sfRoyalty` is absent, the created token has no royalty obligation.

### Modified Ledger Object: `URIToken`

Add optional field:

- `Royalty`

Example object:

```json
{
  "LedgerEntryType": "URIToken",
  "Flags": 0,
  "Issuer": "rIssuerExample...",
  "Owner": "rOwnerExample...",
  "URI": "697066733A2F2F62616679...",
  "Royalty": 2500
}
