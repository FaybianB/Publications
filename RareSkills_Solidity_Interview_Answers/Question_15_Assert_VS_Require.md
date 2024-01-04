# RareSkills Solidity Interview Question #15 Answered: What is the difference between assert and require?

This series will provide answers to the list of [Solidity interview questions](https://www.rareskills.io/post/solidity-interview-questions) that were published by [RareSkills.](https://www.rareskills.io/).

![Alt text](media/Question_15.png)

## *Question #15 (Easy): What is the difference between assert and require?*

**Answer:** Both `assert` and `require` are used for error handling, but they behave differently. For example, if a `require` statement fails, it will revert the transaction and refund the remaining gas to the caller, whereas, `assert` will revert the transaction but will NOT refund the gas. Also, if the condition fails, `require` allows for an optional error message that could be returned to the user, but `assert` does not.

## Demonstration:

```
Ether denominations:

    - 1 Wei = 10^-18 Ether
    - 1 Gwei = 10^-9 Ether
    - 1 Szabo = 10^-6 Ether
    - 1 Finney = 10^-3 Ether
```

## Further Discussion:

1 wei is the smallest unit of Ether.

Medium article: https://coinsbench.com/rareskills-solidity-interview-question-14-answered-how-much-is-1-wei-of-ether-9ba9a5b39d42