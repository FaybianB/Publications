# Understanding Solady's Fixed Point Multiplication and Division Operations

As of the latest version at the time of writing (0.8.22), Solidity does not support decimals. So then how do we, for example, denote fractional amounts in Solidity? Or perform accurate percentage calculations?

Fixed-point numbers.

## Fixed-Point Numbers

Fixed-point numbers are a way to represent numbers that have a fixed number of digits after the decimal point. For example, to denote `0.05` in Solidity, we need to scale the number up by multiplying it by a scaling integer (`10` raised to the power of a scale) so that it can be represented as an integer. For this example, let's use a scale of `2`, which is the number of decimals places to support. The resulting scaling integer is `100` (`10^2`) and the integer representation of `.05` is `5` (`0.05 * 100`). `5` would now be considered a fixed-point number because it represents a decimal with two places.

### Precision

Continuing with the example scale of `2`, the most precise number that can be denoted is `0.01` because it is the smallest number that can be represented by two decimal places. If  smaller numbers are desired then increased precision can be achieved by increasing the scale. For example, the smallest unit on a scale of `3` would be `0.001`... on a scale of `4` would be `0.0001`, etc...

It is important for applications to choose a scale that best suites their use case to mitigate against precision loss.

## Solady

Another option is to use a library that features it's own scale and precisely handles fixed-point arithmetic. One popular arithmetic library with operations for fixed-point numbers is the [Solady Fixed-Point Math Library](https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol). It features functions to multiply, divide, raise to a power, find the exponential, take the natural logarithm, square root, and cube root of fixed-point numbers using a hardcoded scale of `18`, as well as other numerical utilities and operations.

Why `18`? Well, `18` is the scale that is used by ETH and many ERC20 tokens so it is convenient for calculations regarding token balances.

The library refers to a value of `1e18` as a `WAD`.

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L49-L50

So `1 WAD = 1 x 10^18` of whatever unit is being used (tokens, wei, etc...). This enables the functions to perform math with a precision of up to `18` decimal places.

Now let's take a look at a few of these functions in-depth...

### Fixed-Point Multiplication

Note: All of the functions below assume that the parameters `x` and `y` are already scaled up by `WAD` and are fixed-point numbers.

#### mulWad

First, the `mulWad` function...

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L56-L67

Per the dev comment, this function is equivalent to `(x * y) / WAD` rounded down.

The function immediately opens a Yul assembly block and performs this check:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L60-L61

The comment says this calculation is equivalent to `require(y == 0 || x <= type(uint256).max / y)`. Basically, if `x` is such that `x * y` would overflow, the check `mul(y, gt(x, div(not(0), y)))` will result in `y`, triggering the overflow custom error:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L60-L61

If `x` is safe to multiply with `y` without causing an overflow, the result will be `0`, and the multiplication can proceed safely:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L65

Finally, to get the result, the function multiplies `x` and `y` then divides by `WAD`.

If `x` and `y` are both fixed-point values with `18` decimal places, then `x * y` would give a result with `36` decimal places. Dividing by `WAD` scales the result down to `18` decimal places, ensuring the result is correctly remains within the `WAD` fixed-point format.

##### Example Use Case

The `mulWad` function is particularly useful in scenarios dealing with token amounts that are represented in fixed-point notation, such as ERC20 tokens that follow the `18` decimal scale.

Consider this token sale contract where users can buy tokens at a fixed rate. The rate is defined in terms of how much token they get per ether sent. The `mulWad` function can be used to calculate the correct amount of tokens to send to the user, based on the ether they sent and the fixed rate.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.22;

import "./FixedPointMathLib.sol";
import "./IERC20.sol";

contract TokenSale {
    using FixedPointMathLib for uint256;

    // The rate is defined as token amount per 1 ETH.
    // Represented as a fixed-point number with 18 decimal places.
    // 1000 tokens per 1 ETH
    uint256 public constant RATE = 1000e18;

    // Let's assume we have a token that follows the ERC20 standard.
    IERC20 public immutable saleToken;

    constructor(IERC20 _saleToken) {
        saleToken = _saleToken;
    }

    // Function to buy tokens with ETH.
    function buyTokens() external payable {
        // Calculate the number of tokens to send to the user.
        // Uses the mulWad function to handle the fixed-point multiplication.
        uint256 tokensToBuy = RATE.mulWad(msg.value);

        // Transfer the tokens to the msg.sender.
        require(saleToken.transfer(msg.sender, tokensToBuy), "Token transfer failed");
    }

    // Function to allow the owner to withdraw ETH from the contract.
    function withdrawETH() external {
        // Only the owner can withdraw ETH...
        // Transfer ETH to the owner, etc...
    }
}
```

#### mulWadUp

The `mulWadUp` function is similar to `mulWad`, except that it rounds the result up instead of down, as `mulWad` does.

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L69-L80

Every line of the functions is the same except how the return value is calculated:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L78

This line essentially rounds up the result of the multiplication `x * y` when divided by `WAD`.

##### Example Use Case

Applications may want to use `mulWadUp` instead of `mulWad` if rounding up is important. For example, a DeFi application that deals with loan repayments can use `mulWadUp` for interest calculation to ensure that the total interest paid over time fully compensates the lender without accruing fractional debt that could be lost due to rounding down.

#### divWad

The `divWad` function is used for fixed-point division.

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L82-L93

It also features an immediate Yul assembly block with a check:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L86-L87

This checks if the multiplication result is `0`. If it is, then either `y` was `0` (division by zero error) or the multiplication of `x` by `WAD` would cause an overflow. If the result is `0`, this expression evaluates to `true` (or `1`), and the condition for the if statement is met, leading to a revert.

Lastly, the calculation for the result:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L91

1) `mul(x, WAD)`: Multiplies the dividend x by the scaling factor `WAD`.

2) `div(mul(x, WAD), y)`: Divides the product of `mul(x, WAD)` by the divisor `y`.

##### Example Use Case

Suppose a DeFi platform offers a staking service where users can stake their tokens to earn rewards. The platform sets an annual interest rate for the reward, which is calculated and distributed daily.

In this scenario, `divWad` can be used to help determine the daily interest rate per token by dividing the annual interest rate by the number of days in a year.

The calculation would look something like this:

```solidity
// Constants
uint256 annualInterestRate = 5e16; // 5% interest rate, scaled to 18 decimal places
uint256 totalStakedTokens = 1e21; // 1000 tokens, scaled to 18 decimal places
uint256 daysInYear = 365;

// Daily interest calculation per token
uint256 dailyInterestPerToken = divWad(annualInterestRate, daysInYear);

// Suppose a user has staked 100 tokens
uint256 userStakedTokens = 1e20; // 100 tokens, scaled to 18 decimal places

// User's daily reward calculation
uint256 usersDailyReward = mulWad(userStakedTokens, dailyInterestPerToken);
```

#### divWadUp

The `divWadUp` function is similar to `divWad`, except that it rounds the result up instead of down, as `divWad` does.

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L95-L106

These functions are the same except the return value calculation:

https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol#L104

The `add` function takes two arguments, the result of the double negation `iszero` checks and the result of the division `div(mul(x, WAD), y)`. If rounding up is necessary, `1` is added to the division result, otherwise `0` is added.

##### Example Use Case

Applications may want to use `divWadUp` instead of `divWad` to ensure that the result of the division is rounded up. For example, this function would be useful where rounding down could unfairly benefit the application at the user's expense.