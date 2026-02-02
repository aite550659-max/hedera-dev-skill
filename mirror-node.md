# Mirror Node Queries

## Overview
Mirror nodes provide REST and gRPC APIs for querying historical data on Hedera. Use for:
- Account balances and history
- Transaction history
- Token information
- NFT metadata
- Topic messages
- Contract logs

## Endpoints

| Network | REST API |
|---------|----------|
| Mainnet | `https://mainnet.mirrornode.hedera.com/api/v1/` |
| Testnet | `https://testnet.mirrornode.hedera.com/api/v1/` |
| Previewnet | `https://previewnet.mirrornode.hedera.com/api/v1/` |

## Common Queries

### Account Info
```typescript
const accountId = "0.0.123456";
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/accounts/${accountId}`
);
const account = await response.json();

console.log(`Balance: ${account.balance.balance / 100000000} HBAR`);
console.log(`Created: ${account.created_timestamp}`);
```

### Account Transactions
```typescript
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/transactions?account.id=${accountId}&limit=25`
);
const data = await response.json();

data.transactions.forEach(tx => {
  console.log(`${tx.transaction_id}: ${tx.name} - ${tx.result}`);
});
```

### Token Info
```typescript
const tokenId = "0.0.789012";
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/tokens/${tokenId}`
);
const token = await response.json();

console.log(`Name: ${token.name}`);
console.log(`Symbol: ${token.symbol}`);
console.log(`Supply: ${token.total_supply}`);
console.log(`Type: ${token.type}`);
```

### NFT Info
```typescript
// Get all NFTs in collection
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/tokens/${tokenId}/nfts`
);

// Get specific NFT
const nftResponse = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/tokens/${tokenId}/nfts/${serialNumber}`
);
const nft = await nftResponse.json();

// Decode metadata
const metadata = Buffer.from(nft.metadata, 'base64').toString();
```

### Topic Messages
```typescript
const topicId = "0.0.345678";
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/topics/${topicId}/messages?limit=100`
);
const data = await response.json();

data.messages.forEach(msg => {
  const decoded = Buffer.from(msg.message, 'base64').toString();
  console.log(`[${msg.sequence_number}] ${decoded}`);
});
```

### Contract Logs
```typescript
const contractId = "0.0.456789";
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/contracts/${contractId}/results/logs`
);
const logs = await response.json();
```

## Pagination

Mirror node uses cursor-based pagination:
```typescript
async function getAllTransactions(accountId: string) {
  let url = `https://mainnet.mirrornode.hedera.com/api/v1/transactions?account.id=${accountId}&limit=100`;
  let allTransactions = [];
  
  while (url) {
    const response = await fetch(url);
    const data = await response.json();
    allTransactions.push(...data.transactions);
    
    // Get next page URL from links
    url = data.links?.next 
      ? `https://mainnet.mirrornode.hedera.com${data.links.next}` 
      : null;
  }
  
  return allTransactions;
}
```

## Filtering

Common query parameters:
- `limit` - Results per page (max 100)
- `order` - `asc` or `desc`
- `timestamp` - Filter by timestamp range
- `account.id` - Filter by account
- `token.id` - Filter by token
- `result` - Filter by transaction result

```typescript
// Transactions in time range
const url = `https://mainnet.mirrornode.hedera.com/api/v1/transactions?` +
  `account.id=${accountId}&` +
  `timestamp=gte:1704067200&` +  // >= Jan 1, 2024
  `timestamp=lte:1706745600&` +  // <= Feb 1, 2024
  `limit=100&order=asc`;
```

## Best Practices

1. **Use mirror nodes for queries** - Never query consensus nodes for data
2. **Respect rate limits** - Implement exponential backoff
3. **Cache responses** - Historical data doesn't change
4. **Handle pagination** - Large result sets are paginated
5. **Timestamp format** - Use seconds.nanoseconds format
6. **Error handling** - Mirror nodes may have brief delays
