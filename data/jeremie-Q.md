Title:
Use of send function, leading to fail if gas to high.

Proof-of-code:
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L152
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L194
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L211
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L278
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L324
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L424

Recommendation:
Use call function instead:

(bool success, ) = recipient.call{value: amount}("");
require(success, "Transfer failed.");