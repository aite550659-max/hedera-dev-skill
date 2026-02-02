# AI Trust & Attestation via HCS

## Overview
Hedera Consensus Service (HCS) provides the ideal infrastructure for **AI accountability and trust**. By attesting AI outputs, decisions, and actions to HCS, you create an immutable, timestamped audit trail that enables "Trust but Verify" for AI systems.

*"AI needs trust rails. That's Hedera."*

## Real-World Example: OpenClaw (Moltbot)

[OpenClaw](https://openclaw.ai) (internally codenamed "moltbot") is an open-source AI agent framework that demonstrates how AI systems can leverage Hedera for accountability. OpenClaw agents:

- **Execute autonomous actions** (messaging, file operations, web browsing, transactions)
- **Maintain persistent memory** across sessions
- **Operate on schedules** via cron jobs and heartbeats
- **Interact with external services** (Discord, Telegram, email, wallets)

### Why HCS for AI Agents Like OpenClaw

| Challenge | HCS Solution |
|-----------|--------------|
| "Did the AI really do what it claims?" | Immutable attestation of actions |
| "When was this decision made?" | Consensus timestamp (legal weight) |
| "What reasoning led to this?" | Logged chain of thought |
| "Can the AI modify its history?" | No - HCS is append-only |
| "Is this affordable at scale?" | $0.0001 per attestation |

### OpenClaw + HCS Integration Pattern

```typescript
import { TopicMessageSubmitTransaction } from "@hashgraph/sdk";

interface OpenClawAttestation {
  type: "OPENCLAW_ACTION";
  agentId: string;         // e.g., "aite-main"
  sessionKey: string;
  action: {
    tool: string;          // "message", "exec", "browser", etc.
    parameters: object;    // Sanitized (no secrets)
    result: "success" | "failure";
  };
  reasoning?: string;      // Why the agent took this action
  timestamp: number;
  previousHash: string;    // Chain for tamper evidence
}

async function attestOpenClawAction(
  client: Client,
  auditTopic: TopicId,
  action: OpenClawAttestation
) {
  const message = JSON.stringify(action);
  
  const tx = new TopicMessageSubmitTransaction()
    .setTopicId(auditTopic)
    .setMessage(message);
  
  const response = await tx.execute(client);
  const receipt = await response.getReceipt(client);
  
  return {
    sequenceNumber: receipt.topicSequenceNumber,
    consensusTimestamp: receipt.consensusTimestamp,
    actionHash: sha256(message)
  };
}
```

### Example: Attesting a Financial Action

When an OpenClaw agent executes a transaction:

```typescript
// OpenClaw agent about to send HBAR
const attestation: OpenClawAttestation = {
  type: "OPENCLAW_ACTION",
  agentId: "aite-main",
  sessionKey: "session-abc123",
  action: {
    tool: "wallet_transfer",
    parameters: {
      network: "hedera-mainnet",
      recipient: "0.0.456789",
      amount: "10 HBAR",
      memo: "Invoice #1234"
    },
    result: "success"
  },
  reasoning: "User requested payment for invoice #1234. Amount within daily limit policy.",
  timestamp: Date.now(),
  previousHash: "abc123..."
};

// Attest BEFORE or AFTER the action (based on your audit requirements)
await attestOpenClawAction(client, auditTopicId, attestation);

// Then execute the actual transfer
await executeTransfer(...);
```

### Verification Dashboard

Build a verification UI that queries the agent's audit topic:

```typescript
async function verifyAgentHistory(agentId: string, topicId: string) {
  const url = `https://mainnet.mirrornode.hedera.com/api/v1/topics/${topicId}/messages?limit=100`;
  const response = await fetch(url);
  const data = await response.json();
  
  const actions = data.messages
    .map(msg => JSON.parse(Buffer.from(msg.message, 'base64').toString()))
    .filter(entry => entry.agentId === agentId);
  
  // Verify hash chain integrity
  let valid = true;
  for (let i = 1; i < actions.length; i++) {
    const expectedHash = sha256(JSON.stringify(actions[i - 1]));
    if (actions[i].previousHash !== expectedHash) {
      valid = false;
      console.error(`Chain broken at sequence ${i}`);
    }
  }
  
  return { actions, chainValid: valid };
}
```

### OpenClaw Audit Topic Structure

Recommended topic organization for OpenClaw deployments:

```
0.0.XXXXX - Main audit topic (all actions)
├── Memo: "OpenClaw Audit: aite-main"
├── Submit Key: Agent's signing key only
└── Messages:
    ├── [1] Agent initialization
    ├── [2] Tool call: message.send
    ├── [3] Tool call: exec.command
    ├── [4] Tool call: browser.navigate
    └── ...
