# CLI Skill Test Results

**Date**: 2026-03-31
**CLI Version**: 0.4.0
**Auth Method**: API key (configured in CLI)
**Agent**: Cursor (Claude opus-4-6)

---

## Part 1: CLI-Supported Tasks

## Task 1 — Get ETH balance

**Status**: PASS
**Approach**: Used `alchemy balance` on the default `eth-mainnet` network.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct and complete. Returned `32.147257835992150606` ETH (`32147257835992150606` wei) on `eth-mainnet`.
**Notes**: None.

## Task 2 — Get ETH balance on a different network

**Status**: PASS
**Approach**: Re-ran the balance query with `-n base-mainnet`.
**Command(s) used**: `alchemy --json --no-interactive -n base-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct. Returned `0.0724190485030745` ETH on `base-mainnet`.
**Notes**: None.

## Task 3 — Get ERC-20 token balances

**Status**: PASS
**Approach**: Used `tokens balances` with `--metadata` to get names, symbols, decimals, and formatted balances.
**Command(s) used**: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`
**Output quality**: Strong. Returned 80 token balances on the first page with a `pageKey`, including `formattedBalance` values and token metadata. Sample entries: `HOLY` (500), `ANON` (200000), `LYRA` (100000), `CULT` (1150000).
**Notes**: The wallet holds thousands of tokens, many of which are spam or airdrops. Some entries have null metadata.

## Task 4 — Get token metadata

**Status**: PASS
**Approach**: Queried the USDC contract directly through the token metadata wrapper.
**Command(s) used**: `alchemy --json --no-interactive tokens metadata 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
**Output quality**: Correct and complete. Returned `name: USDC`, `symbol: USDC`, `decimals: 6`, and a logo URL.
**Notes**: None.

## Task 5 — Get token allowance

**Status**: PARTIAL
**Approach**: The dedicated `tokens allowance` subcommand failed with an RPC parameter error (`invalid 1st argument: input value was not an object`), so I recovered by calling `alchemy_getTokenAllowance` through `alchemy rpc`.
**Command(s) used**:

- Failed: `alchemy --json --no-interactive tokens allowance --owner 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --spender 0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B --contract 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
- Succeeded: `alchemy --json --no-interactive rpc alchemy_getTokenAllowance '{"owner":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","spender":"0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B","contract":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48"}'`

**Output quality**: The recovered result was correct (`"0"` allowance), but the advertised CLI subcommand did not work.
**Notes**: This is a CLI serialization bug in `@alchemy/cli` 0.4.0 — the wrapper doesn't correctly package the three addresses into the RPC params object.

## Task 6 — Get spot prices

**Status**: PASS
**Approach**: Used the Prices API wrapper with comma-separated symbols.
**Command(s) used**: `alchemy --json --no-interactive prices symbol ETH,USDC`
**Output quality**: Correct. Returned current USD prices for both assets with timestamps. At run time: `ETH = $2093.22`, `USDC = $0.9996`.
**Notes**: None.

## Task 7 — Get historical prices

**Status**: PASS
**Approach**: Queried the historical prices endpoint with a JSON body for the requested date window.
**Command(s) used**: `alchemy --json --no-interactive prices historical --body '{"symbol":"ETH","startTime":"2025-01-01T00:00:00Z","endTime":"2025-01-02T00:00:00Z"}'`
**Output quality**: Correct. Returned two ETH/USD datapoints: `$3336.62` on `2025-01-01` and `$3348.97` on `2025-01-02`.
**Notes**: None.

## Task 8 — List NFTs

**Status**: PASS
**Approach**: Queried owned NFTs with `--limit 3` to match the requested scope.
**Command(s) used**: `alchemy --json --no-interactive nfts 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --limit 3`
**Output quality**: Correct and concise. Returned 3 NFTs: `vitalik.wei` (Wei Name Service), `vitalik.cloud` (NamefiNFT), and `Sergeant WhiskerBlast` (Buttpluggy). Also reported `totalCount: 27364`.
**Notes**: None.

## Task 9 — Get NFT metadata

