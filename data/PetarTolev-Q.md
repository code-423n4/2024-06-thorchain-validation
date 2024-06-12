### [L-01] Missing token address existence check on the all ERC20 calls

Current ERC20 transfer calls will succeed if the `asset` address isn't a smart contract. This happens because all low-level calls to addresses with zero code return `success == true`. If the transfer is executed to a non-existent address, the low-level call will return success despite no tokens being transferred. This could potentially lead to the emission of a false event, which is crucial for the proper functioning of the ThorChain network.

### [L-02] Funds can be locked due to the absence of a check in the transfer functions

Certain functions allow the option to deposit/transfer both native ether and ERC20 tokens, without implementing a check to ensure that only a single option is utilized. This oversight can result in locked native tokens in the `Router` contract.

The `_deposit()` function, called by the payable `depositWithExpiry()` function, has a [requirement](https://github.com/code-423n4/2024-06-thorchain/blob/e3fd3c75ff994dce50d6eb66eb290d467bd494f5/chain/ethereum/contracts/THORChain_Router.sol#L155) that addresses this issue. However, many other payable functions do not include this check.

These functions do not check if the `msg.value == 0` when ERC20 token is used:

- [`transferOut()`](https://github.com/code-423n4/2024-06-thorchain/blob/e3fd3c75ff994dce50d6eb66eb290d467bd494f5/chain/ethereum/contracts/THORChain_Router.sol#L185)
- [`_transferOutV5()`](https://github.com/code-423n4/2024-06-thorchain/blob/e3fd3c75ff994dce50d6eb66eb290d467bd494f5/chain/ethereum/contracts/THORChain_Router.sol#L209) function, called by two payable functions: `transferOutV5()` and `batchTransferOutV5()`
- [`_transferOutAndCallV5()`](https://github.com/code-423n4/2024-06-thorchain/blob/e3fd3c75ff994dce50d6eb66eb290d467bd494f5/chain/ethereum/contracts/THORChain_Router.sol#L304) function, called by two payable functions: `transferOutAndCallV5()` and `batchTransferOutAndCallV5()`

As a recommendation, a check similar to the one in the [`_deposit()`](https://github.com/code-423n4/2024-06-thorchain/blob/e3fd3c75ff994dce50d6eb66eb290d467bd494f5/chain/ethereum/contracts/THORChain_Router.sol#L155) function should be implemented.

```diff
require(msg.value == 0, "unexpected eth")
```

### [L-03] **Did not Approve to zero first**

Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value. For example Tether (USDT)'s `approve()` function will revert if the current approval is not zero, to protect against front-running changes of approvals.
The `router.safeApprove` function sets the new amount without checking for ongoing approvals and without first resetting the approval to zero.

```solidity
  function safeApprove(
    address _asset,
    address _address,
    uint _amount
  ) internal {
    (bool success, ) = _asset.call(
      abi.encodeWithSignature("approve(address,uint256)", _address, _amount)
    ); // Approve to transfer
    require(success);
  }
```

It is recommended to set the allowance to zero before increasing the allowance

### [L-04] Some function missing option to provide deadline parameter

The `transferOutAndCall()` and `_transferOutAndCallV5()` functions are designed to execute swaps on Uniswap, as evidenced by the implementation of the `THORChain_Aggregator.swapOutV5()` and `THORChain_Aggregator.swapOut()` functions. However, these functions currently lack an option to set a deadline parameter, which is typically used when executing a swap on Uniswap. Instead, the `THORChain_Aggregator` uses `type(uint256).max` as a deadline, which is not a best practice.

```solidity
  function swapOut(
    address token,
    address to,
    uint256 amountOutMin
  ) public payable nonReentrant {
    address[] memory path = new address[](2);
    path[0] = WETH;
    path[1] = token;
    swapRouter.swapExactETHForTokens{value: msg.value}(
      amountOutMin,
      path,
      to,
@>    type(uint).max
    );
  }
```

To avoid delayed swap execution that could result in a disadvantageous swap, it is recommended to allow the caller to set their own deadline parameter.