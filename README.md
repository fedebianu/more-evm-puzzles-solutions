# More EVM Puzzles Solutions

My solutions to Dalton Sweeney's [10 more EVM puzzles](https://github.com/daltyboy11/more-evm-puzzles) (June 2023), the sequel to [EVM puzzles](https://github.com/fvictorio/evm-puzzles).
Each puzzle is a contract's bytecode: find the call value and/or calldata that make it run to the end without reverting.

The answers are in [`solutions/`](solutions), and all 10 pass when replayed against the puzzles in order.

| # | What the bytecode does | Answer | Why |
|---|---|---|---|
| 1 | `JUMP` to `CALLVALUE ** CALLDATASIZE`, then to `PC + CALLDATASIZE` | value 2, 6 bytes of data | 2⁶ = 0x40, the first `JUMPDEST`; then 0x41 + 6 = 0x47, the second |
| 2 | `CREATE`s a contract from the calldata, `CALL`s it, and requires 10 bytes of return data | init code + runtime `PUSH1 0x0a PUSH1 0 RETURN` | The runtime returns 10 bytes of memory |
| 3 | `CREATE`s a contract from the calldata, `DELEGATECALL`s it, and requires slot 5 to hold 0xaa | init code + runtime `PUSH1 0xaa PUSH1 5 SSTORE` | Through `DELEGATECALL` the runtime writes the puzzle's own storage |
| 4 | `CREATE`s a contract from the calldata with the puzzle's whole balance, and requires the new contract to hold exactly half of it | value 2, and a constructor that sends half of `CALLVALUE` to address 0 | The constructor burns half of what it receives, so the ratio is 2 |
| 5 | Requires more than 32 bytes of calldata and, once it is copied to memory, `MSIZE - CALLDATASIZE == 3` | 61 bytes of data | Memory grows in 32-byte words: 61 bytes take 64, and 64 − 61 = 3 |
| 6 | Requires `0xff…f0 + CALLVALUE == 1` | value 17 | The addition wraps around: 2²⁵⁶ − 16 + 17 = 1 |
| 7 | A loop that runs `CALLVALUE` times, and the gas spent must be 0xa6 = 166 | value 4 | Each iteration costs 43 gas, and the last one skips the jump back: 43x − 6 = 166 |
| 8 | `CREATE`s a contract from the calldata, sends it the puzzle's whole balance, and requires the balance to be unchanged | init code + runtime `CALLER SELFDESTRUCT` | `SELFDESTRUCT` sends the ETH back without executing the puzzle's code, which a `CALL` would re-enter |
| 9 | Requires the first byte of `keccak256(CALLVALUE)` to be 0xa8 | value 47 | Brute force: 47 is the smallest value whose hash starts with 0xa8 |
| 10 | Requires `(word1 AND 0xf0f0…) OR word2 == 0xabab…` on the first two calldata words | `0xabab…` followed by `0x0b0b…` | The mask keeps the high nibbles (0xa0), the second word supplies the low ones (0x0b) |

## Playing

```bash
yarn install
npx hardhat play
```

The game skips every puzzle that already has a file in `solutions/`: delete them to play from the start.
