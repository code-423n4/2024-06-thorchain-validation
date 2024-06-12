# 1
## Impact
Zero amount is not checked for the `_deposit` function which can lead to a lot of unnecessary transactions to load the block_scanner transactions processing

## Proof of Concept
This can be seen here https://github.com/code-423n4/2024-06-thorchain/blob/main/chain/ethereum/contracts/THORChain_Router.sol#L143-L160

## Recommended Mitigation Steps
add check for zero value and amount on deposit


# 2
## Impact
expiry is better put after the _deposit function rather than before to ensure the expiry is run after all transaction are done or revert the deposit depositWithExpiry function transaction

## Proof of Concept
This can be seen here https://github.com/code-423n4/2024-06-thorchain/blob/main/chain/ethereum/contracts/THORChain_Router.sol#L131

## Recommended Mitigation Steps
put it after the _deposit function call in the depositWithExpiry function