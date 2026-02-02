# Wallet Integration with HashConnect

## Overview
HashConnect is the standard wallet connection library for Hedera dApps, supporting HashPack and other Hedera-native wallets.

## Installation
```bash
npm install @hashconnect/hashconnect @hashconnect/react
```

## React Integration

### Provider Setup
```tsx
import { HashConnectProvider } from '@hashconnect/react';

const appMetadata = {
  name: "My Hedera dApp",
  description: "A Hedera application",
  icon: "https://example.com/icon.png",
  url: "https://example.com"
};

function App() {
  return (
    <HashConnectProvider
      appMetadata={appMetadata}
      network="mainnet" // or "testnet"
      debug={false}
    >
      <YourApp />
    </HashConnectProvider>
  );
}
```

### Using the Hook
```tsx
import { useHashConnect } from '@hashconnect/react';

function WalletButton() {
  const {
    connect,
    disconnect,
    pairingData,
    accountIds,
    isConnected,
    sendTransaction
  } = useHashConnect();

  const handleConnect = async () => {
    await connect();
  };

  const handleTransaction = async () => {
    // Build transaction with @hashgraph/sdk
    const transaction = new TransferTransaction()
      .addHbarTransfer(accountIds[0], Hbar.from(-1))
      .addHbarTransfer("0.0.123456", Hbar.from(1));
    
    const result = await sendTransaction(transaction);
  };

  return (
    <button onClick={isConnected ? handleTransaction : handleConnect}>
      {isConnected ? `Connected: ${accountIds[0]}` : 'Connect Wallet'}
    </button>
  );
}
```

## Best Practices

1. **Always handle disconnection gracefully**
   - Store pairing data for session restoration
   - Clear local state on disconnect

2. **Transaction signing flow**
   - Build transaction with SDK
   - Send via HashConnect for user approval
   - Handle success/failure responses

3. **Multi-account support**
   - Users may connect multiple accounts
   - Let users select which account to use

4. **Network matching**
   - Ensure dApp network matches wallet network
   - Display clear error if mismatched

## WalletConnect Alternative
For broader wallet support (MetaMask, etc. via Hedera EVM):
```bash
npm install @walletconnect/web3-provider
```

Use with JSON-RPC Relay endpoint for EVM-compatible wallets.
