# Hedera Consensus Service (HCS)

## Overview
HCS provides decentralized, timestamped, ordered message logging. Perfect for:
- Audit trails
- Supply chain tracking
- Decentralized logging
- Event ordering for off-chain systems
- NFT provenance

## Creating a Topic

### Public Topic (anyone can submit)
```typescript
import { TopicCreateTransaction, Client } from "@hashgraph/sdk";

const client = Client.forTestnet();
client.setOperator(operatorId, operatorKey);

const transaction = new TopicCreateTransaction()
  .setTopicMemo("My audit log");

const response = await transaction.execute(client);
const receipt = await response.getReceipt(client);
const topicId = receipt.topicId;
console.log(`Topic ID: ${topicId}`);
```

### Private Topic (submit key required)
```typescript
const transaction = new TopicCreateTransaction()
  .setTopicMemo("Private messages")
  .setSubmitKey(submitKey)
  .setAdminKey(adminKey);
```

## Submitting Messages

```typescript
import { TopicMessageSubmitTransaction } from "@hashgraph/sdk";

const message = JSON.stringify({
  action: "transfer",
  from: "0.0.123",
  to: "0.0.456",
  amount: 100,
  timestamp: Date.now()
});

const transaction = new TopicMessageSubmitTransaction()
  .setTopicId(topicId)
  .setMessage(message);

const response = await transaction.execute(client);
const receipt = await response.getReceipt(client);
console.log(`Sequence number: ${receipt.topicSequenceNumber}`);
```

### Large Messages (chunking)
For messages > 1024 bytes, use chunking:
```typescript
const transaction = new TopicMessageSubmitTransaction()
  .setTopicId(topicId)
  .setMessage(largeMessage)
  .setMaxChunks(10); // Max chunks for this message
```

## Subscribing to Messages

### Real-time via Mirror Node
```typescript
import { TopicMessageQuery } from "@hashgraph/sdk";

new TopicMessageQuery()
  .setTopicId(topicId)
  .setStartTime(0) // From beginning, or specific timestamp
  .subscribe(client, null, (message) => {
    const contents = Buffer.from(message.contents).toString();
    console.log(`Received: ${contents}`);
    console.log(`Sequence: ${message.sequenceNumber}`);
    console.log(`Timestamp: ${message.consensusTimestamp}`);
  });
```

### Historical via Mirror Node REST
```typescript
const response = await fetch(
  `https://mainnet.mirrornode.hedera.com/api/v1/topics/${topicId}/messages`
);
const data = await response.json();
data.messages.forEach(msg => {
  const decoded = Buffer.from(msg.message, 'base64').toString();
  console.log(decoded);
});
```

## Topic Management

### Update Topic
```typescript
import { TopicUpdateTransaction } from "@hashgraph/sdk";

const transaction = new TopicUpdateTransaction()
  .setTopicId(topicId)
  .setTopicMemo("Updated memo")
  .setSubmitKey(newSubmitKey);

await transaction.freezeWith(client).sign(adminKey);
await transaction.execute(client);
```

### Delete Topic
```typescript
import { TopicDeleteTransaction } from "@hashgraph/sdk";

const transaction = new TopicDeleteTransaction()
  .setTopicId(topicId);

await transaction.freezeWith(client).sign(adminKey);
await transaction.execute(client);
```

## Use Cases

### Audit Trail
```typescript
const auditEntry = {
  type: "USER_ACTION",
  userId: "user123",
  action: "LOGIN",
  ip: "192.168.1.1",
  timestamp: Date.now(),
  hash: sha256(previousEntry) // Chain entries
};

await new TopicMessageSubmitTransaction()
  .setTopicId(auditTopicId)
  .setMessage(JSON.stringify(auditEntry))
  .execute(client);
```

### NFT Provenance
```typescript
const provenanceEntry = {
  nftId: "0.0.12345/1",
  event: "TRANSFER",
  from: "0.0.111",
  to: "0.0.222",
  price: "100 HBAR",
  timestamp: Date.now()
};
```

## Best Practices

1. **Structure your messages** - Use consistent JSON schemas
2. **Include timestamps** - Client-side timestamps complement consensus timestamps
3. **Hash chaining** - Include hash of previous message for tamper evidence
4. **Rate limiting** - Max ~100 messages/second per topic
5. **Message size** - Keep under 1024 bytes when possible
6. **Topic lifecycle** - Plan for topic rotation for high-volume use cases
