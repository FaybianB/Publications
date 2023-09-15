# RareSkills Solidity Interview Question #3 Answered: What is the difference between CREATE and CREATE2?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_3.gif)

## *Question #3 (Easy): What is the difference between create and create2?*

**Answer:** The `CREATE` and `CREATE2` opcodes differ in the parameters used to calculate the new contract’s address. The `CREATE` opcode computes the new address by taking the keccak256 hash of the sender’s address and a nonce. The `CREATE2` opcode computes the new address by taking the keccak256 hash of the constant `0xFF` (to avoid collision with the `CREATE` opcode), the sender’s address, a salt (arbitrary value provided by the sender) and the new contract’s init code. Since `CREATE2` uses a user defined parameter in it’s calculation, the new contract’s address is able to be (more reliably and flexibly) predetermined.

## Demonstration:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.21;

contract ContractToBeCreated {}

contract Factory {
    // Arbitrary value used to calculate new contract address
    bytes32 constant private SALT = "random salt value";

    /**
     * @return The predicted address of the new contract
     */
    function getPredictedNewContractAddressUsingCreateFormula()
      external
      view
      returns (address)
    {
        // This involves Recursive Length Prefix (RLP) encoding, which is
        // outside the scope of this demonstration...
        // This can be accomplished using Javascript libraries such as rlp.js
        // and web3.js...
    }

    /**
     * @notice Creates a new contract using the CREATE opcode
     */
    function create() external returns (address deployedContractAddress) {
        bytes memory bytecode = type(ContractToBeCreated).creationCode;

        assembly {
            deployedContractAddress := create(
                // wei sent with current call
                0,
                // Actual code starts after skipping the first 32 bytes
                add(bytecode, 0x20),
                // Load the size of code contained in the first 32 bytes
                mload(bytecode)
            )
        }
    }

    /**
     * @return The predicted address of the new contract
     */
    function getPredictedNewContractAddressUsingCreate2Formula()
      external
      view
      returns (address)
    {
        bytes1 constantValue = bytes1(0xff);
        address sendersAddress = address(this);
        bytes memory bytecode = type(ContractToBeCreated).creationCode;
        bytes32 hash = keccak256(
          abi.encodePacked(
            constantValue,
            sendersAddress,
            SALT,
            keccak256(bytecode)
          )
        );

        return address(uint160(uint256(hash)));
    }

    /**
     * @notice Creates a new contract using the CREATE2 opcode
     */
    function create2() external returns (address deployedContractAddress) {
        bytes memory bytecode = type(ContractToBeCreated).creationCode;

        assembly {
            deployedContractAddress := create2(
                // wei sent with current call
                0,
                // Actual code starts after skipping the first 32 bytes
                add(bytecode, 0x20),
                // Load the size of code contained in the first 32 bytes
                mload(bytecode),
                // Developer-specified salt value
                SALT
            )
        }
    }
}
```

## Further Discussion:

It is possible to predict the new contract’s address with the `CREATE` opcode’s formula, however, this is subject to the condition that the nonce doesn’t change between the time the address is calculated and the time the contract is actually created. This predicted address is not reliable in scenarios where the contract is to be deployed in a future transaction, such as in counterfactual systems.

The `CREATE2` formula gives developers the flexibility to choose a desirable address simply by adjusting the salt value or even the bytecode of the contract to be deployed. Theoretically, searching for a desirable address is also possible with the `CREATE` formula, as developers could calculate the new address, submit a transaction that would increase the sender’s nonce value and then recalculate the address. Practically, however, this would be costly to execute.

Medium story: https://medium.com/coinsbench/rareskills-solidity-interview-question-3-answered-c94111942bfd