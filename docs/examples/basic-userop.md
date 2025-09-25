# Basic UserOp Examples

This guide provides practical examples of creating and submitting UserOperations with EntryPoint v0.8.

## Simple ETH Transfer

The most basic UserOp - transferring ETH from one account to another.

```typescript
import { ethers } from 'ethers';
import { UserOperation } from '@account-abstraction/sdk';

async function createTransferUserOp(
  sender: string,
  recipient: string,
  amount: string,
  nonce: bigint
): Promise<UserOperation> {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const signer = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  
  // Encode the transfer call
  const callData = encodeTransfer(recipient, ethers.parseEther(amount));
  
  const userOp: UserOperation = {
    sender,
    nonce,
    initCode: '0x',
    callData,
    callGasLimit: 21000n,
    verificationGasLimit: 100000n,
    preVerificationGas: 21000n,
    maxFeePerGas: ethers.parseUnits('20', 'gwei'),
    maxPriorityFeePerGas: ethers.parseUnits('2', 'gwei'),
    paymasterAndData: '0x',
    signature: '0x'
  };
  
  return userOp;
}

function encodeTransfer(recipient: string, amount: bigint): string {
  const iface = new ethers.Interface([
    'function execute(address,uint256,bytes)'
  ]);
  
  return iface.encodeFunctionData('execute', [recipient, amount, '0x']);
}
```

## Contract Interaction

Interacting with a smart contract through a UserOp.

```typescript
async function createContractCallUserOp(
  sender: string,
  contractAddress: string,
  functionName: string,
  params: any[],
  nonce: bigint
): Promise<UserOperation> {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  
  // Create contract interface
  const contractInterface = new ethers.Interface([
    'function transfer(address to, uint256 amount)',
    'function approve(address spender, uint256 amount)',
    'function mint(address to, uint256 amount)'
  ]);
  
  // Encode function call
  const callData = contractInterface.encodeFunctionData(functionName, params);
  
  const userOp: UserOperation = {
    sender,
    nonce,
    initCode: '0x',
    callData,
    callGasLimit: 100000n,
    verificationGasLimit: 150000n,
    preVerificationGas: 21000n,
    maxFeePerGas: ethers.parseUnits('25', 'gwei'),
    maxPriorityFeePerGas: ethers.parseUnits('3', 'gwei'),
    paymasterAndData: '0x',
    signature: '0x'
  };
  
  return userOp;
}

// Example usage
const userOp = await createContractCallUserOp(
  '0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6',
  '0xA0b86a33E6441b8c4C8C0C8C0C8C0C8C0C8C0C8C',
  'transfer',
  ['0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6', ethers.parseEther('1')],
  123n
);
```

## Batch Operations

Executing multiple operations in a single UserOp.

```typescript
async function createBatchUserOp(
  sender: string,
  operations: Array<{
    target: string;
    value: bigint;
    data: string;
  }>,
  nonce: bigint
): Promise<UserOperation> {
  // Encode batch execution
  const callData = encodeBatchExecution(operations);
  
  const userOp: UserOperation = {
    sender,
    nonce,
    initCode: '0x',
    callData,
    callGasLimit: 200000n, // Higher gas for batch
    verificationGasLimit: 150000n,
    preVerificationGas: 21000n,
    maxFeePerGas: ethers.parseUnits('30', 'gwei'),
    maxPriorityFeePerGas: ethers.parseUnits('5', 'gwei'),
    paymasterAndData: '0x',
    signature: '0x'
  };
  
  return userOp;
}

function encodeBatchExecution(operations: Array<{
  target: string;
  value: bigint;
  data: string;
}>): string {
  const iface = new ethers.Interface([
    'function executeBatch(tuple(address target, uint256 value, bytes data)[] ops)'
  ]);
  
  return iface.encodeFunctionData('executeBatch', [operations]);
}

// Example usage
const batchOps = [
  {
    target: '0xA0b86a33E6441b8c4C8C0C8C0C8C0C8C0C8C0C8C',
    value: 0n,
    data: '0xa9059cbb000000000000000000000000742d35cc...'
  },
  {
    target: '0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6',
    value: ethers.parseEther('0.1'),
    data: '0x'
  }
];

const batchUserOp = await createBatchUserOp(
  '0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6',
  batchOps,
  124n
);
```

## Paymaster Sponsored UserOp

Creating a gasless UserOp using a paymaster.

```typescript
async function createSponsoredUserOp(
  sender: string,
  target: string,
  value: bigint,
  data: string,
  nonce: bigint,
  paymasterAddress: string
): Promise<UserOperation> {
  const callData = encodeExecute(target, value, data);
  
  const userOp: UserOperation = {
    sender,
    nonce,
    initCode: '0x',
    callData,
    callGasLimit: 100000n,
    verificationGasLimit: 150000n,
    preVerificationGas: 21000n,
    maxFeePerGas: ethers.parseUnits('20', 'gwei'),
    maxPriorityFeePerGas: ethers.parseUnits('2', 'gwei'),
    paymasterAndData: encodePaymasterData(paymasterAddress, '0x'),
    signature: '0x'
  };
  
  return userOp;
}

function encodePaymasterData(paymasterAddress: string, paymasterData: string): string {
  return paymasterAddress + paymasterData.slice(2);
}

function encodeExecute(target: string, value: bigint, data: string): string {
  const iface = new ethers.Interface([
    'function execute(address,uint256,bytes)'
  ]);
  
  return iface.encodeFunctionData('execute', [target, value, data]);
}
```

