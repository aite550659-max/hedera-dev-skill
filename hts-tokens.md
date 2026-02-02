# Hedera Token Service (HTS)

## Overview
HTS provides native token functionality on Hedera - faster and cheaper than smart contract tokens.

## Installation
```bash
npm install @hashgraph/sdk
```

## Creating Tokens

### Fungible Token
```typescript
import { 
  Client, 
  TokenCreateTransaction, 
  TokenType,
  PrivateKey 
} from "@hashgraph/sdk";

const client = Client.forTestnet();
client.setOperator(operatorId, operatorKey);

const transaction = new TokenCreateTransaction()
  .setTokenName("My Token")
  .setTokenSymbol("MTK")
  .setTokenType(TokenType.FungibleCommon)
  .setDecimals(8)
  .setInitialSupply(1000000 * 10**8)
  .setTreasuryAccountId(treasuryAccountId)
  .setAdminKey(adminKey)
  .setSupplyKey(supplyKey)
  .setFreezeDefault(false);

const response = await transaction.execute(client);
const receipt = await response.getReceipt(client);
const tokenId = receipt.tokenId;
```

### NFT Collection
```typescript
const transaction = new TokenCreateTransaction()
  .setTokenName("My NFT Collection")
  .setTokenSymbol("MNFT")
  .setTokenType(TokenType.NonFungibleUnique)
  .setDecimals(0)
  .setInitialSupply(0)
  .setTreasuryAccountId(treasuryAccountId)
  .setSupplyKey(supplyKey)
  .setAdminKey(adminKey)
  .setMaxSupply(10000)
  .setSupplyType(TokenSupplyType.Finite);
```

### Minting NFTs
```typescript
import { TokenMintTransaction } from "@hashgraph/sdk";

const mintTx = new TokenMintTransaction()
  .setTokenId(tokenId)
  .addMetadata(Buffer.from("ipfs://QmXxx...")) // CID to metadata
  .setMaxTransactionFee(new Hbar(20));

const response = await mintTx.execute(client);
const receipt = await response.getReceipt(client);
const serialNumbers = receipt.serials; // [1n, 2n, ...]
```

## Token Transfers

### Fungible Transfer
```typescript
import { TransferTransaction } from "@hashgraph/sdk";

const transaction = new TransferTransaction()
  .addTokenTransfer(tokenId, senderAccountId, -100)
  .addTokenTransfer(tokenId, receiverAccountId, 100);

await transaction.execute(client);
```

### NFT Transfer
```typescript
const transaction = new TransferTransaction()
  .addNftTransfer(tokenId, serialNumber, senderAccountId, receiverAccountId);
```

## Token Association
Accounts must associate with tokens before receiving them:
```typescript
import { TokenAssociateTransaction } from "@hashgraph/sdk";

const transaction = new TokenAssociateTransaction()
  .setAccountId(accountId)
  .setTokenIds([tokenId]);

await transaction.freezeWith(client).sign(accountKey);
await transaction.execute(client);
```

## Custom Fees
```typescript
import { CustomFixedFee, CustomRoyaltyFee } from "@hashgraph/sdk";

// Fixed fee
const fixedFee = new CustomFixedFee()
  .setAmount(100)
  .setDenominatingTokenId(tokenId)
  .setFeeCollectorAccountId(collectorId);

// Royalty fee (NFTs)
const royaltyFee = new CustomRoyaltyFee()
  .setNumerator(5)
  .setDenominator(100) // 5%
  .setFeeCollectorAccountId(collectorId)
  .setFallbackFee(new CustomFixedFee().setHbarAmount(new Hbar(1)));
```

## Key Types
- **Admin Key**: Can update/delete token
- **Supply Key**: Can mint/burn
- **Freeze Key**: Can freeze/unfreeze accounts
- **Wipe Key**: Can wipe token from accounts
- **KYC Key**: Can grant/revoke KYC
- **Pause Key**: Can pause/unpause all transfers
- **Fee Schedule Key**: Can update custom fees

## Best Practices

1. **Set appropriate keys** - Don't set keys you don't need
2. **Use auto-association** - Set `maxAutoAssociations` on accounts
3. **Metadata standards** - Use HIP-412 for NFT metadata
4. **Test on testnet first** - Token creation costs HBAR
