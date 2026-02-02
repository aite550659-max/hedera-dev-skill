# Testing Hedera Applications

## Overview
Hedera offers multiple testing environments. Choose based on your needs:

| Environment | Use Case | Cost | Setup |
|------------|----------|------|-------|
| **Local Node** | Unit tests, fast iteration | Free | Docker required |
| **Testnet** | Integration tests, wallet testing | Free (faucet) | Portal account |
| **Previewnet** | Test new Hedera features | Free (faucet) | Portal account |
| **Mainnet** | Production only | Real HBAR | Live network |

## Local Node Setup

### Prerequisites
```bash
# Install Docker and Docker Compose
docker --version  # 20.10+
docker-compose --version  # 1.29+
```

### Start Local Node
```bash
# Clone and start
git clone https://github.com/hashgraph/hedera-local-node.git
cd hedera-local-node
docker-compose up -d

# Default accounts available:
# 0.0.2 - Treasury (genesis)
# 0.0.1002 - Operator (with private key)
```

### Local Node Configuration
```typescript
import { Client, AccountId, PrivateKey } from "@hashgraph/sdk";

// Local node defaults
const client = Client.forNetwork({
  "127.0.0.1:50211": new AccountId(3)
});

client.setMirrorNetwork(["127.0.0.1:5600"]);

// Default operator (pre-funded)
const operatorId = AccountId.fromString("0.0.1002");
const operatorKey = PrivateKey.fromString(
  "302e020100300506032b65700422042091132178e72057a1d7528025956fe39b0b847f200ab59b2fdd367017f3087137"
);

client.setOperator(operatorId, operatorKey);
```

### Reset Local Node
```bash
docker-compose down -v  # Remove volumes
docker-compose up -d    # Fresh start
```

## Testnet Setup

### Get Testnet Account
1. Go to [portal.hedera.com](https://portal.hedera.com)
2. Create account / sign in
3. Generate testnet credentials
4. Save Account ID and Private Key

### Testnet Configuration
```typescript
import { Client, AccountId, PrivateKey } from "@hashgraph/sdk";

const client = Client.forTestnet();
client.setOperator(
  AccountId.fromString(process.env.HEDERA_TESTNET_ACCOUNT_ID!),
  PrivateKey.fromString(process.env.HEDERA_TESTNET_PRIVATE_KEY!)
);
```

### Testnet Faucet
Free HBAR from portal or:
```bash
# Via Hedera Portal API
curl -X POST https://portal.hedera.com/api/testnet/faucet \
  -H "Content-Type: application/json" \
  -d '{"accountId": "0.0.123456"}'
```

## Unit Testing Pattern

### Jest Setup
```typescript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  setupFilesAfterEnv: ['./test/setup.ts'],
  testTimeout: 30000  // Hedera operations need time
};
```

### Test Setup File
```typescript
// test/setup.ts
import { Client, AccountId, PrivateKey } from "@hashgraph/sdk";

let client: Client;

beforeAll(() => {
  // Use local node for fast tests
  client = Client.forNetwork({
    "127.0.0.1:50211": new AccountId(3)
  });
  client.setMirrorNetwork(["127.0.0.1:5600"]);
  client.setOperator(
    AccountId.fromString("0.0.1002"),
    PrivateKey.fromString("302e0201...")
  );
});

afterAll(() => {
  client.close();
});

export { client };
```

### Example Test
```typescript
// test/transfer.test.ts
import { TransferTransaction, Hbar, AccountId } from "@hashgraph/sdk";
import { client } from "./setup";

describe("HBAR Transfers", () => {
  it("should transfer HBAR between accounts", async () => {
    const sender = AccountId.fromString("0.0.1002");
    const receiver = AccountId.fromString("0.0.1003");
    
    const tx = new TransferTransaction()
      .addHbarTransfer(sender, Hbar.from(-1))
      .addHbarTransfer(receiver, Hbar.from(1));
    
    const response = await tx.execute(client);
    const receipt = await response.getReceipt(client);
    
    expect(receipt.status.toString()).toBe("SUCCESS");
  });
});
```

## Integration Testing

### Testing HTS Tokens
```typescript
describe("HTS Token Operations", () => {
  let tokenId: TokenId;
  
  beforeAll(async () => {
    // Create test token
    const tx = new TokenCreateTransaction()
      .setTokenName("Test Token")
      .setTokenSymbol("TEST")
      .setDecimals(8)
      .setInitialSupply(1000000)
      .setTreasuryAccountId(operatorId)
      .setAdminKey(operatorKey);
    
    const response = await tx.execute(client);
    const receipt = await response.getReceipt(client);
    tokenId = receipt.tokenId!;
  });
  
  it("should transfer tokens", async () => {
    // Associate recipient first
    const associateTx = new TokenAssociateTransaction()
      .setAccountId(recipientId)
      .setTokenIds([tokenId]);
    
    await associateTx.freezeWith(client).sign(recipientKey);
    await associateTx.execute(client);
    
    // Transfer
    const transferTx = new TransferTransaction()
      .addTokenTransfer(tokenId, operatorId, -100)
      .addTokenTransfer(tokenId, recipientId, 100);
    
    const response = await transferTx.execute(client);
    const receipt = await response.getReceipt(client);
    
    expect(receipt.status.toString()).toBe("SUCCESS");
  });
});
```

### Testing HCS Topics
```typescript
describe("HCS Topic Operations", () => {
  let topicId: TopicId;
  
  beforeAll(async () => {
    const tx = new TopicCreateTransaction()
      .setTopicMemo("Test Topic");
    
    const response = await tx.execute(client);
    const receipt = await response.getReceipt(client);
    topicId = receipt.topicId!;
  });
  
  it("should submit and retrieve messages", async () => {
    const message = JSON.stringify({ test: "data", timestamp: Date.now() });
    
    const submitTx = new TopicMessageSubmitTransaction()
      .setTopicId(topicId)
      .setMessage(message);
    
    const response = await submitTx.execute(client);
    const receipt = await response.getReceipt(client);
    
    expect(receipt.topicSequenceNumber).toBeDefined();
    
    // Query from mirror node (may need small delay)
    await new Promise(r => setTimeout(r, 3000));
    
    const mirrorResponse = await fetch(
      `http://127.0.0.1:5551/api/v1/topics/${topicId}/messages`
    );
    const data = await mirrorResponse.json();
    
    expect(data.messages.length).toBeGreaterThan(0);
  });
});
```

### Testing Smart Contracts
```typescript
describe("Smart Contract Operations", () => {
  let contractId: ContractId;
  
  beforeAll(async () => {
    // Deploy contract (bytecode from Solidity compilation)
    const fileTx = new FileCreateTransaction()
      .setContents(contractBytecode)
      .setKeys([operatorKey]);
    
    const fileResponse = await fileTx.execute(client);
    const fileReceipt = await fileResponse.getReceipt(client);
    const bytecodeFileId = fileReceipt.fileId!;
    
    const contractTx = new ContractCreateTransaction()
      .setBytecodeFileId(bytecodeFileId)
      .setGas(100000);
    
    const contractResponse = await contractTx.execute(client);
    const contractReceipt = await contractResponse.getReceipt(client);
    contractId = contractReceipt.contractId!;
  });
  
  it("should call contract function", async () => {
    const executeTx = new ContractExecuteTransaction()
      .setContractId(contractId)
      .setGas(50000)
      .setFunction("setValue", new ContractFunctionParameters().addUint256(42));
    
    const response = await executeTx.execute(client);
    const receipt = await response.getReceipt(client);
    
    expect(receipt.status.toString()).toBe("SUCCESS");
    
    // Query the value
    const query = new ContractCallQuery()
      .setContractId(contractId)
      .setGas(50000)
      .setFunction("getValue");
    
    const result = await query.execute(client);
    expect(result.getUint256(0).toNumber()).toBe(42);
  });
});
```

## Testing with Hardhat (EVM)

### Hardhat Test Config
```javascript
// hardhat.config.js
module.exports = {
  solidity: "0.8.19",
  networks: {
    hedera_local: {
      url: "http://127.0.0.1:7546",
      accounts: ["0x..."],  // Local node key
      chainId: 298
    },
    hedera_testnet: {
      url: "https://testnet.hashio.io/api",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 296,
      timeout: 120000
    }
  }
};
```

### Hardhat Test
```javascript
const { expect } = require("chai");

