# Smart Contracts on Hedera

## Overview
Hedera supports Solidity smart contracts via its EVM implementation. Two approaches:
1. **Native SDK** - Deploy/interact via @hashgraph/sdk
2. **JSON-RPC Relay** - Use standard EVM tools (Hardhat, Foundry)

## Native SDK Approach

### Deploy Contract
```typescript
import { 
  ContractCreateTransaction,
  FileCreateTransaction,
  Client 
} from "@hashgraph/sdk";
import fs from "fs";

// 1. Upload bytecode to file
const bytecode = fs.readFileSync("MyContract.bin");
const fileCreateTx = new FileCreateTransaction()
  .setContents(bytecode)
  .setKeys([operatorKey]);

const fileResponse = await fileCreateTx.execute(client);
const fileReceipt = await fileResponse.getReceipt(client);
const bytecodeFileId = fileReceipt.fileId;

// 2. Create contract
const contractTx = new ContractCreateTransaction()
  .setBytecodeFileId(bytecodeFileId)
  .setGas(100000)
  .setConstructorParameters(
    new ContractFunctionParameters().addString("Hello")
  );

const contractResponse = await contractTx.execute(client);
const contractReceipt = await contractResponse.getReceipt(client);
const contractId = contractReceipt.contractId;
```

### Call Contract Function
```typescript
import { 
  ContractExecuteTransaction,
  ContractFunctionParameters 
} from "@hashgraph/sdk";

// State-changing call
const executeTx = new ContractExecuteTransaction()
  .setContractId(contractId)
  .setGas(100000)
  .setFunction(
    "setGreeting",
    new ContractFunctionParameters().addString("Hi!")
  );

await executeTx.execute(client);
```

### Query Contract (read-only)
```typescript
import { ContractCallQuery } from "@hashgraph/sdk";

const query = new ContractCallQuery()
  .setContractId(contractId)
  .setGas(100000)
  .setFunction("getGreeting");

const result = await query.execute(client);
const greeting = result.getString(0);
```

## JSON-RPC Relay Approach

### Hardhat Configuration
```javascript
// hardhat.config.js
module.exports = {
  solidity: "0.8.19",
  networks: {
    hedera_testnet: {
      url: "https://testnet.hashio.io/api",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 296
    },
    hedera_mainnet: {
      url: "https://mainnet.hashio.io/api",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 295
    }
  }
};
```

### Deploy with Hardhat
```bash
npx hardhat run scripts/deploy.js --network hedera_testnet
```

### Use with ethers.js
```typescript
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider("https://testnet.hashio.io/api");
const wallet = new ethers.Wallet(privateKey, provider);

const contract = new ethers.Contract(contractAddress, abi, wallet);
await contract.setGreeting("Hello from ethers!");
```

## HTS Precompiles

Access Hedera Token Service from Solidity:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity ^0.8.0;

import "@hashgraph/smart-contracts/contracts/hts-precompile/IHederaTokenService.sol";

contract TokenManager {
    address constant HTS_PRECOMPILE = address(0x167);
    
    function transferToken(
        address token,
        address from,
        address to,
        int64 amount
    ) external returns (int responseCode) {
        responseCode = IHederaTokenService(HTS_PRECOMPILE)
            .transferToken(token, from, to, amount);
    }
    
    function associateToken(
        address account,
        address token
    ) external returns (int responseCode) {
        responseCode = IHederaTokenService(HTS_PRECOMPILE)
            .associateToken(account, token);
    }
}
```

### Install Precompile Interfaces
```bash
npm install @hashgraph/smart-contracts
```

## Gas and Fees

- Gas price is fixed on Hedera (no gas auctions)
- Use `setGas()` to specify gas limit
- Unused gas is refunded
- Typical operations:
  - Contract deploy: 100,000 - 500,000 gas
  - Simple call: 25,000 - 50,000 gas
  - Complex call: 50,000 - 200,000 gas

## Best Practices

1. **Use HTS precompiles** - For token operations, precompiles are more efficient
2. **Gas estimation** - Start high, refine based on actual usage
3. **Test on testnet** - Contract deployment costs HBAR
4. **Verify contracts** - Use HashScan for contract verification
5. **Consider state rent** - Long-lived contracts have ongoing costs
6. **EVM equivalence** - Not all EVM opcodes behave identically
