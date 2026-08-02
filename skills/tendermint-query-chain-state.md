---
name: Query Tendermint chain state
description: Read the current status of a Tendermint node and inspect blocks, transactions, and the validator set over the RPC.
api: openapi/tendermint-rpc-openapi-original.yml
operations: [health, status, block, tx, tx_search, validators]
---

# Query Tendermint chain state

Use the Tendermint RPC (JSONRPC 2.0 over HTTP, or REST-URI form) to read chain
state. RPC is unauthenticated — target a node endpoint you trust (e.g.
`https://rpc.cosmos.network` or a local node at `http://localhost:26657`).

## Steps

1. **Confirm the node is up.** Call `health` (returns empty result on success),
   then `status` to read `sync_info` (latest_block_height, catching_up) and
   `validator_info`. If `catching_up` is true, data may be stale.
2. **Read a block.** Call `block` with `height` (omit for the latest). Inspect
   `result.block.header` for chain_id, time, and proposer_address.
3. **Look up a transaction.** Call `tx` with the base64/hex `hash` and
   `prove=true` for a Merkle proof. A non-zero `tx_result.code` means the app
   rejected the transaction even though the RPC call itself succeeded.
4. **Search transactions.** Call `tx_search` with a `query` such as
   `tm.event='Tx' AND transfer.recipient='...'`. Page with `page` (1-indexed)
   and `per_page` (max 100); use `result.total_count` to know when to stop.
5. **Inspect the validator set.** Call `validators` at a `height` to read each
   validator's `address`, `voting_power`, and `proposer_priority` (paginated).

## Rules

- All calls return HTTP 200 with a JSONRPC result on success, or HTTP 500 with a
  JSONRPC `error` object on failure — read `error.message` for the cause
  (see `errors/tendermint-problem-types.yml`).
- These operations are read-only and safe to retry.
- Application-level results live inside `result` with their own `code` field;
  distinguish that from transport-level 500 errors.