describe("MyContract", function () {
  it("should deploy and interact", async function () {
    const MyContract = await ethers.getContractFactory("MyContract");
    const contract = await MyContract.deploy("Hello");
    await contract.waitForDeployment();
    
    expect(await contract.greeting()).to.equal("Hello");
    
    await contract.setGreeting("Hi");
    expect(await contract.greeting()).to.equal("Hi");
  });
});
```

## CI/CD Integration

### GitHub Actions Example
```yaml
name: Hedera Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      hedera:
        image: hashgraph/hedera-local-node:latest
        ports:
          - 50211:50211
          - 5600:5600
          - 5551:5551
          - 7546:7546
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Wait for Hedera Local Node
        run: |
          timeout 120 bash -c 'until curl -s http://localhost:5551/api/v1/accounts/0.0.2 > /dev/null; do sleep 2; done'
      
      - name: Run tests
        run: npm test
```

## Best Practices

### 1. Use Local Node for Fast Tests
- Instant feedback loop
- No rate limits
- Deterministic state

### 2. Test on Testnet Before Mainnet
- Real network conditions
- Wallet integration testing
- Mirror node behavior

### 3. Handle Async Properly
```typescript
// Always await receipts for confirmation
const response = await tx.execute(client);
const receipt = await response.getReceipt(client);

// Allow time for mirror node sync
await new Promise(r => setTimeout(r, 3000));
```

### 4. Clean Up Test Artifacts
```typescript
afterAll(async () => {
  // Delete test tokens, topics, etc. if admin keys set
  if (tokenId && adminKey) {
    const deleteTx = new TokenDeleteTransaction()
      .setTokenId(tokenId);
    await deleteTx.freezeWith(client).sign(adminKey);
    await deleteTx.execute(client);
  }
});
```

### 5. Mock External Services
For unit tests, mock mirror node responses:
```typescript
jest.mock('node-fetch', () => jest.fn());

// In test
(fetch as jest.Mock).mockResolvedValue({
  json: () => Promise.resolve({ messages: [] })
});
```

## Debugging Tips

### Enable SDK Logging
```typescript
import { Logger, LogLevel } from "@hashgraph/sdk";

const logger = new Logger(LogLevel.Debug);
client.setLogger(logger);
```

### Check Transaction Status
```typescript
try {
  const receipt = await response.getReceipt(client);
  console.log("Status:", receipt.status.toString());
} catch (error) {
  console.error("Transaction failed:", error.message);
  // Check error.status for specific failure reason
}
```

### Inspect Mirror Node
```bash
# Check account
curl http://127.0.0.1:5551/api/v1/accounts/0.0.1002

# Check transaction
curl http://127.0.0.1:5551/api/v1/transactions/0.0.1002@1234567890.123456789
```
