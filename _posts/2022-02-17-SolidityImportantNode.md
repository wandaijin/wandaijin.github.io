---
layout: post
title:  "Solidity Big Note"
categories: Solidity
---

> When you call a contract code from another contract, the msg.sender will refer to the address of our contract factory. This is a really important point to understand as using a contract to interact with other contracts is a common practice. You should therefore take care of who is the sender in complex cases.

> There is no “cron” concept in Ethereum to call a function at a particular event automatically.


### 参考资料
1. https://docs.soliditylang.org/en/latest/contracts.html
2. https://ethereum.org/en/developers/tutorials/interact-with-other-contracts-from-solidity/
