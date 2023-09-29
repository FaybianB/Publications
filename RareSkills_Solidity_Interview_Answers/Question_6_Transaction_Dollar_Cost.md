# RareSkills Solidity Interview Question #6 Answered: Prior to EIP-1559, how do you calculate the dollar cost of an Ethereum transaction?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_6.gif)

## Question #6 (Easy): Prior to EIP-1559, how do you calculate the dollar cost of an Ethereum transaction?

**Answer:** Prior to EIP-1559, the dollar cost calculation of an Ethereum transaction was `((GAS USED * GAS PRICE / 10^-9) * CURRENT ETHER PRICE`. In this era, the miner received 100% of the gas cost. After EIP-1559, the dollar cost calculation of an Ethereum transaction is `(((BASEFEE + PRIORITY FEE) * GAS USED)) / 10^-9) * CURRENT ETHER PRICE`. Now, the miner receives a portion of the gas cost (`PRIORITY FEE`) and the other portion, the protocol fee (`BASEFEE`), is burned.

## Demonstration:

<ins>GAS USED</ins>: The amount of computational work needed to process a particular transaction or smart contract on the Ethereum network. It is essentially the unit that measures how much “work” is done.
<ins>GAS PRICE</ins>: Before EIP-1559, this was the cost per unit of gas specified by the user willing to pay for a transaction. It is usually denoted in Gwei.
<ins>BASEFEE</ins>: Introduced by EIP-1559, this is a variable fee per gas that aims to make the fee market more predictable. It is adjusted by the network based on overall demand for block space. This part of the transaction fee is burned.
<ins>PRIORITY FEE</ins>: Also known as the “miner tip,” is an optional fee added by the user on top of the `BASEFEE` to prioritize their transaction for faster processing by the miners. Unlike `BASEFEE`, this fee goes to the miner.
<ins>CURRENT ETHER PRICE</ins>: The market value of one Ether (ETH) at any given time, usually expressed in U.S. dollars or other fiat currencies.

## Further Discussion:

The purpose of introducing the `BASEFEE` was to make transaction fees more predictable on the Ethereum network. This fee is automatically adjusted based on network conditions. Higher network congestion leads to a higher `BASEFEE`. Lower network congestion results in a lower `BASEFEE`.

Senders can still incentivize miners to prioritize their transactions by sending a miner’s tip (`PRIORITY FEE`). In times of high network activity there is more competition for block space and transactions with the highest `PRIORITY FEE` are likely to be prioritized by miners. This bidding system resembles how the gas market operated prior to EIP-1559.

Medium story: https://medium.com/coinsbench/rareskills-solidity-interview-question-6-answered-prior-to-eip-1559-how-do-you-calculate-the-96133d9f33e8