---
title: "Projects"
draft: false
ShowToc: true
TocOpen: true
---

## Core projects
None yet
## Active projects

### Sandwich Scanner
Sandwich Scanner is on-chain data analysis tool to detect sandwich attacks. 
As I wanted to learn about sdk development in typescript, the core detection logic will sit in a [library](https://github.com/0xnogo/sandwich-scanner).

Learnings:
* Sandwich attack detection logic
* Create a library with typescript
* Testing with jest with mocking
* CI/CD with github actions

Tech:
* sdk: Typescript, jest, ethers, axios, covalent api

### NFTLender
NFTLender is a playground nft-based lending protocol. In simple world, you can deposit your NFTs as collateral. Each NFT will weight based on the floor price of the collection. With that, borrow ETH from the protocol, reimburse your loans and get liquidated 🤑.

Learnings:
* Foundry (testing + deployment scripts)
* Logic behing lending protocol
* wagmi.js/react: hooks and web3 state management

Tech:
* Backend: Solidity, Foundry
* Frontend: React, TailwindCSS, wagmi

[Try it here](https://nftlender.vercel.app/) (no eth in the smart contract so impossible to borrow). Reach out to me if you want to have a try.

## Inactive projects
### Lobster [Hackathon project]
API to onboard non-custodial users to web3. The user will be able to mint and transfer NFTs without having to create a wallet or pay for tx fees (similar to Reddit Vault).

Learnings:
* Back-end with Express
* Public/private key generation

Tech:
* Backend: Solidity, Express, Postgres, ethers

[Repo](https://github.com/omnifient/lobster-api) of the api.