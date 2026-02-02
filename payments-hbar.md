# HBAR Payments & Treasury Operations

## Overview
Native HBAR transfers, multi-sig treasury management, staking, and payment patterns for Hedera applications.

## Basic HBAR Transfer

### Simple Transfer
```typescript
import { TransferTransaction, Hbar, Client } from "@hashgraph/sdk";

const client = Client.forMainnet();
client.setOperator(operatorId, operatorKey);

const tx = new TransferTransaction()
  .addHbarTransfer(senderAccountId, Hbar.from(-10))  // Debit
  .addHbarTransfer(receiverAccountId, Hbar.from(10)) // Credit
  .setTransactionMemo("Payment for services");

const response = await tx.execute(client);
const receipt = await response.getReceipt(client);

console.log(`Transaction ID: ${response.transactionId}`);
console.log(`Status: ${receipt.status}`);
```

### Transfer with Exchange Rate Check
```typescript
import { Hbar, HbarUnit } from "@hashgraph/sdk";

// Create amounts in different units
const amountInHbar = new Hbar(10);                    // 10 HBAR
const amountInTinybars = Hbar.fromTinybars(1000000000); // 10 HBAR
const amountFromString = Hbar.fromString("10 hbar");  // 10 HBAR

// Convert units
console.log(`Tinybars: ${amountInHbar.toTinybars()}`); // 1000000000
console.log(`HBAR: ${amountInTinybars.toBigNumber()}`); // 10

// USD-denominated (uses live exchange rate at transaction time)
const usdAmount = new Hbar(5, HbarUnit.Hbar); // For fixed HBAR
// Note: For USD-fixed amounts, calculate at application layer
```

## Multi-Signature Treasury

### Create Multi-Sig Account
```typescript
import { 
  AccountCreateTransaction, 
  KeyList, 
  PrivateKey 
} from "@hashgraph/sdk";

// Generate keys for signers
const key1 = PrivateKey.generate();
const key2 = PrivateKey.generate();
const key3 = PrivateKey.generate();

// 2-of-3 threshold key
const thresholdKey = new KeyList([
  key1.publicKey,
  key2.publicKey,
  key3.publicKey
], 2); // Requires 2 signatures

const tx = new AccountCreateTransaction()
  .setKey(thresholdKey)
  .setInitialBalance(Hbar.from(100))
  .setAccountMemo("Treasury Account");

const response = await tx.execute(client);
const receipt = await response.getReceipt(client);
const treasuryAccountId = receipt.accountId;
```

### Execute Multi-Sig Transfer
```typescript
// Build the transaction
const transferTx = new TransferTransaction()
  .addHbarTransfer(treasuryAccountId, Hbar.from(-50))
  .addHbarTransfer(recipientId, Hbar.from(50))
  .freezeWith(client);

// Sign with threshold signers (need 2 of 3)
const signedTx = await transferTx
  .sign(key1)
  .then(tx => tx.sign(key2));

// Execute
const response = await signedTx.execute(client);
const receipt = await response.getReceipt(client);
```

## Scheduled Payments

### Create Scheduled Transaction
```typescript
import { 
  ScheduleCreateTransaction,
  TransferTransaction,
  Timestamp 
} from "@hashgraph/sdk";

// Inner transaction to schedule
const innerTx = new TransferTransaction()
  .addHbarTransfer(treasuryId, Hbar.from(-100))
  .addHbarTransfer(vendorId, Hbar.from(100));

// Schedule it
const scheduleTx = new ScheduleCreateTransaction()
  .setScheduledTransaction(innerTx)
  .setAdminKey(adminKey)
  .setPayerAccountId(treasuryId)
  .setScheduleMemo("Monthly vendor payment");

const response = await scheduleTx.execute(client);
const receipt = await response.getReceipt(client);
const scheduleId = receipt.scheduleId;
```

### Sign Scheduled Transaction
```typescript
import { ScheduleSignTransaction } from "@hashgraph/sdk";

// Another signer adds their signature
const signTx = new ScheduleSignTransaction()
  .setScheduleId(scheduleId);

await signTx.freezeWith(client).sign(signerKey);
const response = await signTx.execute(client);

// Transaction executes when threshold is met
```

