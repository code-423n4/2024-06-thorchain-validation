### [L-1] `expiration` is set to `type(uint).max` in `THORChain_Router::_routerDeposit` which makes validation of `expiration` in `THORChain_Router::depositWithExpiry` redundant


### [L-2] Use `block.number` instead of `block.timestamp` in `THORChain_Router::depositWithExpiry`



### [L-3] `THORChain_Router::depositWithExpiry` should be `nonReentrant` instead of `THORChain_Router::_deposit`