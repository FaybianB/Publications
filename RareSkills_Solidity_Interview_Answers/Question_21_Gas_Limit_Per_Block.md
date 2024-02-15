# RareSkills Solidity Interview Question #21 Answered: As of the Shanghai upgrade, what is the gas limit per block?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_21.png)

## *Question #21 (Easy): As of the Shanghai upgrade, what is the gas limit per block?*

**Answer:** As of the Shanghai upgrade, the gas limit per block is 30 million. This means that the total amount of gas spent on all transactions in a block must be less than 30 million. This is how the number of transactions per block is limited.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

contract GasLimitExample {
    uint256 public counter = 0;

    /**
     * This function's gas usage could hit the block limit of 30 million gas,
     * if it is called with a high enough "_count" and enough gas is provided
     * for execution.
     */
    function loopIncrement(uint256 _count) public {
        for (uint256 i = 0; i < _count; i++) {
            counter += 1;
        }
    }
}
```

## Further Discussion:

Although the gas limit per block is 30 million, the gas target per block is 15 million.

Medium article: https://medium.com/coinmonks/rareskills-solidity-interview-question-21-answered-as-of-the-shanghai-upgrade-what-is-the-gas-4a937cda5b2c