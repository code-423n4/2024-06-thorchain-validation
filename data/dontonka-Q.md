### [ LOW - 1 ] : transferOut*V5 are not validating native asset amount alignment with parameter 
-----
While `transferOutV5` and `transferOutAndCallV5` calls are done by Bifrost, still ensuring the amount sent in parameter is the same as the native asset seems a good assertion todo, so I would highly recommend to add those. In V4 this can't happen since you literally use `msg.value` as the amount, but here there is room for mistake in that regard.
```diff
  function _transferOutV5(TransferOutData memory transferOutPayload) private {
    if (transferOutPayload.asset == address(0)) {
+     require(msg.value == transferOutPayload.amount)
      bool success = transferOutPayload.to.send(transferOutPayload.amount); // Send ETH.
      if (!success) {
        payable(address(msg.sender)).transfer(transferOutPayload.amount); // For failure, bounce back to vault & continue.
      }
    } else {
      _vaultAllowance[msg.sender][
        transferOutPayload.asset
      ] -= transferOutPayload.amount; // Reduce allowance

      (bool success, bytes memory data) = transferOutPayload.asset.call(
        abi.encodeWithSignature(
          "transfer(address,uint256)",
          transferOutPayload.to,
          transferOutPayload.amount
        )
      );

      require(success && (data.length == 0 || abi.decode(data, (bool))));
    }

    emit TransferOut(
      msg.sender,
      transferOutPayload.to,
      transferOutPayload.asset,
      transferOutPayload.amount,
      transferOutPayload.memo
    );
  }
```  
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L209

```
  function _transferOutAndCallV5(
    TransferOutAndCallData calldata aggregationPayload
  ) private {
    if (aggregationPayload.fromAsset == address(0)) {
+     require(msg.value == aggregationPayload.fromAmount)	
      // call swapOutV5 with ether
      (bool swapOutSuccess, ) = aggregationPayload.target.call{
        value: msg.value
      }(
        abi.encodeWithSignature(
          "swapOutV5(address,uint256,address,address,uint256,bytes,string)",
          aggregationPayload.fromAsset,
          aggregationPayload.fromAmount,
```
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L304


## [ GAS - 1 ] : use external instead of public
-----
You should use `external` here as this is not called internally.
```diff
  function transferOutV5(
    TransferOutData calldata transferOutPayload
- ) public payable nonReentrant {
+ ) external payable nonReentrant {
    _transferOutV5(transferOutPayload);
  }
```
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L242


## [ NC - 1 ] : isVaultTransfer is poluting the code
-----
Everything related to `isVaultTransfer` is not used and poluting the code for no good reason. This flag was related to `vaultTransferEvents` event and used for yggdrasil vaults (single sig vaults) which have been depracted. I would recommend to remove all the related logic to have a cleaner code base.

https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/shared/evm/smartcontract_log_parser.go#L174


## [ NC - 2 ] : don't use magic number for status confirmation
-----
Please use the real constant instead of a magic number. There are `4 instances` of this in total.
```diff
-	if receipt.Status != 1 {
+	if receipt.Status != types.ReceiptStatusSuccessful {
		e.logger.Info().Msgf("tx(%s) state: %d means failed , ignore", tx.Hash().String(), receipt.Status)
		return nil, nil
	}
```	
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go#L788
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go#L886
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/evm/evm_block_scanner.go#L525
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/evm/evm_block_scanner.go#L736


### [ NC - 3 ] : CPU optimization
-----
Low hanging fruit CPU optimization in `isToValidContractAddress` function. You should addr.String() before the loop so you do this only once per function call and not per iteration as currently. This calls everytime `func (a *Address) checksumHex()` which is not that lightweight function. There are 2 instances of this.

```diff
func (e *EVMScanner) isToValidContractAddress(addr *ecommon.Address, includeWhiteList bool) bool {
	if addr == nil {
		return false
	}
	// get the smart contract used by thornode
	contractAddresses := e.pubkeyMgr.GetContracts(e.cfg.ChainID)
	if includeWhiteList {
		contractAddresses = append(contractAddresses, e.whitelistContracts...)
	}
+
+       addrStr := addr.String()
	// combine the whitelist smart contract address
	for _, item := range contractAddresses {
-		if strings.EqualFold(item.String(), addr.String()) {
+		if strings.EqualFold(item.String(), addrStr) {
			return true
		}
	}
	return false
}
```
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/evm/evm_block_scanner.go#L706
https://github.com/code-423n4/2024-06-thorchain/blob/main/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go#L671

