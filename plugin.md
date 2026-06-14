---
title: "Gitbank Plugin"
version: "1.0.0"
description: "Manage your Web3 vault on Base via GitHub identity. No wallet required. All transactions executed by the Gitbank relayer -- Gitbank pays all gas."
api_base: "https://gitbank.io/api/public"
mcp_server: "https://gitbank.io/api/mcp"
chain: "Base Mainnet (chainId 8453)"
---

# Gitbank Plugin

Gitbank is an IssueOps platform for Web3 teams. Every GitHub account gets a **soul-bound vault on Base Mainnet**, anchored to the account's permanent GitHub user ID.

**This plugin uses relayer mode.** No wallet connection required. All transactions are signed and submitted by the Gitbank relayer. Gitbank pays all gas fees. The user only needs to confirm their identity by posting one comment on GitHub.

**Supported tokens:** USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`), WETH (`0x4200000000000000000000000000000000000006`)

**Chain:** Base Mainnet (`8453`)

**API base URL:** `https://gitbank.io/api/public`

---

## How It Works

```
User: "Swap 50 USDC to WETH, my GitHub is alice"

1. AI calls GET /vault/by-github/alice
   -> vault_address, USDC: 250.00, WETH: 0.00

2. AI calls GET /prepare/swap?username=alice&amount=50&from_token=USDC&to_token=WETH&mode=relayer
   -> { confirm_code: "mcp1a2b3c4d", confirm_url: "...", instructions: "..." }

3. AI shows the user the instructions field verbatim:
   "Swap 50 USDC to WETH queued.
    To authorize, open: https://github.com/gitbankio/playground/discussions/4
    Post this comment: @gitbankbot confirm mcp1a2b3c4d
    (Expires in 10 minutes. Only @alice can confirm.)"

4. User opens the GitHub link and posts the comment as @alice.

5. Gitbank bot verifies @alice posted (via HMAC-signed webhook).
   Identity check: if commenter is NOT @alice, bot rejects.

6. Bot executes the swap via relayer. Gitbank pays gas.
   Bot posts in the GitHub thread:
   "Swap confirmed. Tx: https://basescan.org/tx/0x..."

7. Done. No wallet. No gas.
```

**Identity guarantee:** GitHub webhook payloads are HMAC-signed by GitHub. The bot reads sender identity from the signed payload only. It cannot be spoofed.

---

## Read Endpoints

### `GET /api/public/vault/by-github/:github_username`

Returns vault address and current USDC + WETH balances. **Always call this first.**

```
GET https://gitbank.io/api/public/vault/by-github/alice
```

Response (vault deployed):
```json
{
  "github_username": "alice",
  "vault_address": "0x...",
  "vault_deployed": true,
  "balances": {
    "USDC": "250.00",
    "WETH": "0.050000"
  },
  "chain": "base",
  "chain_id": 8453
}
```

Response (vault not deployed):
```json
{
  "github_username": "alice",
  "vault_deployed": false,
  "balances": {}
}
```

If `vault_deployed` is `false`, proceed normally. Vault auto-deploys on first prepare request -- free, Gitbank pays deployment gas.

### `GET /api/public/vault/:vault_address`

Returns balances for a known vault address.

### `GET /api/public/transactions/:github_username`

Returns the 20 most recent vault transactions.

```
GET https://gitbank.io/api/public/transactions/alice
```

---

## Prepare Endpoints (Relayer Mode)

All prepare endpoints queue a pending vault operation and return a confirm code. The operation executes only after the correct GitHub account confirms it. **Always use `mode=relayer`.**

**All prepare endpoints return the same shape:**
```json
{
  "ok": true,
  "command": "swap",
  "username": "alice",
  "confirm_code": "mcp1a2b3c4d",
  "instructions": "Swap 50 USDC to WETH in @alice's vault queued.\n\nTo authorize, open:\nhttps://github.com/gitbankio/playground/discussions/4\n\nPost this comment:\n@gitbankbot confirm mcp1a2b3c4d\n\n(Expires in 10 minutes. Only @alice can confirm it.)",
  "confirm_url": "https://github.com/gitbankio/playground/discussions/4",
  "expires_in_seconds": 600
}
```

**Always show the `instructions` field verbatim to the user.**

---

### `GET /api/public/prepare/deposit`

```
GET https://gitbank.io/api/public/prepare/deposit?username=alice&amount=50&token=USDC&mode=relayer
```

| Param | Required | Description |
|-------|----------|-------------|
| `username` | yes | GitHub username |
| `amount` | yes | Human-decimal amount (e.g. `50` for 50 USDC, `0.001` for 0.001 WETH) |
| `token` | yes | `USDC` or `WETH` |
| `mode` | yes | Always `relayer` |

---

### `GET /api/public/prepare/withdraw`

```
GET https://gitbank.io/api/public/prepare/withdraw?username=alice&amount=50&token=USDC&to=0x1234...&mode=relayer
```

| Param | Required | Description |
|-------|----------|-------------|
| `username` | yes | GitHub username |
| `amount` | yes | Human-decimal amount |
| `token` | yes | `USDC` or `WETH` |
| `to` | yes | Destination wallet address (EVM) |
| `mode` | yes | Always `relayer` |

A 0.1% protocol fee applies.

---

### `GET /api/public/prepare/swap`

```
GET https://gitbank.io/api/public/prepare/swap?username=alice&amount=50&from_token=USDC&to_token=WETH&mode=relayer
```

| Param | Required | Description |
|-------|----------|-------------|
| `username` | yes | GitHub username |
| `amount` | yes | Human-decimal amount of `from_token` |
| `from_token` | yes | `USDC` or `WETH` |
| `to_token` | yes | `USDC` or `WETH` (must differ) |
| `mode` | yes | Always `relayer` |

