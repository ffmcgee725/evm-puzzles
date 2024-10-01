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

### Puzzle 1

[CALLVALUE](https://www.evm.codes/#34) opcode gets the value of the current call (ie. the transaction value) in wei and pushes that value to the top of the stack.

[JUMP](https://www.evm.codes/#56) opcode consumes the top value on the stack and jumps to the `n`th instruction in the sequence where `n` is the value at the top of the stack.

Value --> `8`
So we insert 8 into stack and jump directly into the 8th instruction to complete the puzzle!

### Puzzle 2

[CODESIZE](https://www.evm.codes/#38) opcode gets the size of the code running in the current environment. So for this case, 10.

[SUB](https://www.evm.codes/#03) opcode takes the first stack element minus the second stack element, pushing the result on the top of the stack.

Value --> `4`
We insert 4, so we get `10-4=6`, and `JUMP` into 6th instruction to complete the puzzle!

### Puzzle 3

[CALLDATASIZE](https://www.evm.codes/#36) opcode gets the size of the calldata in bytes and pushes it onto the stack.

Meaning we need to pass in calldata such that the `CALLDATASIZE` instruction pushes 4 on the stack.
Knowing `0xff` can be used to represent 1 byte since `ff` in hexadecimal evaluates to 255 in decimal format,

Value --> `0xffffffff`
Repeating `ff` four times to get 4 bytes, and that value is pushed to the stack so we can `JUMP` to `JUMPDEST` and complete the puzzle!

### Puzzle 4

[XOR](https://www.evm.codes/#18) opcode evaluates two numbers in their binary representation and returns a `1` in each bit position where the bits of `either, but not both` (ex.: `1100 XOR 0101` evaluates to `1001`)

Now, we now [CALLSIZE] for this puzzle is `12`, or `0c` in hexadecimal, which would be represented as `1100` in binary.

Knowing we want to [JUMP] to the 10th instruction (which would be represented as `1010` in binary)

Value --> `6`
6 is representeed as `110` in binary
`1100 XOR 110` evaluates to `1010`, which is the 10th instruction we want to [JUMP] to and complete the puzzle!

### Puzzle 5

[DUP1](https://www.evm.codes/#80) opcode duplicates the value at the 1st position on the stack and pushes the duplicate to the top of the stack.

Meaning whichever [CALLVALUE] we insert into the stack, will be follow by the same exact value on top of it when [DUP1] is called.

[MUL](https://www.evm.codes/#02) opcode takes the first two values on the stack, multiplies them together and pushes the result onto the top of the stack.

[PUSH2](https://www.evm.codes/#61) opcode pushes a 2 byte value onto the top of the stack.
In our case, we have [PUSH2] `0100` meaning that it will push the 2 byte hex number `0100` (`256`) onto the top of the stack.
Here is how we obtain `256`

```
0×16^3 + 1×16^2 + 0×16^1 + 0×16^0 = 0 + (1 × 256) + 0 + 0 = 256
```

[EQ](https://www.evm.codes/#14) opcode takes the first two values on the stack, runs an equality comparison and pushes the result on the top of the stack. If the first two values are equal, `1` is pushed to the top, otherwise `0` is pushed to the stack instead.

[JUMPI](https://www.evm.codes/#57) opcode will conditionally alter the program counter. It looks at the second stack element to know if it should jump or not, depending on if the second stack element is a `1` or a `0`. Then the first stack element is used to know what position to jump to.

Knowing all this, we conclude that we need to enter a [CALLVALUE] so that when it gets duplicated once (making the first two elements on the stack the [CALLVALUE]), and after the top stack values are multiplied, our result is the hex number `0100`.

If we convert 2 byte hex number `0100` to a decimal, we get `256`, and if we square root that, we get `16` (we square root because `DUP1 MUL` will be multiplying the number by itself).

Value --> `16`

### Puzzle 6
[CALLDATALOAD](https://www.evm.codes/#36) opcode loads calldata from a specified byte. In this puzzle, it reads calldata starting from byte 0 and pushes 32 bytes onto the stack.

We then encounter a JUMP instruction that needs to alter the program counter to `0x0A`.

The calldata provided should allow the JUMP to work. One may think `0x0A` is the right input, but calldata must be padded to 32 bytes.

Value --> `0x000000000000000000000000000000000000000000000000000000000000000A`

### Puzzle 7

[CALLDATACOPY](https://www.evm.codes/#37) opcode copies calldata to memory,

[CREATE](https://www.evm.codes/#f0) opcode deploys a contract.

[EXTCODESIZE](https://www.evm.codes/#3b) opcode expects an address on the top of the stack and returns the size of the code at that address.

The contract must be deployed with 1 byte of code so that [EXTCODESIZE] can return a value of `1`, fulfilling the condition for the jump.

Value --> `0x60016000526001601ff3` (this could be any sequence that returns 1 byte)

### Puzzle 8

This puzzle is similar to puzzle 7, but now it involves using the [CALL] instruction to invoke another contract after deploying one.

The puzzle requires you to provide [CALLDATA] that, when deployed, causes the [CALL] to fail, so that the [EQ] check can pass with a result of `0`.

To make the CALL fail, we deploy a contract with bytecode that has no stack values when executed (causing a [REVERT]).

The bytecode `0x60016000526001601FF3` will deploy a contract whose code is `01` (an [ADD] instruction). Since there are no values to ADD, the [CALL] fails, solving the puzzle.

Value --> `0x60016000526001601FF3` (this could be any sequence that has no stack values when executed, consequently causing a [REVERT])

### Puzzle 9

In this puzzle, both [CALLDATA] and [CALLVALUE] are used. 

The [CALLDATASIZE] must be greater than `3`, and the product of [CALLDATASIZE] * [CALLVALUE] must equal `08`.

[CALLDATA] --> `0x00000001` (which has 4 bytes)
[CALLVALUE] --> `2`

This satisfies both conditions: [CALLDATASIZE] is greater than `3`, and `4 * 2 = 8`, completing the puzzle!
