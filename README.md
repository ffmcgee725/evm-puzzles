# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

[`repo`](https://github.com/fvictorio/evm-puzzles)

## Solution

### Puzzle 4
Value --> 6
6 evaluates to 110 in binary
CODESIZE is 12, which is 1100 in binary
XOR of 110 with 1100 --> 1010 in binary, which translates to 10 (0A)
