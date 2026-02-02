# Hedera Development Resources

## Official Documentation
- [Hedera Docs](https://docs.hedera.com/) - Main documentation
- [Hedera SDK Reference](https://docs.hedera.com/hedera/sdks-and-apis/sdks) - SDK guides
- [Hedera API Reference](https://docs.hedera.com/hedera/sdks-and-apis/hedera-api) - Protocol docs
- [HIPs (Hedera Improvement Proposals)](https://hips.hedera.com/) - Standards and proposals

## SDKs
- [@hashgraph/sdk (JS/TS)](https://github.com/hashgraph/hedera-sdk-js) - Official JavaScript SDK
- [Hedera Java SDK](https://github.com/hashgraph/hedera-sdk-java) - Official Java SDK
- [Hedera Go SDK](https://github.com/hashgraph/hedera-sdk-go) - Official Go SDK
- [Hedera Swift SDK](https://github.com/hashgraph/hedera-sdk-swift) - Official Swift SDK

## Wallet Integration
- [HashConnect](https://github.com/Hashpack/hashconnect) - Wallet connection library
- [@hashconnect/react](https://www.npmjs.com/package/@hashconnect/react) - React integration
- [HashPack](https://www.hashpack.app/) - Popular Hedera wallet
- [Blade Wallet](https://bladewallet.io/) - Mobile-first wallet
- [Kabila Wallet](https://kabila.app/) - Web3 wallet with NFT focus
- [Ledger](https://www.ledger.com/) - Hardware wallet (HBAR, EVM, HTS assets, staking, swap)
- [Fireblocks](https://www.fireblocks.com/) - Institutional custody (HBAR, EVM)
- [Copper](https://copper.co/) - Institutional custody (HBAR, EVM, HTS assets)

## Wallet Infrastructure (Programmatic)
- [Coinbase CDP](https://docs.cdp.coinbase.com/) - Server-side wallet creation & management
- [Fireblocks](https://www.fireblocks.com/) - MPC wallet infrastructure (enterprise)
- [Dfns](https://www.dfns.co/) - Wallet-as-a-service API
- [Turnkey](https://www.turnkey.com/) - Non-custodial wallet infra
- [Privy](https://www.privy.io/) - Embedded wallets for apps
- [Magic](https://magic.link/) - Passwordless auth + wallets

## Smart Contracts
- [Hedera Smart Contracts](https://github.com/hashgraph/hedera-smart-contracts) - Precompile interfaces
- [HTS Precompiles](https://docs.hedera.com/hedera/core-concepts/smart-contracts/supported-erc-token-standards) - Token precompiles
- [JSON-RPC Relay](https://github.com/hashgraph/hedera-json-rpc-relay) - EVM relay source

## Tools & Infrastructure
- [HashScan](https://hashscan.io/) - Block explorer
- [Hedera Mirror Node](https://docs.hedera.com/hedera/core-concepts/mirror-nodes) - Query service
- [Hedera Local Node](https://github.com/hashgraph/hedera-local-node) - Local development

## Testing
- [Hedera Testnet](https://portal.hedera.com/) - Get testnet HBAR
- [Hedera Previewnet](https://portal.hedera.com/) - Preview new features
- [Local Node Docker](https://github.com/hashgraph/hedera-local-node) - Run locally

## NFT Standards
- [HIP-412](https://hips.hedera.com/hip/hip-412) - NFT Token Metadata
- [HIP-766](https://hips.hedera.com/hip/hip-766) - NFT Collection Metadata

## DeFi & DEX
- [SaucerSwap](https://www.saucerswap.finance/) - Leading DEX (AMM)
- [HeliSwap](https://www.heliswap.io/) - DEX with limit orders
- [Bonzo Finance](https://www.bonzo.finance/) - Lending protocol
- [Stader Labs](https://www.staderlabs.com/) - Liquid staking (HBARX)

## Oracles & Data
- [Chainlink](https://chain.link/) - Price feeds and CCIP (Council member)
- [Pyth Network](https://pyth.network/) - High-frequency price data
- [Supra Oracles](https://supra.com/) - Cross-chain oracles

## Learning Resources
- [Hedera Learning Center](https://hedera.com/learning-center)
- [Hedera YouTube](https://www.youtube.com/c/Hedera)
- [Hedera Blog](https://hedera.com/blog)

## Community
- [Hedera Discord](https://hedera.com/discord)
- [Hedera Twitter](https://twitter.com/hedera)
- [Hedera Forum](https://hedera.com/forum)
- [r/hedera](https://reddit.com/r/hedera)

## AI Agent Frameworks
- [OpenClaw](https://openclaw.ai) - Open-source AI agent framework with Hedera integration
- [OpenClaw GitHub](https://github.com/openclaw/openclaw) - Source code
- [OpenClaw Docs](https://docs.openclaw.ai) - Documentation
- [OpenClaw Discord](https://discord.com/invite/clawd) - Community

## Hedera Council (Governance)
Current council members include (as of Feb 2026):
- **Tech**: Google, IBM, Dell Technologies, LG Electronics, Tata Communications, ServiceNow
- **Finance**: Standard Bank, Nomura Holdings, eftpos
- **Web3**: Chainlink Labs, abrdn, Worldpay
- **Energy**: EDF, Repsol
- **Telecom**: Deutsche Telekom, Swirlds Labs
- **Other**: Ubisoft, Dentons, LSE, IIT Madras, Magalu

Full list: [hederacouncil.org](https://hederacouncil.org)

## Ecosystem
- [Hedera Ecosystem](https://hedera.com/ecosystem) - dApps and projects
- [Hashgraph Association](https://hashgraph-association.com/) - Swiss non-profit
- [HBAR Foundation](https://hbarfoundation.org/) - Grants and funding
- [Hedera Foundation](https://hedera.foundation/) - Network development

## API Services & Infrastructure
- [Arkhia](https://www.arkhia.io/) - Infrastructure provider (JSON-RPC, Mirror Node)
- [QuickNode](https://www.quicknode.com/chains/hedera) - RPC & Mirror Node (enterprise-grade)
- [INFStones](https://infstones.com/) - Multi-chain node infrastructure
- [Validation Cloud](https://www.validationcloud.io/) - Node infrastructure
- [Hashio](https://hashio.io/) - Free public JSON-RPC relay
- [Dragon Glass](https://dragonglass.me/) - Analytics & explorer
- [Ledger Works](https://lworks.io/) - Security monitoring
- [Kabuto](https://kabuto.sh/) - GraphQL API for Hedera

## Code Examples
- [Hedera Examples](https://github.com/hashgraph/hedera-sdk-js/tree/main/examples)
- [Hedera Cookbook](https://docs.hedera.com/hedera/getting-started/try-examples)

## Chain IDs
| Network | Chain ID | Currency |
|---------|----------|----------|
| Mainnet | 295 | HBAR |
| Testnet | 296 | HBAR |
| Previewnet | 297 | HBAR |

## Account ID Format
- Format: `shard.realm.num` (e.g., `0.0.123456`)
- Mainnet shard: 0
- Realm: 0 (currently only one realm)
- Num: Account number
