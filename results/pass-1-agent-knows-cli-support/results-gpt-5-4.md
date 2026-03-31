# CLI Skill Test Results

**Date**: 2026-03-31
**CLI Version**: 0.4.0
**Auth Method**: API key (configured in CLI)
**Agent**: Cursor (GPT-5.4)

---

## Part 1: CLI-Supported Tasks

## Task 1.1 — Get ETH balance

**Status**: PASS
**Approach**: Used `alchemy balance` on the default `eth-mainnet` network.
**Command(s) used**: `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct and complete. Returned `32.147257835992150606` ETH (`32147257835992150606` wei) on `eth-mainnet`.
**Notes**: None.

## Task 1.2 — Get ETH balance on a different network

**Status**: PASS
**Approach**: Re-ran the balance query with `-n base-mainnet`.
**Command(s) used**: `alchemy --json --no-interactive -n base-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
**Output quality**: Correct. Returned `0.0724190485030745` ETH on `base-mainnet`.
**Notes**: None.

## Task 1.3 — Get ERC-20 token balances

**Status**: PASS
**Approach**: Used `tokens balances` with `--metadata` to get names, symbols, decimals, and formatted balances.
**Command(s) used**: `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`
**Output quality**: Strong. Returned 80 token balances on the first page plus a `pageKey`, with `formattedBalance` values and token metadata. Sample entries included `HOLY`, `ANON`, `LYRA`, and `CULT`.
**Notes**: The wallet contains many spam or airdrop tokens, and some entries still have null metadata.

## Task 1.4 — Get token metadata

**Status**: PASS
**Approach**: Queried the USDC contract directly through the token metadata wrapper.
**Command(s) used**: `alchemy --json --no-interactive tokens metadata 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
**Output quality**: Correct and complete. Returned `name: USDC`, `symbol: USDC`, `decimals: 6`, and a logo URL.
**Notes**: None.

## Task 1.5 — Get token allowance

**Status**: PARTIAL
**Approach**: The dedicated `tokens allowance` subcommand failed with an RPC parameter error, so I recovered by calling `alchemy_getTokenAllowance` through `alchemy rpc`.
**Command(s) used**:

- Failed: `alchemy --json --no-interactive tokens allowance --owner 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --spender 0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B --contract 0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`
- Succeeded: `alchemy --json --no-interactive rpc alchemy_getTokenAllowance '{"owner":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","spender":"0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B","contract":"0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48"}'`

**Output quality**: The recovered result was correct and returned `"0"`, but the advertised CLI subcommand did not work as-is.
**Notes**: This looks like a CLI serialization bug in `@alchemy/cli` 0.4.0.

## Task 1.6 — Get spot prices

**Status**: PASS
**Approach**: Used the Prices API wrapper with comma-separated symbols.
**Command(s) used**: `alchemy --json --no-interactive prices symbol ETH,USDC`
**Output quality**: Correct. Returned current USD prices for both assets, including timestamps. At run time: `ETH = 2097.9488883903`, `USDC = 0.9996439367`.
**Notes**: None.

## Task 1.7 — Get historical prices

**Status**: PASS
**Approach**: Queried the historical prices endpoint with a JSON body for the requested date window.
**Command(s) used**: `alchemy --json --no-interactive prices historical --body '{"symbol":"ETH","startTime":"2025-01-01T00:00:00Z","endTime":"2025-01-02T00:00:00Z"}'`
**Output quality**: Correct. Returned two ETH/USD datapoints: `3336.6175137659` on `2025-01-01T00:00:00Z` and `3348.9672471236` on `2025-01-02T00:00:00Z`.
**Notes**: None.

## Task 1.8 — List NFTs

**Status**: PASS
**Approach**: Queried owned NFTs with a `--limit 3` cap so the response matched the requested scope.
**Command(s) used**: `alchemy --json --no-interactive nfts 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --limit 3`
**Output quality**: Correct and concise. Returned 3 NFTs with names, token IDs, and contract addresses: `vitalik.wei`, `vitalik.cloud`, and `Sergeant WhiskerBlast`.
**Notes**: The response also reported `totalCount: 27364`, so this wallet is extremely NFT-heavy.

## Task 1.9 — Get NFT metadata

**Status**: PASS
**Approach**: Queried BAYC token `1` with the NFT metadata wrapper and inspected both normalized fields and the raw metadata block.
**Command(s) used**: `alchemy --json --no-interactive nfts metadata --contract 0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D --token-id 1`
**Output quality**: Good. Returned contract name `BoredApeYachtClub`, collection name `Bored Ape Yacht Club`, image URLs, and 5 metadata attributes (`Grin`, `Vietnam Jacket`, `Orange`, `Blue Beams`, `Robot`).
**Notes**: The normalized `name` and `description` fields were null in this response, but the raw metadata was still present and usable.

## Task 1.10 — Get transfer history

**Status**: PASS
**Approach**: The plain `transfers` wrapper returned the oldest results first, so I switched to `alchemy_getAssetTransfers` through the CLI's raw RPC mode with `order: "desc"` to satisfy "last 5".
**Command(s) used**:

- Initial wrapper: `alchemy --json --no-interactive transfers --from-address 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --category erc20 --max-count 5`
- Final query: `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromBlock":"0x0","toBlock":"latest","fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["erc20"],"withMetadata":true,"maxCount":"0x5","order":"desc"}'`

