# Hedera Development Skill

An [OpenClaw](https://openclaw.ai) skill for end-to-end Hedera development.

## Overview

This skill provides AI agents with comprehensive knowledge for building on Hedera, including:

- **Hedera SDK** (`@hashgraph/sdk`) - Accounts, transactions, queries
- **Hedera Token Service (HTS)** - Fungible tokens and NFTs
- **Hedera Consensus Service (HCS)** - Timestamped, ordered messages
- **Smart Contracts** - Solidity via Hedera EVM
- **HashConnect** - Wallet integration for dApps
- **Mirror Node** - Historical data queries
- **JSON-RPC Relay** - EVM tooling compatibility (Hardhat, Foundry, ethers.js)

### AI Trust & Attestation

This skill includes patterns for integrating AI agents with Hedera for accountability:

- Attestation of agent actions to HCS
- Immutable audit trails of AI decisions
- Hash-chained logs for tamper-evident accountability
- OpenClaw/moltbot integration examples

## Installation

```bash
npx openclaw skills add https://github.com/aite550659-max/hedera-dev-skill
```

Or manually clone to your OpenClaw skills directory:

```bash
cd ~/.openclaw/workspace/skills
git clone https://github.com/aite550659-max/hedera-dev-skill hedera-dev
```

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition and operating procedures |
| `ai-trust-attestation.md` | AI accountability patterns using HCS |
| `wallet-hashconnect.md` | HashConnect wallet integration |
| `hts-tokens.md` | Hedera Token Service guide |
| `hcs-consensus.md` | Hedera Consensus Service guide |
| `smart-contracts.md` | Solidity on Hedera EVM |
| `mirror-node.md` | Mirror node REST API queries |
| `json-rpc-relay.md` | EVM compatibility layer |
| `payments-hbar.md` | HBAR transfers and treasury operations |
| `testing.md` | Testing strategies (local node, testnet) |
| `security.md` | Security checklist |
| `resources.md` | Links to docs, tools, and ecosystem |

## About Hedera

Hedera is a public DLT network using **hashgraph consensus** (not blockchain):

- **aBFT security** - Asynchronous Byzantine Fault Tolerant
- **Finality <3 seconds** - Transactions cannot be reversed
- **Fixed USD fees** - Starting at $0.0001 per transaction
- **EVM compatible** - Works with Ethereum tools
- **Carbon negative** - More efficient than Visa

Governed by 30+ leading enterprises including Google, IBM, Dell, LG, Deutsche Telekom, ServiceNow, and Chainlink Labs.

## Links

- [Hedera Docs](https://docs.hedera.com/)
- [Hedera Portal](https://portal.hedera.com/) - Get testnet HBAR
- [HashScan](https://hashscan.io/) - Block explorer
- [OpenClaw](https://openclaw.ai) - AI agent framework

## License

MIT