A 0.3% protocol fee applies.

---

### `GET /api/public/prepare/transfer`

Send USDC or WETH from one Gitbank vault to another (GitHub-to-GitHub transfer).

```
GET https://gitbank.io/api/public/prepare/transfer?username=alice&to_username=bob&amount=50&token=USDC&mode=relayer
```

| Param | Required | Description |
|-------|----------|-------------|
| `username` | yes | Sender GitHub username |
| `to_username` | yes | Recipient GitHub username |
| `amount` | yes | Human-decimal amount |
| `token` | yes | `USDC` or `WETH` |
| `mode` | yes | Always `relayer` |

Uses 2-step commit-reveal to prevent front-running. Relayer handles both steps.

---

## Operation Summary

| Operation | Endpoint | Gas paid by | Wallet needed |
|-----------|----------|-------------|---------------|
| Check balance | `GET /vault/by-github/:username` | n/a | No |
| Deposit USDC/WETH | `GET /prepare/deposit?...&mode=relayer` | Gitbank relayer | No |
| Withdraw USDC/WETH | `GET /prepare/withdraw?...&mode=relayer` | Gitbank relayer | No |
| Swap USDC <-> WETH | `GET /prepare/swap?...&mode=relayer` | Gitbank relayer | No |
| Transfer to another user | `GET /prepare/transfer?...&mode=relayer` | Gitbank relayer | No |
| View transactions | `GET /transactions/:username` | n/a | No |

---

## Orchestration Pattern

```
1. GET /vault/by-github/:username
   -> note vault_address and balances

2. GET /prepare/<operation>?username=<username>&...&mode=relayer
   -> confirm_code, instructions, confirm_url

3. Show the user the instructions field verbatim.
   Remind them: only their GitHub account can confirm this.

4. User opens confirm_url and posts the comment on GitHub.

5. Gitbank bot verifies identity. If commenter is wrong GitHub user, bot rejects.

6. Bot executes via relayer, posts Basescan tx link in the GitHub thread.

7. Tell the user: "Transaction confirmed. Check the GitHub thread for the Basescan link."
```

---

## Example Sessions

**Check vault balance**

```
What's in my Gitbank vault? GitHub: alice
```

1. `GET /vault/by-github/alice` -> show USDC and WETH balances.

---

**Deposit 100 USDC**

```
Deposit 100 USDC into my Gitbank vault. GitHub: alice.
```

1. `GET /vault/by-github/alice` -> vault_address.
2. `GET /prepare/deposit?username=alice&amount=100&token=USDC&mode=relayer` -> instructions.
3. Show instructions. User confirms on GitHub. Relayer executes. Bot posts Basescan link.

---

**Swap 50 USDC to WETH**

```
Swap 50 USDC to WETH. GitHub: alice.
```

1. `GET /vault/by-github/alice` -> confirm USDC balance >= 50.
2. `GET /prepare/swap?username=alice&amount=50&from_token=USDC&to_token=WETH&mode=relayer`.
3. Show instructions. User confirms on GitHub. Relayer executes Uniswap v3 swap.

---

**Withdraw 50 USDC to wallet**

```
Withdraw 50 USDC from my vault to 0x1234... GitHub: alice.
```

1. `GET /vault/by-github/alice` -> confirm USDC balance >= 50.
2. `GET /prepare/withdraw?username=alice&amount=50&token=USDC&to=0x1234...&mode=relayer`.
3. Show instructions. User confirms. Relayer executes. Done.

---

**Send 10 USDC to another user**

```
Send 10 USDC from my vault to @bob. GitHub: alice.
```

1. `GET /vault/by-github/alice` -> confirm USDC >= 10.
2. `GET /vault/by-github/bob` -> confirm bob has a vault.
3. `GET /prepare/transfer?username=alice&to_username=bob&amount=10&token=USDC&mode=relayer`.
4. Alice confirms on GitHub. Relayer executes 2-step transfer. Bob receives 10 USDC.

---

## Security Model

- **GitHub identity is mandatory.** Confirm code is bound to a specific GitHub username. Only that account can authorize the operation.
- **No execution before identity check.** Prepare endpoint returns only a confirm code. Nothing is signed or executed until the bot verifies via HMAC-signed GitHub webhook.
- **Relayer is the signer.** Gitbank relayer holds the execution keypair server-side (AES-256-GCM encrypted). User never holds a private key.
- **Destination locked at prepare time.** For withdrawals and transfers, destination is embedded in the prepared transaction and cannot be changed after the confirm code is generated.
- **Soul-bound vaults.** gitTokens are non-transferable ERC-20. Cannot be phished or drained via wallet approvals.

---

## Error Handling

| HTTP | `error` field | Action |
|------|--------------|--------|
| 400 | `"username and amount are required"` | Re-prompt user for missing info |
| 400 | `"Invalid destination address"` | Ask for a valid EVM address |
| 400 | `"Unsupported token. Use USDC or WETH"` | Clarify token symbol |
| 400 | `"Insufficient balance"` | Show current balance, ask for lower amount |
| 404 | `"User not found"` | User needs to sign up at gitbank.io first |
| 429 | `"Rate limit exceeded"` | Wait 1 hour, inform user |

---

## Notes

- GitHub username lookup is case-insensitive.
- Vaults auto-deploy on first prepare request. No prior setup required.
- Confirm codes expire in 10 minutes. If expired, call prepare again for a fresh code.
- GitVaultFactory on Base Mainnet: `0xAA0a4ff46733EBaE8E658642A1314f18980fc77B`
- For MCP-capable clients (Claude Desktop, Cursor, Windsurf), use `https://gitbank.io/api/mcp` for richer tool support.
