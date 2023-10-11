# RareSkills Solidity Interview Question #10 Answered: Which is better to use for an address allowlist: a mapping or an array? Why?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_10.gif)

## *Question #10 (Easy): Which is better to use for an address allowlist: a mapping or an array? Why?*

**Answer:** A mapping is better for an address allowlist because it is more gas efficient. With a mapping, it is possible to check if an address is on the allowlist by directly accessing its value. Using an array, verifying an address could be costly because it would require looping through each element.

## Demonstration:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

contract AllowlistMapping {
    mapping(address => bool) public allowlist;

    address owner = msg.sender;

    function addToAllowlist(address addr) external {
        require(msg.sender == owner, "Only owner can modify allowlist");

        allowlist[addr] = true;
    }

    function removeFromAllowlist(address addr) external {
        require(msg.sender == owner, "Only owner can modify allowlist");

        delete allowlist[addr];
    }

    function isAllowed(address addr) external view returns(bool) {
        return allowlist[addr];
    }
}

contract AllowlistArray {
    address[] public allowlist;

    address owner = msg.sender;

    // Add an address to the allowlist
    function addToAllowlist(address _address) public {
        require(msg.sender == owner, "Only owner can modify allowlist");

        // Check if the address is already in the allowlist
        for (uint i = 0; i < allowlist.length; i++) {
            if (allowlist[i] == _address) {
                revert("Address already in the allowlist");
            }
        }

        allowlist.push(_address);
    }

    // Remove an address from the allowlist
    function removeFromAllowlist(address _address) public {
        require(msg.sender == owner, "Only owner can modify allowlist");

        uint index = 0;
        bool found = false;

        // Find the index of the address to be removed
        for (uint i = 0; i < allowlist.length; i++) {
            if (allowlist[i] == _address) {
                index = i;
                found = true;
                break;
            }
        }

        // If the address was found, remove it from the allowlist
        if (found) {
            // Move the last element to the index to delete
            allowlist[index] = allowlist[allowlist.length - 1];
            // Remove the last element
            allowlist.pop();
        } else {
            revert("Address not found in the allowlist");
        }
    }

    // Check if an address is in the allowlist
    function isAllowed(address _address) public view returns (bool) {
        for (uint i = 0; i < allowlist.length; i++) {
            if (allowlist[i] == _address) {
                return true;
            }
        }
        return false;
    }
}
```

## Further Discussion

Managing an allowlist is generally more straightforward and gas efficient using a mapping versus an array. The gas cost of managing an array increases as the array grows. Thus, as allowlists often hold thousands of addresses, this design choice could result in hefty gas fees.

Medium article: https://medium.com/@fbyrd/rareskills-solidity-interview-question-10-answered-which-is-better-to-use-for-an-address-848561a00215