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
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L422
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L176

Recommendations:
Emit an event


Title:
Protocol doesn't handle ERC20 tokens with decimals other than 18, leading tokens with many decimal values may cause issues due to overflow, while tokens with few decimal values may result in a loss of precision.

Code:
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol

Recommendations:
To mitigate this issue, it is recommended that the decimal values of all tokens be checked, and appropriate handling measures implemented.
Maybe add a function like this:
function getAdjustedAmount(IERC20 token, uint256 amount) internal view returns (uint256) {
        uint8 decimals = token.decimals();
        return amount * (10 ** (18 - decimals));
}