**Output quality**: Correct. Returned 5 recent ERC-20 transfer events, including February 2026 `BTT` transfers with timestamps.
**Notes**: The wrapper is usable, but the raw RPC path is better when precise ordering matters.

## Task 1.11 — Get latest block

**Status**: PASS
**Approach**: Queried `block latest` and summarized the block header plus transaction count.
**Command(s) used**: `alchemy --json --no-interactive block latest`
**Output quality**: Correct and complete. The response included block number `0x17a19a6`, hash `0xdb94dd04acb9929bfc72be4e4e2a2e5dd9405d80578d71ad61640331779cffc9`, `534` transactions, `gasUsed`, and `baseFeePerGas`.
**Notes**: None.

## Task 1.12 — Get gas prices

**Status**: PASS
**Approach**: Used the dedicated gas wrapper.
**Command(s) used**: `alchemy --json --no-interactive gas`
**Output quality**: Correct. Returned `gasPrice`, `gasPriceGwei`, `maxPriorityFeePerGas`, and `maxPriorityFeePerGasGwei`. At run time: `0.405339531` Gwei gas price and `0.000448` Gwei priority fee.
**Notes**: None.

## Task 1.13 — Look up a transaction

**Status**: PASS
**Approach**: Queried the provided hash directly with `alchemy tx`.
**Command(s) used**: `alchemy --json --no-interactive tx 0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060`
**Output quality**: Correct. Returned transaction hash, sender, recipient, legacy `type: 0x0`, block `0xb443`, and value `0x7a69`.
**Notes**: This is a historic early-Ethereum transaction, so the data shape is simple and stable.

## Task 1.14 — Raw JSON-RPC call

**Status**: PASS
**Approach**: Called `eth_blockNumber` via the CLI's raw RPC wrapper.
**Command(s) used**: `alchemy --json --no-interactive rpc eth_blockNumber`
**Output quality**: Correct. Returned the current block number as hex: `0x17a19a6`.
**Notes**: None.

## Task 1.15 — Simulate a transaction

**Status**: PASS
**Approach**: Used `simulate asset-changes` with a 1 ETH transfer to the zero address.
**Command(s) used**: `alchemy --json --no-interactive simulate asset-changes --tx '{"from":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","to":"0x0000000000000000000000000000000000000000","value":"0xde0b6b3a7640000"}'`
**Output quality**: Correct. Returned one native asset change showing a `1` ETH transfer from the source address to the zero address, with `gasUsed: 0x5208`.
**Notes**: None.

## Task 1.16 — List available networks

**Status**: PASS
**Approach**: Listed all configured network definitions, then filtered to the Solana family.
**Command(s) used**: `alchemy --json --no-interactive network list`
**Output quality**: Complete. Returned `166` networks total and correctly included `solana-devnet` and `solana-mainnet` as the Solana entries.
**Notes**: The CLI output is well-structured for machine filtering.

