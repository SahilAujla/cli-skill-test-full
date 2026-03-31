# CLI Skill Test Plan

Run every task below in order. For each task, record the result in `results/results.md` using the template at the bottom.

Use address `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` (vitalik.eth) for all address-based queries unless stated otherwise.

Use `eth-mainnet` as the default network unless stated otherwise.

---

## Part 1: CLI-Supported Tasks

These tasks have direct CLI commands. Test whether the agent can complete them.

### 1.1 — Get ETH balance
Get the ETH balance for the address above.

### 1.2 — Get ETH balance on a different network
Get the ETH balance for the same address on `base-mainnet`.

### 1.3 — Get ERC-20 token balances
Get ERC-20 token balances for the address. Show human-readable amounts with token names.

### 1.4 — Get token metadata
Get the metadata (name, symbol, decimals) for USDC: `0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eB48`

### 1.5 — Get token allowance
Check the USDC allowance from the address to Uniswap router: `0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B`

### 1.6 — Get spot prices
Get the current price of ETH and USDC in USD.

### 1.7 — Get historical prices
Get the historical price of ETH from 2025-01-01 to 2025-01-02.

### 1.8 — List NFTs
List 3 NFTs owned by the address.

### 1.9 — Get NFT metadata
Get metadata for NFT contract `0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D` (BAYC), token ID `1`.

### 1.10 — Get transfer history
Get the last 5 ERC-20 transfers sent FROM the address.

### 1.11 — Get latest block
Get the latest block details.

### 1.12 — Get gas prices
Get current gas prices on eth-mainnet.

### 1.13 — Look up a transaction
Get details for this transaction: `0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060`

### 1.14 — Raw JSON-RPC call
Get the current block number using a raw JSON-RPC call.

### 1.15 — Simulate a transaction
Simulate a transfer of 1 ETH from the address to `0x0000000000000000000000000000000000000000` and show the asset changes.

### 1.16 — List available networks
List all available networks, then filter to show only Solana networks.

### 1.17 — Solana RPC
Get the current Solana slot number using Solana RPC.

---

## Part 2: Tasks NOT Directly Supported by CLI

These test whether the agent can handle tasks that need deeper knowledge beyond CLI commands.

### 2.1 — Explain webhook setup
Explain how to set up an address activity webhook for the address, including what payload format to expect and how to verify the webhook signature.

### 2.2 — Parse a token balance response
Given this raw response, explain what the balance actually is in human-readable terms:
```json
{
  "contractAddress": "0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
  "tokenBalance": "0x0000000000000000000000000000000000000000000000000000000005f5e100"
}
```

### 2.3 — Recommend an ecosystem library
The user wants to build a React dApp that connects a wallet, reads token balances, and sends transactions. Recommend the right libraries and show how to configure them with Alchemy.

### 2.4 — Account Kit / Smart Wallets
Explain how to integrate Alchemy's Account Kit for embedded wallet onboarding with gas sponsorship.

### 2.5 — Rate limits and compute units
Explain Alchemy's rate limiting model. How are compute units calculated? What are the limits per plan?

### 2.6 — WebSocket subscriptions
Show how to subscribe to pending transactions in real-time using WebSocket subscriptions via Alchemy.

### 2.7 — Solana DAS query
Get all NFTs owned by a Solana address using the DAS API. Use address: `GsbwXfJraMomNxBcjYLcG3mxkBUiyWXAB32fGbSQQRre`

### 2.8 — Multi-step workflow
Build a complete portfolio view: get the ETH balance, top 5 ERC-20 tokens with names and USD values, and the 3 most recent transfers for the address. Present it as a summary.

### 2.9 — Pagination
Get ALL ERC-20 token balances for the address (not just the first page). Handle pagination until there are no more results, or until you've fetched at least 3 pages.

### 2.10 — Error recovery
Intentionally use a wrong network slug (e.g. `ethereum-mainnet` instead of `eth-mainnet`) and demonstrate how you recover from the error.

---

## Recording Results

Save results to `results/results.md` using this format for each task:

```markdown
## Task [number] — [name]

**Status**: PASS / PARTIAL / FAIL
**Approach**: [1-2 sentences on what the agent did]
**Command(s) used**: [exact commands run, if any]
**Output quality**: [was the answer correct, complete, well-formatted?]
**Notes**: [anything notable — gaps, wrong assumptions, hallucinations, etc.]
```

After all tasks, add a summary section:

```markdown
## Summary

**Total**: X/27
**Pass**: X
**Partial**: X
**Fail**: X

### Key gaps found
- [list any tasks where the agent struggled or failed]

### Key strengths
- [list any tasks where the agent performed well]
```
