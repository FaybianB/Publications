# RareSkills Solidity Interview Question #17 Answered: What is the check-effects pattern?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_17.png)

## *Question #17 (Easy): What is the check-effects pattern?*

**Answer:** The *check-effects* pattern (also known as *check-effects-interactions*) is a Solidity smart contract design framework that defines the order of operations within a function. The *check* is the first operation, where the method validates the parameters and/or state to ensure that the passed in data is acceptable and the contract is in the correct state to execute the function. Then, the *effects* operation, where the core functionality of the method follows and the contract state can be updated.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract ChecksEffectsInteractions {

    mapping(address => uint) balances;

    function deposit() public payable {
        balances[msg.sender] = msg.value;
    }

    function withdraw(uint amount) public {
        // Check
        require(balances[msg.sender] >= amount);

        // Effects
        balances[msg.sender] -= amount;

        // Interactions
        (bool sent, bytes memory data) = msg.sender.call{value: amount}("");
    }
}
```

## Further Discussion:

The *interactions* operation, following *effects*, in the *check-effects-interactions* pattern is an important security measure for defending against certain types of attacks, especially reentrancy. In this phase, the contract interacts with external contracts or addresses, such as transferring Ether or calling external functions.

The purpose of the pattern is to prevent external contracts from taking control of the execution flow before the original contract’s state is fully updated.

Medium article: https://medium.com/coinmonks/rareskills-solidity-interview-question-17-answered-what-is-the-check-effects-pattern-975e262eb616