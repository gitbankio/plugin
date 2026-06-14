# Gitbank AI Plugin

> Manage your Web3 vault on Base via GitHub identity. No wallet required. All transactions executed by the Gitbank relayer -- Gitbank pays all gas.

## What is this?

This repository contains a single plugin file (`plugin.md`) that teaches any AI assistant how to interact with Gitbank. Load it into your favorite AI and you can:

- Check your vault balance
- Deposit USDC or WETH
- Withdraw to any wallet address
- Swap USDC to WETH (Uniswap v3)
- Transfer to another GitHub user's vault
- View transaction history

All without connecting a wallet. All without paying gas. The only thing required is a GitHub account.

## How it works

```
You: "Swap 50 USDC to WETH, my GitHub is alice"
  |
AI reads plugin.md -- knows how to call Gitbank API
  |
AI calls: GET /api/public/prepare/swap?username=alice&amount=50&...&mode=relayer
  |
AI shows you: "Open this GitHub link and post: @gitbankbot confirm mcp1a2b3c4d"
  |
You post the comment on GitHub as @alice
  |
Gitbank verifies your identity (HMAC-signed GitHub webhook)
  |
Gitbank relayer signs and submits the transaction on Base -- Gitbank pays gas
  |
Done. Tx link posted in the GitHub thread.
```

**No MetaMask. No Coinbase Wallet. No seed phrase. No gas fees.**

## Download

**[Download plugin.md](https://gitbankio.github.io/plugin/)**

Or via terminal:

```bash
curl -O https://raw.githubusercontent.com/gitbankio/plugin/main/plugin.md
```

---

## Load into any AI

### ChatGPT

1. Go to [chat.openai.com](https://chat.openai.com)
2. Start a new conversation
3. Click the paperclip icon -- upload `plugin.md`
4. Type: `"Load this plugin and help me check my Gitbank vault balance. My GitHub is <your_username>"`

### Claude

1. Go to [claude.ai](https://claude.ai)
2. Start a new conversation
3. Click the paperclip icon -- upload `plugin.md`
4. Type: `"Load this plugin and help me check my Gitbank vault. GitHub: <your_username>"`

Or set it as a Project instruction in Claude Projects for persistent access.

### Gemini

1. Go to [gemini.google.com](https://gemini.google.com)
2. Start a new conversation
3. Click the attachment icon -- upload `plugin.md`
4. Type: `"Use this plugin to help me manage my Gitbank vault. GitHub: <your_username>"`

### Grok

1. Go to [grok.com](https://grok.com)
2. Start a new conversation
3. Upload `plugin.md` via the attachment icon
4. Type: `"Load this as a plugin and help me check my Gitbank vault. GitHub: <your_username>"`

### Kimi

1. Go to [kimi.ai](https://kimi.ai)
2. Start a new conversation
3. Upload `plugin.md`
4. Type: `"Use this as a plugin guide. Check my Gitbank vault. GitHub: <your_username>"`

### Venice.ai

1. Go to [venice.ai](https://venice.ai)
2. Start a new conversation
3. Click the attachment icon -- upload `plugin.md`
4. Type: `"Load this plugin and help me manage my Gitbank vault. GitHub: <your_username>"`

Venice.ai is a privacy-preserving AI -- your prompts are not stored or used for training.

### Cursor / Windsurf / VS Code

Use the MCP server directly -- no file upload needed:

```json
{
  "mcpServers": {
    "gitbank": {
      "url": "https://gitbank.io/api/mcp"
    }
  }
}
```

### ChatGPT GPT Builder (permanent GPT)

1. Go to [chat.openai.com/gpts/editor](https://chat.openai.com/gpts/editor)
2. Tab **Configure** -- paste the contents of `plugin.md` into **Instructions**
3. Tab **Actions** -- **Import from URL** -- `https://gitbank.io/api/openapi.json`
4. Save and publish

---

## Requirements

- A GitHub account
- A Gitbank account (sign up free at [gitbank.io](https://gitbank.io))
- That is all

No wallet. No crypto. No gas fees.

---

## Available Commands

Once the plugin is loaded, you can ask your AI in plain English:

| What you want | What to say |
|---|---|
| Check balance | `"What's in my Gitbank vault? GitHub: alice"` |
| Deposit | `"Deposit 100 USDC into my vault. GitHub: alice"` |
| Withdraw | `"Withdraw 50 USDC to 0x1234... GitHub: alice"` |
| Swap | `"Swap 50 USDC to WETH. GitHub: alice"` |
| Transfer | `"Send 10 USDC to @bob. GitHub: alice"` |
| History | `"Show my recent transactions. GitHub: alice"` |

---

## How Authentication Works

Gitbank uses your **permanent GitHub user ID** as your vault's identity anchor. It cannot be spoofed.

When you confirm a transaction:
1. You post one comment on GitHub: `@gitbankbot confirm <code>`
2. The Gitbank bot receives a GitHub webhook -- HMAC-signed by GitHub
3. Bot reads the commenter's identity from the signed payload
4. If it matches the account that requested the operation, it proceeds
5. If it does not match, it rejects: *"This command was requested by @alice. Only they can confirm it."*

No one can execute a transaction on your behalf -- not even the AI assistant.

---

## Why No Wallet?

Gitbank uses a **relayer pattern**:

1. When you sign up at gitbank.io, a server-side execution keypair is generated for your vault
2. The keypair is encrypted with AES-256-GCM and stored in the Gitbank database
3. When you confirm on GitHub, Gitbank decrypts the keypair, signs the meta-transaction, and submits it to Base
4. The Gitbank deployer address pays all gas costs
5. Your vault receives the result

---

## Fees

| Operation | Fee |
|-----------|-----|
| Deposit | Free |
| Withdraw | 0.1% |
| Swap | 0.3% |
| Transfer | Free |
| Gas | Free (paid by Gitbank relayer) |

---

## Contracts (Base Mainnet)

| Contract | Address |
|----------|---------|
| GitVaultFactory | [`0xAA0a4ff46733EBaE8E658642A1314f18980fc77B`](https://basescan.org/address/0xAA0a4ff46733EBaE8E658642A1314f18980fc77B#code) |
| GitVault impl | [`0x3602197A1b445AA4746c47C9D69436d9B7cF5dc9`](https://basescan.org/address/0x3602197A1b445AA4746c47C9D69436d9B7cF5dc9#code) |

---

## MCP Server

For Claude Desktop, Cursor, Windsurf, and other MCP-capable clients:

```
https://gitbank.io/api/mcp
```

---

## OpenAPI Spec

```
https://gitbank.io/api/openapi.json
```

---

## License

Apache 2.0 -- see [LICENSE](LICENSE)