## Task 1.17 — Solana RPC

**Status**: PASS
**Approach**: Used the Solana RPC wrapper to call `getSlot`.
**Command(s) used**: `alchemy --json --no-interactive solana rpc getSlot`
**Output quality**: Correct. Returned current slot `410138745`.
**Notes**: None.

---

## Part 2: Tasks NOT Directly Supported by CLI

## Task 2.1 — Explain webhook setup

**Status**: PASS
**Approach**: Used the local Notify reference files for webhooks overview, address activity, and signature verification.
**Command(s) used**: None.
**Output quality**: Complete. Covered `POST /create-webhook` at `https://dashboard.alchemy.com/api`, `X-Alchemy-Token`, `webhook_type: ADDRESS_ACTIVITY`, required `addresses[]`, the expected activity payload shape, and HMAC verification against `X-Alchemy-Signature` using the raw request body.
**Notes**: The distinction between the dashboard auth token and the webhook signing secret is important.

## Task 2.2 — Parse a token balance response

**Status**: PASS
**Approach**: Converted the raw hex balance to decimal and applied USDC's 6 decimals.
**Command(s) used**: None.
**Output quality**: Correct. `0x5f5e100` = `100000000` base units, which equals `100` USDC.
**Notes**: None.

## Task 2.3 — Recommend an ecosystem library

**Status**: PASS
**Approach**: Synthesized a standard React + EVM integration stack around Alchemy RPC and enhanced APIs.
**Command(s) used**: None.
**Output quality**: Good recommendation. Suggested `wagmi` + `viem` for React hooks and RPC access, `RainbowKit` or injected connectors for wallet UX, and optional `alchemy-sdk` for enhanced Alchemy endpoints. Correct transport pattern: `http("https://eth-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}")`.
**Notes**: The repo's local reference bundle did not actually contain the promised `ecosystem-*` files, so this answer depended on general ecosystem knowledge rather than bundled docs.

## Task 2.4 — Account Kit / Smart Wallets

**Status**: PASS
**Approach**: Combined the local Account Kit, Smart Wallets, and Gas Manager reference files with the standard ERC-4337 integration flow.
**Command(s) used**: None.
**Output quality**: Solid high-level answer. Explained embedded onboarding, smart wallet creation, bundler/paymaster concepts, and gas sponsorship via Gas Manager policies.
**Notes**: The local references are overview-level; for copy-paste implementation details I would still open the official Account Kit docs.

## Task 2.5 — Rate limits and compute units

**Status**: PASS
**Approach**: Used the local compute-unit guidance plus official Alchemy pricing and compute-unit documentation to fill in current plan details.
**Command(s) used**: None.
**Output quality**: Complete. Explained CU-based metering, sample method costs (`eth_blockNumber` = 10 CU, `eth_call` = 26 CU, `alchemy_getTokenBalances` = 20 CU), `429` handling, and current plan-level guidance: Free = `30M CU/month` and `500 CU/s`; Pay As You Go = `$0.45/1M CU` up to `300M`, then `$0.40/1M`, with `10,000 CU/s`; Enterprise = custom throughput and pricing, starting around `1000` requests/sec on the pricing page.
**Notes**: Pricing and limits can change; the official pricing page is the source of truth.

## Task 2.6 — WebSocket subscriptions

**Status**: PASS
**Approach**: Used the local WebSocket subscription reference to outline a Node.js implementation for pending transactions.
**Command(s) used**: None.
**Output quality**: Complete. Included the correct WebSocket URL pattern, `eth_subscribe` request shape, subscription types (`newHeads`, `logs`, `newPendingTransactions`, `alchemy_minedTransactions`), and reconnect/de-duplication guidance.
**Notes**: `newPendingTransactions` is high volume, so practical consumers should filter aggressively or downsample.

## Task 2.7 — Solana DAS query

