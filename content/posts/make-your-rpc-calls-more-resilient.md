---
title: "Make your RPC calls more resilient"
date: 2022-10-13T10:35:27+02:00
draft: false
---

Calling an RPC node is most of the time a flawless process. The standardized [JSON-RPC](https://ethereum.org/en/developers/docs/apis/json-rpc/) API is allowing convenience libraries (ethers.js, web3.js) to abstract the calls to the node with some utility methods. 
It is getting more complicated when it comes to request intensive tasks (like bots or frontends). Live all online services, RPC nodes are fallible, which will lead to some errors on the client end. The most common one is the rate limit error. In this post, we will cover how to make your calls to RPC more resilient.

Repo with final approach: https://github.com/0xnogo/rpc-calls-resilient

## Context
I encountered issues when calling an RPC node to get all the transaction details from a specific address. The RPC node was returning a rate limit error. I was using ethers.js to make the calls. The code was looking like this:

```ts
import { ethers } from "ethers";

const ANKR_MAINNET_RPC = 'RPC_LINK_HERE';

const provider = new ethers.providers.JsonRpcProvider(ANKR_MAINNET_RPC);

async function getTxHashesFromBlock(blockNo: number): Promise<string[]> {
  const blockData = await provider.getBlock(blockNo);
  return blockData.transactions;
}

async function getTxReceipt(txHash: string): Promise<ethers.providers.TransactionReceipt> {
  return await provider.getTransactionReceipt(txHash);
}

async function getTxReceiptFromBlock(blockNo: number): Promise<ethers.providers.TransactionReceipt[]> {
  const txHashes = await getTxHashesFromBlock(blockNo);
  const txReceipts = await Promise.all(txHashes.map(getTxReceipt));
  return txReceipts;
}

async function init(): Promise<void> {
  try {
    for (let i = 12345678; i < 15345678; i++) {
      await getTxReceiptFromBlock(i);
      console.log(`Block ${i} done`);
    }
  } catch (error) {
    console.log(error);
  }
}

init();
```

A simple code to get all the transactions from a specific block range. The code was working fine for a while, but then it started to fail with the following error:

```bash
requests.exceptions.HTTPError: 503 Server Error: Service Unavailable for url: https://mainnet.infura.io/v3/
```

## Solutions

### Retries

The first solution is to retry the call. The most common way to do it is to use a recursive function. The function will be called until it succeeds or the maximum number of retries is reached. The code will look like this:

```ts
async function callProviderWithRetries<T>(
  promise: Promise<T>,
  retriesLeft = 10,
): Promise<T> {
  try {
    const data = await promise;
    return data;
  } catch (error) {
    if (retriesLeft === 0) {
      console.log(`${retriesLeft} retries left`);
      return Promise.reject(error);
    }
    return callProviderWithRetries(promise, retriesLeft - 1);
  }
}
```

The usage of the function will be the following:

```ts
async function getTxHashesFromBlock(blockNo: number): Promise<string[]> {
  const blockData = await callProviderWithRetries(provider.getBlock(blockNo));
  return blockData.transactions;
}
```

### Retry with exponential backoff

The second solution is to retry the call with exponential backoff. The idea is to wait a bit longer between each retry. The code will look like this:

```ts
async function callProviderWithRetriesWithBackoff<T>(
  promise: Promise<T>,
  retriesLeft = 10,
): Promise<T> {
  try {
    const data = await promise;
    return data;
  } catch (error) {
    if (retriesLeft === 0) {
      console.log(`${retriesLeft} retries left`);
      return Promise.reject(error);
    }
    wait(1000 * (10 - retriesLeft));
    return callProviderWithRetries(promise, retriesLeft - 1);
  }
}
```

First failing call will wait 0 second, the second 1s * (10 - 9) = 1s, the third 1s * (10 - 8) = 2s, etc. The usage of the function will be the following:

```ts
async function getTxHashesFromBlock(blockNo: number): Promise<string[]> {
  const blockData = await callProviderWithRetriesWithBackoff(provider.getBlock(blockNo));
  return blockData.transactions;
}
```

### Load balancing

The third solution is to use a load balancer. The load balancer will redirect the calls to a different node if the first one is not responding. 
For this example, we need to have several RPC nodes available.
The code will look like this:

```ts
async function callProviderWithRetriesOnDifferentRPC(
  fn: (provider: ethers.providers.JsonRpcProvider) => Promise<any>,
  retries = 0,
): Promise<any> {
  try {
    const data = await fn(providers[retries]!);
    return data;
  } catch (error) {
    if (retries > providers.length - 1) {
      console.log(`${retries} retry. Trying with different provider`);
      return Promise.reject(error);
    }
    return callProviderWithRetriesOnDifferentRPC(fn, retries + 1);
  }
}
```

The logic here is to try with the first provider, if it fails, try with the second one, etc. We are passing a function with a provider as an input which is allowing the flexibility we want. The usage of the function will be the following:

```ts
async function getTxHashesFromBlock(blockNo: number): Promise<string[]> {
  const blockData = await callProviderWithRetriesOnDifferentRPC((provider) => provider.getBlock(blockNo));
  return blockData.transactions;
}
```
> We could also combine the two solutions and use a load balancer with an exponential backoff and retries per provider.

## Call a contract

The previous solutions are working well when calling the RPC node. But what if you want to call a contract? Some changes need to be applied to call the contract instead of the provider but the logic remains the same. The code will look like this:

```ts
async function callContractMethodWithRetries(
  method: string,
  params: any[],
  address: string,
  _interface: Interface,
  retries = 0,
): Promise<any> {
  try {
    const contractPair = new ethers.Contract(
      address,
      _interface,
      providers[retries],
    );
    const data = await contractPair[method](...params);
    return data;
  } catch (error) {
    if (retries > providers.length - 1) {
      return Promise.reject(error);
    }
    return callContractMethodWithRetries(
      method,
      params,
      address,
      _interface,
      retries + 1,
    );
  }
}
```

The usage of the function will be the following:

```ts
try {
    const token0 = await callContractMethodWithRetries(
      'token0',
      [],
      "UNISWAPV2_PAIR_ADDRESS",
      UNISWAPV2_PAIR_INTERFACE,
    );

    const token1 = await callContractMethodWithRetries(
      'token0',
      [],
      "UNISWAPV2_PAIR_ADDRESS",
      UNISWAPV2_PAIR_INTERFACE,
    );

    console.log(token0, token1);
  } catch (error) {
    console.log(error);
  }
```

## Wrapper

The code above is not very DRY. We can create a wrapper to make the calls more resilient. The code will look like this:

```ts
export class ProviderWrapper {
  private providers: ethers.providers.JsonRpcProvider[] = [];

  constructor(
    provider: ethers.providers.JsonRpcProvider,
    backupProviders?: ethers.providers.JsonRpcProvider[],
  ) {
    this.providers = [provider, ...(backupProviders ? backupProviders : [])];
  }

  public async callProviderWithRetries(
    fn: (provider: ethers.providers.JsonRpcProvider) => Promise<any>,
    retries = 0,
  ): Promise<any> {
    ...
  }

  public async callContractMethodWithRetries(
    method: string,
    params: any[],
    address: string,
    _interface: Interface,
    retries = 0,
  ): Promise<any> {
    ...
  }

  public async callProviderWithRetriesAndWait(
    fn: (provider: ethers.providers.JsonRpcProvider) => Promise<any>,
    retries = 0,
  ): Promise<any> {
    ...
  }

  public getProvider(retries: number): ethers.providers.JsonRpcProvider {
    return this.providers[retries]!;
  }

  private wait(ms: number) {
    ...
  }
}
```

Most of the methods above are derived from the previous examples. The main thing to note here is how the constructor is built. We are passing a provider as a mandatory parameter and a list of backup providers as an optional parameter. 

For the usage on both provider and contract calls, check the final code in the repository:
* [callingProvider.ts](https://github.com/0xnogo/rpc-calls-resilient/blob/main/src/callingProvider.ts)
* [callingContract.ts](https://github.com/0xnogo/rpc-calls-resilient/blob/main/src/callingContract.ts)

## Conclusion
This approach will allow you some reusability and flexibility. Feel free to reach out in twitter if you have any questions or suggestions.

### Credits
* [Chainstack](https://chainstack.com/tips-to-handle-rpc-request-errors/)