### Delete Scheduled Transaction
```typescript
import { ScheduleDeleteTransaction } from "@hashgraph/sdk";

const deleteTx = new ScheduleDeleteTransaction()
  .setScheduleId(scheduleId);

await deleteTx.freezeWith(client).sign(adminKey);
await deleteTx.execute(client);
```

## Staking

### Stake to Node
```typescript
import { AccountUpdateTransaction } from "@hashgraph/sdk";

const stakeTx = new AccountUpdateTransaction()
  .setAccountId(myAccountId)
  .setStakedNodeId(0)  // Node ID to stake to
  .setDeclineStakingReward(false);

await stakeTx.freezeWith(client).sign(accountKey);
await stakeTx.execute(client);
```

### Stake to Account (Indirect)
```typescript
const stakeTx = new AccountUpdateTransaction()
  .setAccountId(myAccountId)
  .setStakedAccountId(stakedAccountId); // Stake to another account

await stakeTx.freezeWith(client).sign(accountKey);
await stakeTx.execute(client);
```

### Check Staking Info
```typescript
import { AccountInfoQuery } from "@hashgraph/sdk";

const info = await new AccountInfoQuery()
  .setAccountId(myAccountId)
  .execute(client);

console.log(`Staked to node: ${info.stakingInfo.stakedNodeId}`);
console.log(`Staked to account: ${info.stakingInfo.stakedAccountId}`);
console.log(`Pending reward: ${info.stakingInfo.pendingReward}`);
console.log(`Stake period start: ${info.stakingInfo.stakePeriodStart}`);
```

## Allowances (Approved Spending)

### Approve HBAR Allowance
```typescript
import { AccountAllowanceApproveTransaction } from "@hashgraph/sdk";

const approveTx = new AccountAllowanceApproveTransaction()
  .approveHbarAllowance(ownerAccountId, spenderAccountId, Hbar.from(100));

await approveTx.freezeWith(client).sign(ownerKey);
await approveTx.execute(client);
```

### Spend from Allowance
```typescript
const spendTx = new TransferTransaction()
  .addApprovedHbarTransfer(ownerAccountId, Hbar.from(-10))
  .addHbarTransfer(recipientId, Hbar.from(10))
  .setTransactionId(TransactionId.generate(spenderAccountId));

await spendTx.freezeWith(client).sign(spenderKey);
await spendTx.execute(client);
```

### Delete Allowance
```typescript
import { AccountAllowanceDeleteTransaction } from "@hashgraph/sdk";

const deleteTx = new AccountAllowanceDeleteTransaction()
  .deleteAllHbarAllowances(ownerAccountId);

await deleteTx.freezeWith(client).sign(ownerKey);
await deleteTx.execute(client);
```

## Payment Patterns

### Invoice Payment with Memo
```typescript
const payment = new TransferTransaction()
  .addHbarTransfer(payerAccountId, Hbar.from(-amount))
  .addHbarTransfer(merchantAccountId, Hbar.from(amount))
  .setTransactionMemo(`INV-${invoiceId}`);  // Reference for reconciliation
```

### Split Payment
```typescript
const splitPayment = new TransferTransaction()
  .addHbarTransfer(buyerAccountId, Hbar.from(-100))
  .addHbarTransfer(sellerAccountId, Hbar.from(95))    // 95%
  .addHbarTransfer(platformAccountId, Hbar.from(5));  // 5% fee
```

### Batch Payments
```typescript
const batchTx = new TransferTransaction()
  .addHbarTransfer(treasuryId, Hbar.from(-300));

// Add multiple recipients
for (const recipient of recipients) {
  batchTx.addHbarTransfer(recipient.accountId, Hbar.from(recipient.amount));
}

await batchTx.execute(client);
```

### Escrow Pattern
```typescript
// 1. Create escrow account with threshold key
const escrowKey = new KeyList([buyerKey.publicKey, sellerKey.publicKey], 2);
const escrowAccount = await createAccount(client, escrowKey, Hbar.from(0));

// 2. Buyer funds escrow
await transfer(client, buyerAccountId, escrowAccount, amount);

// 3. On completion, both sign release
const releaseTx = new TransferTransaction()
  .addHbarTransfer(escrowAccount, Hbar.from(-amount))
  .addHbarTransfer(sellerAccountId, Hbar.from(amount))
  .freezeWith(client);

await releaseTx.sign(buyerKey).then(tx => tx.sign(sellerKey));
await releaseTx.execute(client);
```