**Status**: PASS
**Approach**: Queried the provided address with the CLI's `solana das` wrapper and inspected the returned `items` plus `total`.
**Command(s) used**: `alchemy --json --no-interactive solana das getAssetsByOwner '{"ownerAddress":"GsbwXfJraMomNxBcjYLcG3mxkBUiyWXAB32fGbSQQRre","page":1,"limit":5}'`
**Output quality**: Correct. The request succeeded and returned `0` assets.
**Notes**: The empty result is still a valid DAS response and not a failure.

## Task 2.8 — Multi-step workflow

**Status**: PARTIAL
**Approach**: Combined live ETH balance, heavily paginated portfolio-token data, and descending transfer history to build a portfolio summary.
**Command(s) used**:

- `alchemy --json --no-interactive balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
- `alchemy --json --no-interactive prices symbol ETH`
- `alchemy --json --no-interactive portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}]}'`
- `alchemy --json --no-interactive portfolio tokens --body '{"addresses":[{"address":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","networks":["eth-mainnet"]}],"pageKey":"..."}'` (repeated across many pages)
- `alchemy --json --no-interactive rpc alchemy_getAssetTransfers '{"fromBlock":"0x0","toBlock":"latest","fromAddress":"0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","category":["external","erc20"],"withMetadata":true,"maxCount":"0x3","order":"desc"}'`

**Output quality**: A usable summary was produced, including `32.147257835992150606` ETH (about `$67.4k` at run time) and recent February 2026 transfers. The weakness is the "top 5 ERC-20 by USD" portion: this wallet holds thousands of spam or airdrop tokens, the portfolio endpoint returns only 100 results per page with no server-side sort by value, and even `60` pages / `6000` tokens did not exhaust the dataset. The highest-valued returned assets were mostly spam-like tokens before `USDC`.
**Notes**: A production-quality portfolio view would need spam filtering, allowlists, or a bounded token universe instead of naive global sorting.

## Task 2.9 — Pagination

**Status**: PASS
**Approach**: Fetched three sequential pages of token balances with `--page-key` and summed the results.
**Command(s) used**:

- `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata`
- `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata --page-key "0x01f3106b674341ff1f18af133e7c844fc43aacdf"`
- `alchemy --json --no-interactive tokens balances 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --metadata --page-key "0x052493c962b90f081edaa3ba4b2291fc755b52bf"`

**Output quality**: Correct. Page counts were `80`, `86`, and `81`, for `247` total tokens across three pages, and the third response still returned another `pageKey`.
**Notes**: Pagination works reliably through the CLI's `--page-key` flag.

## Task 2.10 — Error recovery

**Status**: PASS
**Approach**: Intentionally used the wrong network slug, captured the structured CLI error, then retried with the correct slug.
**Command(s) used**:

- `alchemy --json --no-interactive -n ethereum-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`
- `alchemy --json --no-interactive -n eth-mainnet balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

**Output quality**: Excellent. The failure returned a clear `INVALID_ARGS` error with the message `Unknown network 'ethereum-mainnet'. Run 'alchemy network list' to see available networks.` Recovery with `eth-mainnet` worked immediately.
**Notes**: The CLI's error messages are actionable and machine-friendly.

---

## Summary

**Total**: 27/27
**Pass**: 25
**Partial**: 2
**Fail**: 0

### Key gaps found

- `alchemy tokens allowance` is broken in CLI `0.4.0`; a raw RPC workaround is required for Task 1.5.
- Building a trustworthy "top 5 ERC-20 by USD" for this wallet is expensive and misleading without spam filtering because the wallet holds thousands of low-quality tokens and the portfolio endpoint does not sort by value.
- The `transfers` wrapper defaults to ascending order, so raw RPC is the better path when a task explicitly asks for the most recent transfers.

### Key strengths

- `alchemy --json --no-interactive agent-prompt` provides a comprehensive machine-readable command contract.
- Success and failure output are both structured JSON, which makes automation straightforward.
- Raw RPC mode cleanly covers wrapper gaps without leaving the CLI.
- Solana RPC and Solana DAS work from the same CLI surface.
- The CLI's errors are clear, typed, and actionable.
- `tokens balances --metadata` gives formatted balances and easy pagination out of the box.
- The local reference bundle plus official docs were enough to answer webhook, WebSocket, and rate-limit questions confidently.
