# Hedera Security Checklist

## Key Management

### ❌ Never Do
- Hardcode private keys in source code
- Commit keys to version control
- Log private keys or mnemonics
- Store keys in plain text
- Share operator keys between environments

### ✅ Always Do
- Use environment variables or secret managers
- Use separate keys for testnet/mainnet
- Implement key rotation procedures
- Use multi-sig for high-value accounts
- Use HSMs for production treasury keys

```typescript
// ❌ Bad
const key = PrivateKey.fromString("302e020100...");

// ✅ Good
const key = PrivateKey.fromString(process.env.HEDERA_PRIVATE_KEY!);
```

## Account Security

### Key Structures
```typescript
// Single key - simple but risky
const singleKey = PrivateKey.generate();

// Threshold key - requires M of N signatures
const thresholdKey = new KeyList([key1, key2, key3], 2); // 2 of 3

// Key list - requires all signatures
const keyList = new KeyList([key1, key2]);
```

### Account Protection
- Set account memo for identification
- Use auto-renew account for fees
- Monitor account balance
- Set up alerts for large transfers

## Token Security

### Token Key Best Practices
```typescript
const tokenCreate = new TokenCreateTransaction()
  .setAdminKey(adminKey)        // Can update token, set to null to make immutable
  .setSupplyKey(supplyKey)      // Can mint/burn
  .setFreezeKey(freezeKey)      // Can freeze accounts (compliance)
  .setWipeKey(wipeKey)          // Can wipe from accounts (compliance)
  .setKycKey(kycKey)            // Can grant/revoke KYC
  .setPauseKey(pauseKey)        // Can pause all transfers
  .setFeeScheduleKey(feeKey);   // Can update fees
```

### Immutable Tokens
For maximum decentralization, omit keys:
```typescript
// No admin key = token cannot be modified/deleted
const tokenCreate = new TokenCreateTransaction()
  .setTokenName("Immutable Token")
  .setSupplyKey(supplyKey)  // Keep only if minting needed
  // No setAdminKey() = immutable
```

## Smart Contract Security

### Solidity Best Practices
- Follow OpenZeppelin patterns
- Use reentrancy guards
- Check-effects-interactions pattern
- Validate all inputs
- Use SafeMath (or Solidity 0.8+)

### HTS Precompile Security
```solidity
// Always check response codes
int responseCode = IHederaTokenService(HTS_PRECOMPILE)
    .transferToken(token, from, to, amount);
    
require(responseCode == HederaResponseCodes.SUCCESS, "Transfer failed");
```

### Audit Recommendations
- Get professional audit for mainnet contracts
- Use static analysis tools (Slither, Mythril)
- Test edge cases thoroughly
- Document all assumptions

## Transaction Security

### Max Transaction Fee
Always set reasonable limits:
```typescript
const tx = new TransferTransaction()
  .setMaxTransactionFee(new Hbar(2)) // Cap fees
  .addHbarTransfer(...)
```

### Transaction Validation
```typescript
// Verify transaction before signing
const bytes = tx.toBytes();
const recreated = Transaction.fromBytes(bytes);
// Inspect recreated transaction details
```

### Scheduled Transactions
For sensitive operations, use scheduled transactions:
```typescript
const scheduledTx = new ScheduleCreateTransaction()
  .setScheduledTransaction(innerTx)
  .setPayerAccountId(payerId)
  .setAdminKey(adminKey);
```

## Network Security

### Environment Separation
```typescript
// Use different operators per environment
const client = network === 'mainnet'
  ? Client.forMainnet()
  : Client.forTestnet();

client.setOperator(
  process.env[`${network.toUpperCase()}_OPERATOR_ID`]!,
  process.env[`${network.toUpperCase()}_OPERATOR_KEY`]!
);
```

### Rate Limiting
- Implement client-side rate limiting
- Handle BUSY responses gracefully
- Use exponential backoff

## Monitoring & Alerts

### Set Up Monitoring For
- Account balance changes
- Failed transactions
- Unusual activity patterns
- Contract events/logs
- Topic message anomalies

### Tools
- HashScan for manual inspection
- Mirror node REST for automated checks
- Custom alerting via HCS or webhooks

## Incident Response

1. **Immediately** - Rotate compromised keys
2. **Assess** - Check transaction history
3. **Contain** - Pause tokens if keys available
4. **Document** - Record all findings
5. **Remediate** - Update security practices
