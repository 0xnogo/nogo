---
title: "Daily log"
draft: false
---

Here is a daily log of my work and learning. The sole goal of this log is to keep track of my progress and to keep me accountable. I will try to keep it as short as possible.

## 2022
### September
#### 29
* Quick reading and set up of this hugo blog
* finished sprint 2 around going deep on some defi protocols. Looking for the next project to work on
* worked on my backlog of projects
#### 30
* working on a sandwich attack detector
* checked [mev-inspect-py](https://github.com/flashbots/mev-inspect-py/blob/main/mev_inspect/sandwiches.py): understand how to classify a sandwich attack 
* drafted an naive approach to detect sandwich attacks with a tx as an input
### October
#### 03
* worked on the implementation of the draft
* currently able to parse/filter/order swap tx coming from a specific dex. By manually testing it, it seems to work fine. And I was actually able to detect some sandwich attacks by the logs I am getting.
* Multi hop support is working fine (In case of A -> B -> C swap, adding A -> B as 1 swap and B -> C as another swap).
* Multi hop swaps are transiting through the Pair first, which is explaining why mev-inspect-py is filtering this case out. 
#### 04
* implemented the approach to detect sandwich attacks in a block
* read and tried to implemented a library with typescript.
* Currently messing with the tsconfig.json to make it work with the library
#### 05
* After few attempts, I was able to make it work but with some workaround command.
* After taking a close look to uniswap v3 sdk, it looks like they are using tsdx to build their library. This thing is working perfectly well.
* Defined sandwich detection logic for a specific address
* Worked on the implementation of the logic which is implying to get all the tx from an address. Interestingly enough, nodes cannot return all the tx from an address. 
* Integrated with Covalent api to get all the tx from an address and will finish the rest of the logic tomorrow