**Status**: PASS
**Approach**: Queried BAYC token 1 with the NFT metadata wrapper.
**Command(s) used**: `alchemy --json --no-interactive nfts metadata --contract 0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D --token-id 1`
**Output quality**: Good. Returned contract name `BoredApeYachtClub`, collection name `Bored Ape Yacht Club`, image URLs, and 5 metadata attributes (`Grin`, `Vietnam Jacket`, `Orange`, `Blue Beams`, `Robot`).
**Notes**: Normalized `name` and `description` fields were null, but raw metadata was present and usable.

## Task 10 — Get transfer history

**Status**: PASS
**Approach**: Used `alchemy_getAssetTransfers` through the CLI's raw RPC mode with `order: "desc"` to get the most recent transfers.
**Command(s) used**: `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromBlock":"0x0","toBlock":"latest","fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["erc20"],"withMetadata":true,"maxCount":"0x5","order":"desc"}'`
**Output quality**: Correct. Returned 5 recent ERC-20 transfers, all `BTT` (500 each) from February 2026.
**Notes**: Used raw RPC directly rather than the `transfers` wrapper because the wrapper defaults to ascending order.

## Task 11 — Get latest block

**Status**: PASS
**Approach**: Queried `block latest` and inspected the block header.
**Command(s) used**: `alchemy --json --no-interactive block latest`
**Output quality**: Correct and complete. Returned block number `0x17a1bf2`, hash, miner `0x95222290dd7278aa3ddd389cc1e1d165cc4bafe5` (beaverbuild.org), `gasUsed`, `baseFeePerGas`, and full transaction list.
**Notes**: None.

## Task 12 — Get gas prices

**Status**: PASS
**Approach**: Used the dedicated gas wrapper.
**Command(s) used**: `alchemy --json --no-interactive gas`
**Output quality**: Correct. Returned `gasPrice: 0.237579651 Gwei` and `maxPriorityFeePerGas: 0.000000024 Gwei`.
**Notes**: None.

## Task 13 — Look up a transaction

**Status**: PASS
**Approach**: Queried the provided hash directly with `alchemy tx`.
**Command(s) used**: `alchemy --json --no-interactive tx 0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060`
**Output quality**: Correct. Returned transaction hash, sender (`0xa1e4380a...`), recipient (`0x5df9b879...`), legacy `type: 0x0`, block `0xb443`, and value `0x7a69`.
**Notes**: This is a historic early-Ethereum transaction.

## Task 14 — Raw JSON-RPC call

**Status**: PASS
**Approach**: Called `eth_blockNumber` via the CLI's raw RPC wrapper.
**Command(s) used**: `alchemy --json --no-interactive rpc eth_blockNumber`
**Output quality**: Correct. Returned the current block number as hex: `0x17a1bf2`.
**Notes**: None.

## Task 15 — Simulate a transaction

**Status**: PASS
**Approach**: Used `simulate asset-changes` with a 1 ETH transfer to the zero address.
**Command(s) used**: `alchemy --json --no-interactive simulate asset-changes --tx '{"from":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","to":"0x0000000000000000000000000000000000000000","value":"0xde0b6b3a7640000"}'`
**Output quality**: Correct. Returned one native asset change showing a `1` ETH transfer from the source address to the zero address, with `gasUsed: 0x5208` (21000 gas).
**Notes**: None.

## Task 16 — List available networks

**Status**: PASS
**Approach**: Listed all configured network definitions, then filtered to the Solana family using the `id` field.
**Command(s) used**: `alchemy --json --no-interactive network list`
**Output quality**: Complete. Returned `166` networks total and correctly identified `solana-devnet` and `solana-mainnet` as the Solana entries.
**Notes**: None.

## Task 17 — Solana RPC

**Status**: PASS
**Approach**: Used the Solana RPC wrapper to call `getSlot`.
**Command(s) used**: `alchemy --json --no-interactive solana rpc getSlot`
**Output quality**: Correct. Returned current slot `410156863`.
**Notes**: None.

---

## Part 2: Tasks NOT Directly Supported by CLI

## Task 18 — Explain webhook setup

**Status**: PASS
**Approach**: Used the local reference files for address activity webhooks (`webhooks-address-activity.md`) and signature verification (`webhooks-verify-signatures.md`).
**Command(s) used**: None (knowledge-based).
**Output quality**: Complete. Covered the webhook creation endpoint (`POST /create-webhook` at the Alchemy dashboard API), `webhook_type: ADDRESS_ACTIVITY`, required `addresses[]` array, the expected activity payload (array of activities with tx hash, from/to, value, token metadata), and HMAC-SHA256 verification of `X-Alchemy-Signature` against the raw request body using the webhook signing secret. Distinguished between the dashboard auth token and the webhook signing key.
**Notes**: None.

