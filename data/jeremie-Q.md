Title:
Use of send function, leading to fail if gas to high.

Proof-of-code:
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L152
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L424

Recommendation:
Use call function instead:

(bool success, ) = recipient.call{value: amount}("");
require(success, "Transfer failed.");


Title:
Costly operations inside a loop

Operations such as state variable updates inside a loop cost a lot of gas, are expensive and may lead to out-of-gas errors. (https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop)

Code:
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L250
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L400
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L415
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L420

Recommendations:
Optimizations using local variables are preferred.


Title:
Missing events

Detect missing events for critical arithmetic parameters.
https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-arithmetic

Code:
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L408-L426

Recommendations:
Add something like this after the line 422: 
emit VaultTransfer(msg.sender, asgard, coins, memo);