```

## Why HCS for AI Trust?

| Feature | Benefit for AI |
|---------|---------------|
| **Immutable records** | AI decisions cannot be altered after the fact |
| **Consensus timestamps** | Provable ordering of AI actions |
| **Sub-3-second finality** | Real-time audit trail updates |
| **Fixed USD fees ($0.0001)** | Affordable high-frequency attestations |
| **aBFT security** | Tamper-proof even against sophisticated attacks |

## Use Cases

### 1. AI Output Provenance
Prove what an AI system produced and when:
```typescript
import { TopicMessageSubmitTransaction } from "@hashgraph/sdk";

interface AIOutputAttestation {
  type: "AI_OUTPUT";
  modelId: string;
  modelVersion: string;
  inputHash: string;      // SHA-256 of input
  outputHash: string;     // SHA-256 of output
  timestamp: number;
  metadata: {
    promptTokens: number;
    completionTokens: number;
    temperature: number;
  };
}

async function attestAIOutput(
  client: Client,
  topicId: TopicId,
  attestation: AIOutputAttestation
) {
  const message = JSON.stringify(attestation);
  
  const tx = new TopicMessageSubmitTransaction()
    .setTopicId(topicId)
    .setMessage(message);
  
  const response = await tx.execute(client);
  const receipt = await response.getReceipt(client);
  
  return {
    sequenceNumber: receipt.topicSequenceNumber,
    consensusTimestamp: receipt.consensusTimestamp
  };
}
```

### 2. AI Decision Audit Trail
Log every decision an AI agent makes:
```typescript
interface AIDecisionLog {
  type: "AI_DECISION";
  agentId: string;
  sessionId: string;
  decision: string;
  reasoning: string;       // Chain of thought summary
  confidence: number;
  inputContext: string;    // Hash of context used
  previousDecisionHash: string;  // Chain for tamper evidence
  timestamp: number;
}

// Create dedicated topic for AI agent
const auditTopicId = await createAuditTopic(client, "AI Agent Audit Log");

// Log every significant decision
await attestDecision(client, auditTopicId, {
  type: "AI_DECISION",
  agentId: "agent-001",
  sessionId: "session-xyz",
  decision: "APPROVE_TRANSACTION",
  reasoning: "User requested transfer within daily limit",
  confidence: 0.95,
  inputContext: sha256(contextData),
  previousDecisionHash: lastDecisionHash,
  timestamp: Date.now()
});
```

### 3. Autonomous Agent Transactions
When AI agents execute transactions, attest the authorization:
```typescript
interface AgentTransactionAttestation {
  type: "AGENT_TRANSACTION";
  agentId: string;
  transactionType: string;
  transactionId: string;
  authorizedBy: string;    // Policy or human approval
  amount?: string;
  recipient?: string;
  policyHash: string;      // Hash of governing policy
  timestamp: number;
}

async function executeWithAttestation(
  client: Client,
  auditTopic: TopicId,
  transaction: Transaction,
  agentId: string,
  policyHash: string
) {
  // 1. Execute the transaction
  const txResponse = await transaction.execute(client);
  const receipt = await txResponse.getReceipt(client);
  
  // 2. Attest to audit log
  const attestation: AgentTransactionAttestation = {
    type: "AGENT_TRANSACTION",
    agentId,
    transactionType: transaction.constructor.name,
    transactionId: txResponse.transactionId.toString(),
    authorizedBy: "policy:spending-limit-v1",
    policyHash,
    timestamp: Date.now()
  };
  
  await attestToHCS(client, auditTopic, attestation);
  
  return receipt;
}
```

### 4. AI Training Data Provenance
Track data used to train or fine-tune models:
```typescript
interface TrainingDataAttestation {
  type: "TRAINING_DATA";
  datasetId: string;
  datasetHash: string;
  sourceUrl?: string;
  recordCount: number;
  dateRange: { start: string; end: string };
  preprocessingSteps: string[];
  modelTargetId: string;
  timestamp: number;
}
```

### 5. Model Version Registry
Immutable record of model deployments:
```typescript
interface ModelDeploymentAttestation {
  type: "MODEL_DEPLOYMENT";
  modelId: string;
  version: string;
  weightsHash: string;
  configHash: string;
  trainingDataTopicId: string;  // Reference to training data attestations
  deployedBy: string;
  environment: "staging" | "production";
  timestamp: number;
}
```

## Implementation Pattern

### Create an AI Audit Topic
```typescript
import { TopicCreateTransaction, PrivateKey } from "@hashgraph/sdk";

