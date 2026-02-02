# Hedera JSON-RPC Relay

## Overview
The JSON-RPC Relay enables standard Ethereum tooling (Hardhat, Foundry, ethers.js, viem) to work with Hedera's EVM.

## Public Relay Endpoints

| Network | Endpoint | Chain ID |
|---------|----------|----------|
| Mainnet | `https://mainnet.hashio.io/api` | 295 |
| Testnet | `https://testnet.hashio.io/api` | 296 |
| Previewnet | `https://previewnet.hashio.io/api` | 297 |

Alternative providers:
- Arkhia: `https://pool.arkhia.io/hedera/mainnet/json-rpc/v1/`
- Validation Cloud: Check their docs for endpoint

## Using with ethers.js

```typescript
import { ethers } from "ethers";

// Connect to Hedera testnet
const provider = new ethers.JsonRpcProvider(
  "https://testnet.hashio.io/api",
  { chainId: 296, name: "hedera-testnet" }
);

// Create wallet
const wallet = new ethers.Wallet(privateKey, provider);

// Get balance (returns HBAR in wei format)
const balance = await provider.getBalance(wallet.address);
console.log(`Balance: ${ethers.formatEther(balance)} HBAR`);

// Send transaction
const tx = await wallet.sendTransaction({
  to: recipientAddress,
  value: ethers.parseEther("1.0")
});
await tx.wait();
```

## Using with viem

```typescript
import { createPublicClient, createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { hederaTestnet } from "viem/chains";

const publicClient = createPublicClient({
  chain: hederaTestnet,
  transport: http("https://testnet.hashio.io/api")
});

const account = privateKeyToAccount(privateKey);

const walletClient = createWalletClient({
  account,
  chain: hederaTestnet,
  transport: http("https://testnet.hashio.io/api")
});

// Send transaction
const hash = await walletClient.sendTransaction({
  to: recipientAddress,
  value: parseEther("1.0")
});
```

## Hardhat Configuration

```javascript
// hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    hedera_testnet: {
      url: "https://testnet.hashio.io/api",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 296,
      timeout: 120000 // Hedera needs longer timeouts
    },
    hedera_mainnet: {
      url: "https://mainnet.hashio.io/api",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 295,
      timeout: 120000
    }
  }
};
```

## Foundry Configuration

```toml
# foundry.toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]

[rpc_endpoints]
hedera_testnet = "https://testnet.hashio.io/api"
hedera_mainnet = "https://mainnet.hashio.io/api"
```

Deploy with Foundry:
```bash
forge create --rpc-url hedera_testnet \
  --private-key $PRIVATE_KEY \
  src/MyContract.sol:MyContract
```

## Account Mapping

Hedera accounts map to EVM addresses:
- Account `0.0.123456` → EVM address derived from account ID
- Use `AccountId.toSolidityAddress()` for conversion

```typescript
import { AccountId } from "@hashgraph/sdk";

const accountId = AccountId.fromString("0.0.123456");
const evmAddress = accountId.toSolidityAddress();
// Returns: 0x000000000000000000000000000000000001e240
```

## Limitations

1. **No pending transactions** - `eth_pendingTransactions` not supported
2. **Gas estimation** - May differ from actual consumption
3. **Block time** - Hedera has ~3-5 second finality
4. **Opcodes** - Some EVM opcodes behave differently
5. **Rate limits** - Public relays have rate limits
6. **Long-lived connections** - May timeout, implement reconnection

## Best Practices

1. **Use longer timeouts** - Hedera transactions take longer than L1 Ethereum
2. **Handle errors gracefully** - Relay errors differ from standard EVM
3. **Test thoroughly** - Not all EVM patterns work identically
4. **Consider native SDK** - For complex Hedera operations, use @hashgraph/sdk
5. **Private relay** - For production, consider running your own relay
