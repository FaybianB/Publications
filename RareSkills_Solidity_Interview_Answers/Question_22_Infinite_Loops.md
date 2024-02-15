# RareSkills Solidity Interview Question #22 Answered: What prevents infinite loops from running forever?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_22.png)

## *Question #22 (Easy): What prevents infinite loops from running forever?*

**Answer:** The transaction gas limit prevents infinite loops from running forever. When a transaction is submitted, the user specifies a gas limit, which is the maximum amount of gas the user is willing to use and pay to execute the transaction. Once the gas consumed by a transaction reaches its gas limit, the execution stops and the transaction reverts (typically with an “Out of gas” error).

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

contract GasLimitExample {
    uint256 public counter = 0;

    /**
     * This function's gas usage could hit the transaction's gas limit,
     * if it is called with a high enough "_count".
     */
    function loopIncrement(uint256 _count) public {
        for (uint256 i = 0; i < _count; i++) {
            counter += 1;
        }
    }
}
```

## Further Discussion:

The [gas limit per block](https://medium.com/coinmonks/rareskills-solidity-interview-question-21-answered-as-of-the-shanghai-upgrade-what-is-the-gas-4a937cda5b2c) would also indirectly prevent infinite loops from running forever since a user cannot set a transaction gas limit that exceeds the gas limit per block. In this scenario, the network would reject the transaction since it could consume too much gas to be included in any block.

This serves as a network protection mechanism by ensuring that no block can consume an excessive amount of computational resources.

Medium article: https://medium.com/@fbyrd/rareskills-solidity-interview-question-22-answered-what-prevents-infinite-loops-from-running-f76574fad9b4