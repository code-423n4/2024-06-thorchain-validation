## No Zero Address checks for some functions

Found for vault `depositWithExpiry` function parameter.
Found for router, newVault and asset `transferAllowance` function parameters.
Found for to `transferOut` function parameter.
Found for TransferOutData.to in `transferOutV5` and `batchTransferOutV5` functions.
Found for aggregator, finalToken and to `transferOutAndCall` function parameters.
Found for TransferOutData.target, TransferOutData.toAsset and TransferOutData.recipient in `transferOutAndCallV5` and `batchTransferOutAndCallV5` functions.

## Zero Amount checks for some functions

Found for amount `depositWithExpiry` function parameter.
Found for amount `transferAllowance` function parameter.
Found for amount `transferOut` function parameter.
Found for TransferOutData.amount in `transferOutV5` and `batchTransferOutV5` functions.
Found for amountOutMin `transferOutAndCall` function parameter.
Found for TransferOutData.fromAmount in `transferOutAndCallV5` and `batchTransferOutAndCallV5` functions.