## Task 19 — Parse a token balance response

**Status**: PASS
**Approach**: Converted the raw hex balance to decimal and applied USDC's 6 decimals.
**Command(s) used**: None (calculation).
**Output quality**: Correct. `0x5f5e100` = `100000000` base units ÷ 10^6 = `100 USDC`. The contract address `0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` confirms this is USDC.
**Notes**: None.

## Task 20 — Recommend an ecosystem library

**Status**: PASS
**Approach**: Synthesized a standard React + EVM integration stack around Alchemy RPC and enhanced APIs.
**Command(s) used**: None (knowledge-based).
**Output quality**: Good recommendation. Suggested `wagmi` + `viem` for React hooks and type-safe RPC access, `RainbowKit` or `ConnectKit` for wallet connection UX, and optional `alchemy-sdk` for enhanced Alchemy endpoints (NFTs, token balances, transfers). Correct transport pattern: `http("https://eth-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}")`.
**Notes**: The local reference bundle did not contain dedicated ecosystem integration files, so this relied on general knowledge.

## Task 21 — Account Kit / Smart Wallets

**Status**: PASS
**Approach**: Combined the local Account Kit (`wallets-account-kit.md`), Smart Wallets (`wallets-smart-wallets.md`), and Gas Manager (`wallets-gas-manager.md`) reference files with ERC-4337 knowledge.
**Command(s) used**: None (knowledge-based).
**Output quality**: Solid answer. Explained Account Kit as Alchemy's SDK for embedded wallet onboarding (email/social login creates a smart wallet under the hood), smart wallets as ERC-4337 accounts enabling programmable features (session keys, batched transactions, gas sponsorship), and gas sponsorship via Gas Manager policies that define which contracts/methods to sponsor and spend caps. Noted the bundler + paymaster infrastructure requirement and the gotcha that some dApps require EOA-only signatures.
**Notes**: The local references are overview-level; full copy-paste implementation would require the official Account Kit docs.

## Task 22 — Rate limits and compute units

**Status**: PASS
**Approach**: Used the local rate limits reference (`operational-rate-limits-and-compute-units.md`) plus general Alchemy pricing knowledge.
**Command(s) used**: None (knowledge-based).
**Output quality**: Complete. Explained compute-unit (CU) metering: each API method has a CU cost (e.g. `eth_blockNumber` ≈ 10 CU, `eth_call` ≈ 26 CU, `alchemy_getTokenBalances` ≈ 20 CU), rate limiting returns `429` responses requiring exponential backoff, and plans are tiered by monthly CU allowance and per-second throughput. Recommended batching requests, using Data APIs over repeated JSON-RPC calls, caching metadata aggressively, and using `pageKey` pagination for large datasets.
**Notes**: Exact plan limits change; the official pricing page is the source of truth.

## Task 23 — WebSocket subscriptions

**Status**: PASS
**Approach**: Used the local WebSocket subscription reference (`node-websocket-subscriptions.md`) and pending transactions recipe (`recipes-subscribe-pending-txs.md`).
**Command(s) used**: None (knowledge-based).
**Output quality**: Complete. Included the correct WebSocket URL pattern (`wss://eth-mainnet.g.alchemy.com/v2/$ALCHEMY_API_KEY`), `eth_subscribe` request shape, subscription types (`newHeads`, `logs`, `newPendingTransactions`, `alchemy_minedTransactions`), a Node.js code example using the `ws` library, and operational guidance (reconnect + resubscribe, de-duplicate by block hash or log index, high volume warning for `newPendingTransactions`).
**Notes**: None.

## Task 24 — Solana DAS query

**Status**: PASS
**Approach**: Queried the provided Solana address with the CLI's `solana das` wrapper using `getAssetsByOwner`.
**Command(s) used**: `alchemy --json --no-interactive solana das getAssetsByOwner '{"ownerAddress":"GsbwXfJraMomNxBcjYLcG3mxkBUiyWXAB32fGbSQQRre","page":1,"limit":5}'`
**Output quality**: Correct. The request succeeded and returned `total: 0` items — the address holds no NFTs.
**Notes**: An empty result is a valid DAS response, not a failure.

