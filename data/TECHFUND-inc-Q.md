### 1) THORChain_Router::batchTransferOutV5()
batchTransferOutV5() function should add a validation to check if the msg.value covers the sum of amount for all entries in the batch where asset = 0 address, meaning ether. 