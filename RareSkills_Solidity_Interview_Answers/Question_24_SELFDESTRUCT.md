# RareSkills Solidity Interview Question #24 Answered: How do you send Ether to a contract that does not have payable functions, or a receive or fallback?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_24.png)

## *Question #24 (Easy): How do you send Ether to a contract that does not have payable functions, or a receive or fallback?*

**Answer:** To send Ether to a contract that does not have payable functions, or a `receive` or `fallback` you would provide its address as a parameter to a `selfdestruct` call from another contract that has Ether. The `selfdestruct` function forces Ether to be sent to a contract, even if the contract does not have any functions to receive Ether. The contract that calls `selfdestruct` will have all of its Ether balance transferred to the specified address.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

/**
 * This contract does NOT have a payable, receive or fallback function,
 * so it cannot receive Ether through regular transactions.
 */
contract UnwillingRecipient {
    function getBalance() external view returns (uint) {
        return address(this).balance;
    }
}

contract ForceEtherSender {
    // Constructor to optionally receive Ether upon deployment
    constructor() payable {}

    // Function to allow the contract to receive Ether
    receive() external payable {}

    // Function to self-destruct and force-send Ether to an address
    function forceSend(address payable recipient) external {
        // Requires that the contract has a balance greater than 0
        require(address(this).balance > 0, "No Ether to send");

        // selfdestruct sends all Ether held by the contract to the recipient
        selfdestruct(recipient);
    }
}
```

## Further Discussion:

Forcefully sending Ether via selfdestruct can been used as an attack mechanism on contracts that contain logic that are dependent on contract balances. It is important to mitigate this vulnerability by avoiding patterns where the exact amount of Ether held by the contract is critical to the contract’s security or logic. It is also good practice to use a separate mechanism to track deposits or expected balances within the contract rather than relying on address(this).balance directly for logic decisions.

Note: In the latest Solidity version 0.8.24, the `SELFDESTRUCT` opcode is deprecated but it can still be used to force Ether into a contract.

Medium article: https://medium.com/@fbyrd/rareskills-solidity-interview-question-24-answered-how-do-you-send-ether-to-a-contract-that-does-f90818290d46