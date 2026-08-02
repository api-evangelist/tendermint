---
name: Broadcast and track a transaction
description: Submit a signed transaction to a Tendermint node and confirm it was included in a block.
api: openapi/tendermint-rpc-openapi-original.yml
operations: [check_tx, broadcast_tx_sync, broadcast_tx_commit, tx, subscribe]
---

# Broadcast and track a transaction

Submit an already-signed, encoded transaction to a Tendermint node and confirm
inclusion. Transactions are built and signed by the application layer (e.g.
Cosmos SDK); the RPC only relays the opaque bytes.

## Steps

1. **(Optional) Dry-run.** Call `check_tx` with the `tx` bytes to validate
   against the mempool without broadcasting. A non-zero `code` means it would be
   rejected.
2. **Broadcast.** Choose one:
   - `broadcast_tx_sync` — returns after CheckTx (mempool admission), not
     commit. Preferred for most clients.
   - `broadcast_tx_commit` — waits for the transaction to be committed in a
     block. Convenient but can time out under load; do not use in high-throughput
     paths.
   - `broadcast_tx_async` — returns immediately with no result.
3. **Track inclusion.** Broadcast is NOT idempotent at the RPC layer; the
   mempool deduplicates by transaction hash. To confirm inclusion, either:
   - poll `tx` with the transaction hash until it resolves, or
   - `subscribe` over the `/websocket` endpoint with
     `tm.event='Tx' AND tx.hash='<HASH>'` and wait for the event.
4. **Resend policy.** If nothing appears after a couple of blocks, resend the
   same signed bytes (same hash → deduplicated, safe). If it still fails, send to
   a different node.

## Rules

- Never re-sign/rebuild on retry — resend the identical bytes so the hash is
  stable and the mempool dedupes rather than double-submitting.
- A 200 from broadcast means the node accepted the request, not that the tx
  succeeded — always check `tx_result.code` / the committed `tx` result.
- Transport errors return HTTP 500 with a JSONRPC `error` object
  (see `errors/tendermint-problem-types.yml`).
