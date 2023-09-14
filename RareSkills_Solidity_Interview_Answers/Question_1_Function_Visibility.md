# RareSkills Solidity Interview Question #1 Answered: What is the difference between private, internal, public, and external functions?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_1.gif)

## *Question #1 (Easy): What is the difference between private, internal, public, and external functions?*

**Answer:** Private functions can only be accessed within the contract where they are defined. While internal functions can be accessed by the contract where they are defined and child contracts. Public functions can be accessed both internally, within the contract where they are defined and externally, outside of the contracts where they are defined. External functions can only be accessed outside of the contract where they are defined.

## Demonstration:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.21;

contract ContractA {
    function externalFunction() external {}

    function publicFunction() public {}

    function internalFunction() internal {}

    function privateFunction() private {}
}

contract ContractB {
    ContractA contractA;

    constructor() {
        contractA = new ContractA();
    }

    function callExternalFunction() external {
        contractA.externalFunction();
    }

    function callPublicFunction() external {
        contractA.publicFunction();
    }

    /**
     * TypeError: Member "internalFunction" not found or not visible after
     * argument-dependent lookup in contract ContractA.
     */
    function callInternalFunction() external {
        contractA.internalFunction();
    }

    /**
     * TypeError: Member "privateFunction" not found or not visible after
     * argument-dependent lookup in contract ContractA.
     */
    function callPrivateFunction() external {
        contractA.privateFunction();
    }
}
```

## Further Discussion:

An important tenet of maintaining a good security posture in Solidity is assigning proper access controls. The function visibility modifiers define a function’s level of exposure and developers should use these appropriately to defend against unauthorized access or contract manipulation.

Some considerations when defining a contract’s visibility:

Does the function need to be called by external entities, such as other contracts?
Does the function need to be called internally, for example, by other functions in the same contract or child contracts?

Medium story: https://medium.com/@fbyrd/rareskills-solidity-interview-question-1-answered-ca821f26d943