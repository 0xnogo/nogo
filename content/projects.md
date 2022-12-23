---
title: "Projects"
draft: false
ShowToc: true
TocOpen: true
---

## Full-stack
### NFTLender
* Build time: 1 month 
* Repo(s): [Contracts](https://github.com/0xnogo/nft-lender-contract), [Frontend](https://github.com/0xnogo/nft-lender-frontend)

NFTLender is a playground nft-based lending protocol. In simple world, you can deposit your NFTs as collateral. Each NFT will weight based on the floor price of the collection. With that, borrow ETH from the protocol, reimburse your loans and get liquidated 🤑.

Learnings:
* Foundry (testing + deployment scripts)
* Logic behing lending protocol
* wagmi.js/react: hooks and web3 state management

Tech:
* Backend: Solidity, Foundry
* Frontend: React, TailwindCSS, wagmi

[Try it here](https://nftlender.vercel.app/) (no eth in the smart contract so impossible to borrow). Reach out to me if you want to have a try.

### Lobster [Hackathon project]
* Build time: 48 hours 
* Repo(s): [API](https://github.com/omnifient/lobster-api), [Frontend](https://github.com/thekidnamedkd/hapi-meal-fe)

API to onboard non-custodial users to web3. The user will be able to mint and transfer NFTs without having to create a wallet or pay for tx fees (similar to Reddit Vault).

Learnings:
* Back-end with Express
* Public/private key generation

Tech:
* Backend: Solidity, Express, Postgres, ethers

## Solidity
### Sandwich Scanner
* Build time: 2 weeks
* Repo(s): [library](https://github.com/0xnogo/sandwich-scanner)

Sandwich Scanner is on-chain data analysis tool to detect sandwich attacks. 
As I wanted to learn about sdk development in typescript, the core detection logic will sit in a [library](https://github.com/0xnogo/sandwich-scanner).

Learnings:
* Sandwich attack detection logic
* Create a library with typescript
* Testing with jest with mocking
* CI/CD with github actions

Tech:
* sdk: Typescript, jest, ethers, axios, covalent api

## Rust
### On-chain smart contract analysis
* Build time: 2 weeks
* Repo(s): private (for now)

Script to analyze smart contracts on-chain. The script will filter all smart contract creation transactions, get the bytecode and perform a static analysis on it. The analysis is done by scrapping etherscan for verified contracts. The code has been written in Rust in a modular way to be able to add more analysis in the future.

Learnings:
* rust concepts: async, impl, traits, lifetimes, option, result, error handling...

Tech:
* rust lib: tokio, reqwest, ethers.rs, futures, regex, scraper


