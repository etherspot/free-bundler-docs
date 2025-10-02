#Send First Userop

This guide will help understanding how to broadcast a [7702](https://eip7702.io/) userop using smart wallet implemented [here](https://github.com/eth-infinitism/account-abstraction/blob/develop/contracts/accounts/Simple7702Account.sol)

### 1. Create Smart Account Client
Refer [here](https://viem.sh/account-abstraction/accounts/smart#smart-accounts) to understand how to create different smart account clients. **Only use wallets which support entrypoint v0.8**
=== "account.ts"
    ```ts
    import { commonClient } from './client'
    import { toSimple7702SmartAccount } from 'viem/account-abstraction'
    import { privateKeyToAccount } from 'viem/accounts'
    const owner = privateKeyToAccount('0x...') // add private key here

    export const smartAccount = await toSimple7702SmartAccount({
        client: commonClient,
        owner,
    });
    ```
=== "client.ts"
    ```ts
    import { createFreeBundler } from '@etherspot/free-bundler'
    import { publicActions, walletActions } from 'viem'
    import { mainnet } from 'viem/chains'
    const chain = mainnet

    export const commonClient = createFreeBundler({chain})
                                    .extend(publicActions)
                                    .extend(walletActions)
    ```

### 2. Send Userop
=== "example.ts"
    ```ts
    import { commonClient } from './client'
    import { toSimple7702SmartAccount } from 'viem/account-abstraction'
    import { privateKeyToAccount } from 'viem/accounts'
    import { SignAuthorizationReturnType } from 'viem'

    const owner = privateKeyToAccount('0x...') // add private key here

    const smartAccount = await toSimple7702SmartAccount({
        client: commonClient,
        owner,
    })

    console.log("wallet:: ", smartAccount.address)

    // check sender's code to decide if eip7702Auth tuple is necessary for userOp.
    const senderCode = await commonClient.getCode({
        address: smartAccount.address
    })

    let authorization: SignAuthorizationReturnType | undefined
    const { address: delegateAddress } = smartAccount.authorization

    if(senderCode !== `0xef0100${delegateAddress.toLowerCase().substring(2)}`) {
        authorization = await commonClient.signAuthorization(smartAccount.authorization)
    }

    const userOpHash = await commonClient.sendUserOperation({
        account: smartAccount,
        authorization,
        calls: [
            {
                to: "0x09FD4F6088f2025427AB1e89257A44747081Ed59",
                value: parseUnits('0.0000001', 18)
            }
        ]
    })

    console.log('userOpHash:: ', userOpHash)
    ```
=== "client.ts"
    ```ts
    import { createFreeBundler } from '@etherspot/free-bundler'
    import { publicActions, walletActions } from 'viem'
    import { mainnet } from 'viem/chains'
    const chain = mainnet

    export const commonClient = createFreeBundler({chain})
                                    .extend(publicActions)
                                    .extend(walletActions)
    ```
[![Open in StackBlitz](https://developer.stackblitz.com/img/open_in_stackblitz.svg)](https://stackblitz.com/github/etherspot/free-bundler?file=examples%2Findex.ts)

After opening StackBlitz, run:
```bash
npx tsx examples/index.ts --private-key 0x...
```
<small><em>For more run options, see the <a href="https://github.com/etherspot/free-bundler/blob/master/examples/USAGE.md">GitHub examples usage</a>.</em></small>
