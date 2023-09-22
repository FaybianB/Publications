# RareSkills Solidity Interview Question #5 Answered: What special CALL is required for proxies to work?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_5.gif)

## *Question #5 (Easy): What special CALL is required for proxies to work?*

**Answer:** `DELEGATECALL` is required for proxy contracts to work because this type of call will preserve the msg (msg.sender, msg.value, etc…) and run in the context of the calling contract instead of the called contract. This enables the proxy contract to call an implementation contract to modify the proxy contract’s state. By using `DELEGATECALL`, the implementation contract can be upgraded without the smart contract system losing any information or having to change the address of the proxy.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

// Proxy Contract
contract Proxy {
    address public implementationAddress;
    // This aligns with the 'data' variable in the Implementation contract
    uint256 public data;

    constructor(address _implementationAddress) {
        implementationAddress = _implementationAddress;
    }

    fallback() external payable {
        address impl = implementationAddress;

        require(impl != address(0));

        assembly {
            let ptr := mload(0x40)

            calldatacopy(ptr, 0, calldatasize())

            let result := delegatecall(gas(), impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()

            returndatacopy(ptr, 0, size)

            switch result
            case 0 {
                revert(ptr, size)
            }
            default {
                return(ptr, size)
            }
        }
    }

    receive() external payable {}
}

// Implementation Contract
contract Implementation {
    // Introduce a storage gap to align with the Proxy's storage layout
    address private _storageGap;

    uint256 public data;

    function setData(uint256 _data) external {
        // If calling this function through the Proxy contract, this will
        // update the Proxy contract's `data` variable.
        data = _data;
    }

    function getData() external view returns (uint256) {
        // Returns the value of Proxy contract's `data` variable.
        return data;
    }
}
```

## Further Discussion:

It is important to only use `DELEGATECALL` when calling trusted contracts, otherwise, the calling contract’s storage variables could be exposed to an attacker using an untrusted contract.

Additionally, it’s crucial to make sure that the storage variables in the implementation contract have the same layout as those in the calling contract. Causing the storage slots to align helps prevent unintentional overwrites in the calling contract.

Medium article: https://coinsbench.com/rareskills-solidity-interview-question-5-answered-what-special-call-is-required-for-proxies-to-84733179fb95