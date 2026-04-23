# Units patch — NIP-47 struct field renames

Status: **on branch `patch-units`**, pending PR.

## What this patch does

Cascades [Red-Token/nips#1](https://github.com/Red-Token/nips/pull/1) (merged) into the NIP-47 wire structs defined in `crates/nostr/src/nips/nip47.rs`. The new spec codifies the `_sats` suffix rule: every amount-bearing field whose value is denominated in sats carries a `_sats` suffix; all other amounts remain plain-named and msats.

## Field renames

| Struct | Field | Change |
| --- | --- | --- |
| `PayOnchainRequest` | `amount` | ⟶ `amount_sats` (rename only — value was already sats per doc comment) |
| `GetBalanceResponse` | `onchain_balance` | ⟶ `onchain_balance_sats` (rename **plus** semantic unit flip msats → sats — see note below) |
| `AddressTransaction` | `amount` | ⟶ `amount_sats` (rename; value was already sats per doc comment) |
| `LookupAddressResponse` | `total_received` | ⟶ `total_received_sats` (rename; value was already sats per doc comment) |
| `AddressEntry` | `total_received` | ⟶ `total_received_sats` (rename; value was already sats per doc comment) |

### `onchain_balance_sats` — the one semantic flip

Before this patch, `GetBalanceResponse.onchain_balance` was documented as msats. The spec patch pins on-chain balances to sats to match the physical precision of UTXOs. `ldk-controller` was filling the old field with msat values; downstream, its `b95b4a8` rename work needs a follow-up assignment change so the new field gets a sat value (divide internal msat total by 1000).

This is flagged in the ldk-controller cascade issue ([dukeh3/ldk-controller#121](https://github.com/dukeh3/ldk-controller/issues/121)).

## Deferred

**`list_transactions` polymorphism is not included in this patch.** The spec says Lightning entries keep `amount` / `fees_paid` (msats) while on-chain entries use `amount_sats` / `fees_paid_sats`. The current Rust type `LookupInvoiceResponse` is a single struct reused for both invoices and list-transactions entries, so adding the polymorphism requires a bigger refactor (probably a tagged enum keyed on `payment_method`). Scope for a follow-up patch.

## Cascade after merge

1. `dukeh3/ldk-controller` — bump `Cargo.toml` rev to this commit; update field access (`.amount` → `.amount_sats`, etc.); fix the `onchain_balance_sats` population to deliver sat values (divide the service's msat total by 1000). This is the second half of ldk-controller#121; the NNC half is already on `cascade-units` branch there.
2. `dukeh3/ldk-controller-stage-test` (`nwc-check` binary) — bump rev and update field access for `GetBalanceResponse.onchain_balance_sats`.
3. SDK (`DarkWebDivingClub/nostr-js-nwc`) — independent TS types. See its own cascade task.
4. Integration test — verifies end-to-end once service is deployed.

## Acceptance

- [x] Five structs renamed in `crates/nostr/src/nips/nip47.rs`.
- [x] `cargo check -p nostr -p nwc` clean.
- [ ] PR merged.
- [ ] ldk-controller and nwc-check bumped to new rev.
- [ ] Follow-up issue opened for `list_transactions` polymorphism.