async function createAIAuditTopic(
  client: Client,
  agentName: string,
  adminKey: PrivateKey
) {
  const tx = new TopicCreateTransaction()
    .setTopicMemo(`AI Audit: ${agentName}`)
    .setAdminKey(adminKey)
    .setSubmitKey(adminKey);  // Only agent can submit
  
  const response = await tx.execute(client);
  const receipt = await response.getReceipt(client);
  
  return receipt.topicId;
}
```

### Structured Attestation Schema
```typescript
// Base attestation structure
interface BaseAttestation {
  version: "1.0";
  type: string;
  agentId: string;
  timestamp: number;
  previousHash?: string;  // For chaining
}

// Extend for specific attestation types
interface OutputAttestation extends BaseAttestation {
  type: "OUTPUT";
  inputHash: string;
  outputHash: string;
  modelInfo: { id: string; version: string };
}
```

### Query Audit History
```typescript
async function getAuditHistory(topicId: string, agentId?: string) {
  const url = `https://mainnet.mirrornode.hedera.com/api/v1/topics/${topicId}/messages?limit=100`;
  const response = await fetch(url);
  const data = await response.json();
  
  return data.messages
    .map(msg => ({
      ...JSON.parse(Buffer.from(msg.message, 'base64').toString()),
      consensusTimestamp: msg.consensus_timestamp,
      sequenceNumber: msg.sequence_number
    }))
    .filter(entry => !agentId || entry.agentId === agentId);
}
```

## Best Practices

### 1. Hash Chaining
Include hash of previous attestation for tamper evidence:
```typescript
const previousHash = sha256(JSON.stringify(lastAttestation));
const newAttestation = { ...data, previousHash };
```

### 2. Structured Schemas
Use consistent JSON schemas for easy querying and verification.

### 3. Separate Topics by Purpose
- One topic per AI agent
- Or one topic per use case (decisions, outputs, transactions)

### 4. Include Verification Data
Store hashes of inputs/outputs, not raw data (privacy + efficiency).

### 5. Real-time Attestation
Attest immediately after AI actions, not in batches.

### 6. Policy References
Always reference the policy/rules governing the AI's behavior.

## Verification Flow

```
1. AI produces output
   ↓
2. Hash input + output
   ↓
3. Submit attestation to HCS
   ↓
4. Receive consensus timestamp + sequence number
   ↓
5. Store reference for later verification
   ↓
6. Auditor queries mirror node
   ↓
7. Verify hashes match, timestamps valid
   ↓
8. Trust established ✓
```

## Integration with AI Frameworks

### OpenAI/Anthropic Wrapper
```typescript
async function attestedCompletion(
  prompt: string,
  auditTopic: TopicId,
  client: Client
) {
  const inputHash = sha256(prompt);
  
  // Call AI API
  const response = await anthropic.messages.create({
    model: "claude-sonnet-4-5",
    messages: [{ role: "user", content: prompt }]
  });
  
  const outputHash = sha256(JSON.stringify(response));
  
  // Attest to HCS
  await attestAIOutput(client, auditTopic, {
    type: "AI_OUTPUT",
    modelId: "claude-sonnet-4-5",
    modelVersion: "2024-01",
    inputHash,
    outputHash,
    timestamp: Date.now(),
    metadata: {
      promptTokens: response.usage.input_tokens,
      completionTokens: response.usage.output_tokens,
      temperature: 1.0
    }
  });
  
  return response;
}
```

## Cost Estimation

| Action | HCS Fee | Volume (daily) | Daily Cost |
|--------|---------|----------------|------------|
| AI output attestation | $0.0001 | 10,000 | $1.00 |
| Decision logging | $0.0001 | 50,000 | $5.00 |
| Transaction attestation | $0.0001 | 1,000 | $0.10 |

**Total: ~$6/day for comprehensive AI audit trail**

Compare to: Custom blockchain logging, centralized databases with audit overhead, or no accountability at all.

## Summary

HCS provides AI systems with:
- ✅ **Immutable proof** of what AI did and when
- ✅ **Consensus timestamps** with legal weight
- ✅ **Affordable attestation** ($0.0001 per record)
- ✅ **Fast finality** (<3 seconds)
- ✅ **Queryable history** via mirror nodes
- ✅ **Enterprise-grade security** (aBFT)

*"Trust but Verify" - now possible for AI at scale.*