## Complete Example with Signing

A complete example that creates, signs, and submits a UserOp.

```typescript
import { ethers } from 'ethers';
import { UserOperation } from '@account-abstraction/sdk';

class UserOpManager {
  private provider: ethers.JsonRpcProvider;
  private signer: ethers.Wallet;
  private entryPoint: ethers.Contract;
  
  constructor(rpcUrl: string, privateKey: string, entryPointAddress: string) {
    this.provider = new ethers.JsonRpcProvider(rpcUrl);
    this.signer = new ethers.Wallet(privateKey, this.provider);
    this.entryPoint = new ethers.Contract(
      entryPointAddress,
      [
        'function getUserOpHash(tuple) view returns (bytes32)',
        'function getNonce(address,uint192) view returns (uint256)'
      ],
      this.provider
    );
  }
  
  async createAndSignUserOp(
    target: string,
    value: bigint,
    data: string
  ): Promise<UserOperation> {
    const sender = await this.signer.getAddress();
    const nonce = await this.getNonce(sender);
    
    // Create UserOp
    const userOp: UserOperation = {
      sender,
      nonce,
      initCode: '0x',
      callData: this.encodeExecute(target, value, data),
      callGasLimit: 100000n,
      verificationGasLimit: 100000n,
      preVerificationGas: 21000n,
      maxFeePerGas: ethers.parseUnits('20', 'gwei'),
      maxPriorityFeePerGas: ethers.parseUnits('2', 'gwei'),
      paymasterAndData: '0x',
      signature: '0x'
    };
    
    // Sign UserOp
    const userOpHash = await this.entryPoint.getUserOpHash(userOp);
    const signature = await this.signer.signMessage(ethers.getBytes(userOpHash));
    
    return {
      ...userOp,
      signature
    };
  }
  
  async submitUserOp(userOp: UserOperation, bundlerUrl: string): Promise<string> {
    const response = await fetch(bundlerUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        jsonrpc: '2.0',
        id: 1,
        method: 'eth_sendUserOperation',
        params: [userOp, await this.entryPoint.getAddress()]
      })
    });
    
    const result = await response.json();
    
    if (result.error) {
      throw new Error(`Bundler error: ${result.error.message}`);
    }
    
    return result.result;
  }
  
  private async getNonce(address: string): Promise<bigint> {
    return await this.entryPoint.getNonce(address, 0);
  }
  
  private encodeExecute(target: string, value: bigint, data: string): string {
    const iface = new ethers.Interface([
      'function execute(address,uint256,bytes)'
    ]);
    
    return iface.encodeFunctionData('execute', [target, value, data]);
  }
}

// Usage example
async function main() {
  const userOpManager = new UserOpManager(
    process.env.RPC_URL!,
    process.env.PRIVATE_KEY!,
    process.env.ENTRYPOINT_ADDRESS!
  );
  
  try {
    // Create and sign UserOp
    const userOp = await userOpManager.createAndSignUserOp(
      '0x742d35Cc6634C0532925a3b8D4C9db96C4b4d8b6',
      ethers.parseEther('0.001'),
      '0x'
    );
    
    // Submit to bundler
    const userOpHash = await userOpManager.submitUserOp(
      userOp,
      'https://api.stackup.sh/v1/bundler'
    );
    
    console.log('UserOp submitted:', userOpHash);
    
  } catch (error) {
    console.error('Error:', error);
  }
}

main();
```

## Error Handling

Proper error handling for UserOp operations.

```typescript
async function submitUserOpWithRetry(
  userOp: UserOperation,
  bundlerUrl: string,
  maxRetries: number = 3
): Promise<string> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(bundlerUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          jsonrpc: '2.0',
          id: 1,
          method: 'eth_sendUserOperation',
          params: [userOp, process.env.ENTRYPOINT_ADDRESS]
        })
      });
      
      const result = await response.json();
      
      if (result.error) {
        throw new Error(`Bundler error: ${result.error.message}`);
      }
      
      return result.result;
      
    } catch (error) {
      lastError = error as Error;
      console.warn(`Attempt ${attempt} failed:`, error);
      
      if (attempt < maxRetries) {
        // Wait before retry
        await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}
```

## Next Steps

Now that you understand basic UserOp creation:

1. **[Advanced Patterns](advanced-patterns.md)** - Complex use cases and patterns
2. **[Integration Examples](integration.md)** - Real-world integration examples
3. **[Monitoring UserOps](userops/monitoring.md)** - Track UserOp execution

---

*Ready for more complex examples? Check out our [Advanced Patterns](advanced-patterns.md) guide.*