## Fee Management

### Set Max Transaction Fee
```typescript
const tx = new TransferTransaction()
  .addHbarTransfer(...)
  .setMaxTransactionFee(Hbar.from(2));  // Cap at 2 HBAR
```

### Query Transaction Cost
```typescript
import { TransferTransaction, Hbar } from "@hashgraph/sdk";

const tx = new TransferTransaction()
  .addHbarTransfer(sender, Hbar.from(-10))
  .addHbarTransfer(receiver, Hbar.from(10));

const cost = await tx.getCost(client);
console.log(`Estimated cost: ${cost.toBigNumber()} HBAR`);
```

## AI Agent Payment Authorization

### OpenClaw Policy-Based Payments
```typescript
interface PaymentPolicy {
  maxSingleTransaction: number;  // HBAR
  dailyLimit: number;           // HBAR
  allowedRecipients?: string[];  // Account IDs
  requireAttestation: boolean;
}

async function executeAgentPayment(
  client: Client,
  auditTopic: TopicId,
  payment: { to: string; amount: number; reason: string },
  policy: PaymentPolicy
): Promise<TransactionReceipt> {
  // 1. Validate against policy
  if (payment.amount > policy.maxSingleTransaction) {
    throw new Error(`Amount ${payment.amount} exceeds single transaction limit`);
  }
  
  // 2. Attest to HCS BEFORE execution
  if (policy.requireAttestation) {
    await attestPayment(client, auditTopic, {
      type: "PAYMENT_INITIATED",
      amount: payment.amount,
      recipient: payment.to,
      reason: payment.reason,
      policyHash: sha256(JSON.stringify(policy)),
      timestamp: Date.now()
    });
  }
  
  // 3. Execute payment
  const tx = new TransferTransaction()
    .addHbarTransfer(agentAccountId, Hbar.from(-payment.amount))
    .addHbarTransfer(AccountId.fromString(payment.to), Hbar.from(payment.amount))
    .setTransactionMemo(payment.reason);
  
  const response = await tx.execute(client);
  const receipt = await response.getReceipt(client);
  
  // 4. Attest completion
  if (policy.requireAttestation) {
    await attestPayment(client, auditTopic, {
      type: "PAYMENT_COMPLETED",
      transactionId: response.transactionId.toString(),
      status: receipt.status.toString(),
      timestamp: Date.now()
    });
  }
  
  return receipt;
}
```

## Best Practices

### 1. Always Set Transaction Memo
Memos enable reconciliation and auditing:
```typescript
.setTransactionMemo(`ORDER-${orderId}|${customerId}`)
```

### 2. Use Max Transaction Fee
Protect against unexpected costs:
```typescript
.setMaxTransactionFee(Hbar.from(1))
```

### 3. Verify Receipts
Always check transaction success:
```typescript
const receipt = await response.getReceipt(client);
if (receipt.status.toString() !== "SUCCESS") {
  throw new Error(`Transaction failed: ${receipt.status}`);
}
```

### 4. Handle Network Errors
```typescript
try {
  const response = await tx.execute(client);
  const receipt = await response.getReceipt(client);
} catch (error) {
  if (error.status === Status.InsufficientAccountBalance) {
    // Handle insufficient funds
  } else if (error.status === Status.InvalidSignature) {
    // Handle signature issues
  }
}
```

### 5. Use Testnet First
Always test payment flows on testnet before mainnet deployment.

## Fee Reference

| Operation | Approximate Cost |
|-----------|-----------------|
| HBAR Transfer | $0.0001 |
| Account Create | $0.05 |
| Schedule Create | $0.01 |
| Schedule Sign | $0.0001 |
| Account Update (Staking) | $0.0001 |
| Allowance Approve | $0.05 |

*Fees are fixed in USD and converted to HBAR at execution time.*
