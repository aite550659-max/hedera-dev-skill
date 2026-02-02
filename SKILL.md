---
name: hedera-dev
description: End-to-end Hedera development playbook (Feb 2026). Covers Hedera SDK (@hashgraph/sdk), Hedera Token Service (HTS), Hedera Consensus Service (HCS), smart contracts (Solidity via EVM), HashConnect wallet integration, mirror node queries, and JSON-RPC relay for EVM compatibility. Hedera uses hashgraph consensus (not blockchain) with aBFT security, <3 sec finality, and fixed USD fees starting at $0.0001.
user-invocable: true
---

# Hedera Development Skill

## About Hedera (context for development)
Hedera is a public DLT network using **hashgraph consensus** (not blockchain):
- **aBFT security**: Asynchronous Byzantine Fault Tolerant (strongest form)
- **Finality <3 seconds**: Transactions cannot be reversed once finalized
- **Fixed USD fees**: Starting at $0.0001 per transaction
- **EVM compatible**: Supports Solidity, works with Ethereum tools
- **Carbon negative**: More efficient than Visa transactions
- **Hedera Council**: Governed by 30+ leading enterprises (Google, IBM, Dell, LG, Deutsche Telekom, ServiceNow, Chainlink Labs, etc.)

Key insight: *"Hashgraph uses gossip protocol + virtual voting for consensus. No miners, no wasted work, 100% efficient."*

## What this Skill is for
Use this Skill when the user asks for:
- Hedera dApp UI work (React / Next.js)
- Wallet connection with HashConnect or WalletConnect
- HBAR transfers and account management
- Hedera Token Service (HTS) - fungible and NFT tokens
- Hedera Consensus Service (HCS) - timestamped, ordered messages
- **AI trust and attestation** - Audit trails for AI accountability
- **Autonomous agent logging** - Verifiable AI decision records (see OpenClaw/moltbot integration)
- Smart contract deployment and interaction (Solidity)
- Mirror node queries and indexing
- Hedera JSON-RPC Relay for EVM tooling compatibility

### OpenClaw (Moltbot) Integration
This skill includes patterns for integrating [OpenClaw](https://openclaw.ai) AI agents with Hedera:
- **Attestation of agent actions** to HCS for "Trust but Verify"
- **Immutable audit trails** of AI decisions and transactions
- **Hash-chained logs** for tamper-evident accountability
- See [ai-trust-attestation.md](ai-trust-attestation.md) for implementation details

## Default stack decisions (opinionated)

1) **SDK: @hashgraph/sdk first**
   - Use official Hedera SDK for all native Hedera services
   - TypeScript/JavaScript: `@hashgraph/sdk`
   - Supports: accounts, tokens, consensus, files, smart contracts, scheduled transactions

2) **Wallet: HashConnect for Hedera-native**
   - HashConnect for HashPack and other Hedera wallets
   - WalletConnect for broader ecosystem support
   - Always support both when building consumer dApps

3) **Smart Contracts: Solidity via Hedera EVM**
   - Deploy Solidity contracts using ContractCreateTransaction
   - Or use JSON-RPC Relay with standard EVM tools (Hardhat, Foundry)
   - Precompiles available for HTS integration from Solidity

4) **Tokens: HTS over custom contracts**
   - Prefer Hedera Token Service for fungible/NFT tokens
   - Native performance, lower fees, built-in compliance features
   - Use HTS precompiles when interacting from smart contracts

5) **Data: Mirror nodes for queries**
   - Use mirror nodes for historical data, account balances, transaction history
   - REST API: `https://mainnet.mirrornode.hedera.com/api/v1/`
   - Never query consensus nodes for historical data

6) **Testing**
   - Testnet: Use Hedera testnet (free HBAR from faucet)
   - Local: Hedera Local Node for isolated development
   - Previewnet: For testing new network features

## Operating procedure (how to execute tasks)

### 1. Classify the task layer
- UI/wallet layer (HashConnect, React hooks)
- SDK/scripts layer (transactions, queries)
- Token layer (HTS fungible, NFT, custom fees)
- Consensus layer (HCS topics, messages)
- Smart contract layer (Solidity, precompiles)
- Data layer (mirror node REST, GraphQL)

### 2. Pick the right building blocks
- Native Hedera operations: @hashgraph/sdk
- EVM tooling compatibility: JSON-RPC Relay + ethers.js/viem
- Wallet UX: HashConnect (@hashconnect/react)
- Data queries: Mirror node REST API
- Real-time data: Mirror node gRPC/websocket

### 3. Implement with Hedera-specific correctness
Always be explicit about:
- Network (mainnet/testnet/previewnet/local)
- Operator account ID and private key handling
- Transaction fees and max transaction fee settings
- Account auto-association slots for tokens
- Key structures (single key, threshold, key list)
- Memo fields for compliance/tracking

### 4. Add tests
- Unit tests: Mock SDK client responses
- Integration tests: Hedera testnet or local node
- For wallet UX: Test with HashPack on testnet

### 5. Deliverables expectations
When you implement changes, provide:
- Exact files changed + diffs
- Commands to install/build/test
- Network configuration (testnet vs mainnet)
- Risk notes for anything touching keys/fees/token transfers

## Progressive disclosure (read when needed)
- **AI Trust & Attestation: [ai-trust-attestation.md](ai-trust-attestation.md)** ⭐ (includes OpenClaw/moltbot integration)
- Wallet integration: [wallet-hashconnect.md](wallet-hashconnect.md)
- Token Service: [hts-tokens.md](hts-tokens.md)
- Consensus Service: [hcs-consensus.md](hcs-consensus.md)
- Smart Contracts: [smart-contracts.md](smart-contracts.md)
- Mirror Node: [mirror-node.md](mirror-node.md)
- JSON-RPC Relay: [json-rpc-relay.md](json-rpc-relay.md)
- Payments & Treasury: [payments-hbar.md](payments-hbar.md)
- Testing strategy: [testing.md](testing.md)
- Security checklist: [security.md](security.md)
- Reference links: [resources.md](resources.md)
