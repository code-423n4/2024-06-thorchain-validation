#[L-01] Lack of 0 amount check
Some ERC20 tokens like LEND revert on transfer of 0 amount
https://github.com/code-423n4/2024-06-thorchain/blob/733dbe7cd7eef0dffc5e8a2d02e36bf74b196eff/chain/ethereum/contracts/THORChain_Router.sol#L134
https://github.com/code-423n4/2024-06-thorchain/blob/733dbe7cd7eef0dffc5e8a2d02e36bf74b196eff/chain/ethereum/contracts/THORChain_Router.sol#L188
## Recommendation 
Add 
``` diff
+  require(amount >0, "cannot transfer 0 amount");
```

#[L-02] Revert on 0 value approval check
Some ERC20 tokens like BNB revert on Approving of 0 amount
https://github.com/code-423n4/2024-06-thorchain/blob/733dbe7cd7eef0dffc5e8a2d02e36bf74b196eff/chain/ethereum/contracts/THORChain_Router.sol#L173
## Recommendation 
Add 
``` diff
+  require(amount >0, "cannot approve 0 amount");
```