---

## Part 3: Multi-Step and Advanced Tasks

## Task 25 — Multi-step workflow

**Status**: PARTIAL
**Approach**: Combined multiple CLI calls to build a portfolio view: ETH balance, ETH price, portfolio token data with prices, and recent transfers (descending).
**Command(s) used**:

- `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
- `alchemy --json --no-interactive prices symbol ETH`
- `alchemy --json --no-interactive portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'`
- `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromBlock":"0x0","toBlock":"latest","fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["external","erc20"],"withMetadata":true,"maxCount":"0x3","order":"desc"}'`

**Output quality**: Produced a usable summary: `32.15 ETH` (~$67.3k at $2094/ETH), recent transfers (0.01 ETH send on 2026-02-28, two 500 BTT sends on 2026-02-22). The "top 5 ERC-20 by USD" portion was weak: the portfolio endpoint returns 100 tokens per page with no server-side sort by value, and the vast majority of this wallet's thousands of tokens are spam/airdrops with no price data. Only a few tokens like `CULT`, `CANA`, and `BEAR` had USD prices, and those prices were negligible.
**Notes**: A production-quality "top 5 by USD value" for this wallet would require multi-page pagination + spam filtering. The portfolio endpoint doesn't support sorting by value, making this fundamentally expensive for wallets with massive token counts.

## Task 26 — Pagination

**Status**: PASS
**Approach**: Fetched three sequential pages of token balances using `--page-key`, summing results.
**Command(s) used**:

- Page 1: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata` → 80 tokens
- Page 2: `alchemy --json --no-interactive tokens balances ... --page-key "0x01f3106b674341ff1f18af133e7c844fc43aacdf"` → 86 tokens
- Page 3: `alchemy --json --no-interactive tokens balances ... --page-key "0x052493c962b90f081edaa3ba4b2291fc755b52bf"` → 81 tokens

**Output quality**: Correct. Total across 3 pages: `247` tokens. The third page still returned another `pageKey`, confirming more data exists.
**Notes**: Pagination works reliably through the CLI's `--page-key` flag.

## Task 27 — Error recovery

**Status**: PASS
**Approach**: Intentionally used the wrong network slug `ethereum-mainnet`, captured the structured error, then retried with the correct slug `eth-mainnet`.
**Command(s) used**:

- Failed: `alchemy --json --no-interactive -n ethereum-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
- Recovered: `alchemy --json --no-interactive -n eth-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

**Output quality**: Excellent. The failure returned a clear `INVALID_ARGS` error with the message `Unknown network 'ethereum-mainnet'. Run 'alchemy network list' to see available networks.` Recovery with `eth-mainnet` worked immediately and returned the correct balance.
**Notes**: The CLI's error messages are actionable and machine-friendly.

---

## Summary

**Total**: 27/27
**Pass**: 25
**Partial**: 2
**Fail**: 0

### Key gaps found

- `alchemy tokens allowance` is broken in CLI `0.4.0`; a raw RPC workaround is required for Task 5. The wrapper fails to serialize the three address parameters into the expected RPC object.
- Building a reliable "top 5 ERC-20 by USD value" for this wallet is expensive and noisy without spam filtering. The portfolio endpoint does not sort by value and returns only 100 tokens per page, while the wallet holds thousands of low-quality tokens. Most tokens have no price data at all.
- The `transfers` wrapper defaults to ascending order, making raw RPC the better path when a task explicitly asks for the most recent transfers.

### Key strengths

- CLI + `--json --no-interactive` provides a clean, automatable interface. Both success and error outputs are structured JSON.
- Raw RPC mode (`alchemy rpc`) cleanly covers wrapper gaps without leaving the CLI surface.
- Solana RPC and Solana DAS work from the same CLI — no separate tooling needed.
- The CLI's errors are clear, typed (`INVALID_ARGS`, `RPC_ERROR`), and actionable with recovery hints.
- `tokens balances --metadata` gives formatted balances and easy `--page-key` pagination out of the box.
- The local reference bundle (webhooks, WebSocket subscriptions, Account Kit, rate limits) was sufficient to answer all knowledge-based questions confidently.
- Network listing and filtering is straightforward: 166 networks with structured metadata.
- Transaction simulation returns clean asset change diffs with human-readable amounts.
