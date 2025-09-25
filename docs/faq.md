# Frequently Asked Questions

Common questions and answers about EntryPoint v0.8 and UserOperations.

## General Questions

### What is EntryPoint v0.8?

EntryPoint v0.8 is the latest version of the core contract in the ERC-4337 Account Abstraction ecosystem. It processes UserOperations and enables smart contract wallets with custom validation logic.

### How is v0.8 different from v0.6?

Key improvements in v0.8 include:
- **EIP-7702 Support**: Native support for EIP-7702 authorizations
- **Enhanced Security**: Improved validation mechanisms
- **Better Gas Optimization**: More efficient gas usage
- **Improved Error Handling**: Better error reporting

### What are UserOperations?

UserOperations (UserOps) are structures that describe transactions to be executed by account contracts. They enable:
- Account abstraction
- Gasless transactions
- Batch operations
- Custom validation logic

## Technical Questions

### How do I create a UserOp?

```typescript
const userOp: UserOperation = {
  sender: accountAddress,
  nonce: currentNonce,
  initCode: '0x',
  callData: encodedCallData,
  callGasLimit: 100000n,
  verificationGasLimit: 100000n,
  preVerificationGas: 21000n,
  maxFeePerGas: parseUnits('20', 'gwei'),
  maxPriorityFeePerGas: parseUnits('2', 'gwei'),
  paymasterAndData: '0x',
  signature: '0x'
};
```

### How do I sign a UserOp?

1. Get the UserOp hash from EntryPoint
2. Sign the hash with your private key
3. Include the signature in the UserOp

```typescript
const userOpHash = await entryPoint.getUserOpHash(userOp);
const signature = await signer.signMessage(ethers.getBytes(userOpHash));
```

### What gas limits should I use?

- **callGasLimit**: Estimate based on your operation
- **verificationGasLimit**: Usually 100,000-200,000
- **preVerificationGas**: Usually 21,000 (base transaction cost)

### How do I estimate gas?

```typescript
// Estimate call gas
const callGasEstimate = await entryPoint.estimateGas.call(userOp);

// Add buffer for safety
const callGasLimit = callGasEstimate * 120n / 100n;
```

## Bundler Questions

### What is a bundler?

A bundler is a service that:
- Receives UserOps from clients
- Validates UserOps
- Submits them to EntryPoint
- Handles gas payments

### How do I submit a UserOp to a bundler?

```typescript
const response = await fetch('https://api.stackup.sh/v1/bundler', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'eth_sendUserOperation',
    params: [userOp, entryPointAddress]
  })
});
```

### Which bundlers support v0.8?

Popular bundlers supporting v0.8:
- **Stackup**: `https://api.stackup.sh/v1/bundler`
- **Pimlico**: `https://api.pimlico.io/v2/sepolia/rpc`
- **Etherspot**: `https://api.etherspot.io/v1/bundler`

## Paymaster Questions

### What is a paymaster?

A paymaster is a contract that can sponsor UserOps by paying for gas fees, enabling gasless transactions.

### How do I use a paymaster?

```typescript
const userOp: UserOperation = {
  // ... other fields
  paymasterAndData: encodePaymasterData(paymasterAddress, paymasterData)
};
```

### Can I create my own paymaster?

Yes! Paymasters are smart contracts that implement the IPaymaster interface. They can have custom sponsorship logic.

## Error Handling

### Common UserOp errors

- **"Invalid signature"**: Check your signature calculation
- **"Insufficient funds"**: Ensure account has enough ETH
- **"Gas limit exceeded"**: Increase gas limits
- **"Nonce too low"**: Use correct nonce value

### How do I debug UserOp failures?

1. Check the UserOp hash
2. Verify signature calculation
3. Ensure sufficient gas limits
4. Check account balance
5. Monitor EntryPoint events

## Security Questions

### Is EntryPoint v0.8 secure?

Yes, EntryPoint v0.8 has been audited and includes:
- Enhanced validation mechanisms
- Anti-DoS protection
- Gas limit enforcement
- Signature verification

### How do I protect against replay attacks?

UserOps include a nonce field that prevents replay attacks. Each UserOp must have a unique, sequential nonce.

### Can UserOps be front-run?

UserOps can be front-run, but this is generally not profitable due to gas costs and validation requirements.

## Integration Questions

### Which wallets support EntryPoint v0.8?

Popular wallets supporting v0.8:
- **Safe**: Smart contract wallet
- **Argent**: Mobile wallet
- **Braavos**: StarkNet wallet
- **Custom wallets**: Any ERC-4337 compatible wallet

### How do I integrate with existing dApps?

Most dApps can integrate with EntryPoint v0.8 by:
1. Detecting account abstraction support
2. Using UserOps instead of regular transactions
3. Handling gas sponsorship through paymasters

## Performance Questions

### How fast are UserOps?

UserOp processing speed depends on:
- Network congestion
- Gas price settings
- Bundler performance
- EntryPoint validation time

### Can I batch multiple operations?

Yes! You can batch multiple operations in a single UserOp or submit multiple UserOps to be processed together.

### How much gas do UserOps use?

Gas usage varies by operation:
- **Simple transfer**: ~21,000 gas
- **Contract interaction**: 50,000-200,000 gas
- **Complex operations**: 200,000+ gas

## Troubleshooting

### My UserOp is stuck

1. Check if it was submitted to a bundler
2. Monitor EntryPoint events
3. Verify gas limits are sufficient
4. Check network status

### I'm getting validation errors

1. Verify your signature calculation
2. Check nonce values
3. Ensure account exists
4. Validate gas limits

### Gas estimation is wrong

1. Test on testnets first
2. Add buffer to estimates
3. Monitor gas prices
4. Use dynamic gas estimation

## Getting Help

### Where can I get support?

- **Documentation**: This guide and linked resources
- **Community**: Discord, Telegram, or GitHub discussions
- **Stack Overflow**: Tag questions with `erc-4337` or `account-abstraction`

### How do I report bugs?

Report bugs through:
- GitHub issues in relevant repositories
- Community channels
- Direct contact with maintainers

---

*Still have questions? Check out our [Troubleshooting Guide](troubleshooting/common-issues.md) or join our community Discord.*
