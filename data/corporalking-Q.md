# [L-1] Unprotected transferAllowance() Could Lead To Unwanted Event Logs Parsed By Bitfrost
## Description
In the readme, it is said that `transferAllowance()` function in the router contract is expected to be called only by a Thorchain Vault. However, the function is unprotected and thus it can be called by anyone. What is more, it can be made to emit `TransferAllowance` event with the `oldVault` argument being any address. This can potentially be a small isue if the `newVault` argument is a currently active Thorchain vault. 
- Scenario:
  - An attacker deposits some funds using `depositWithExpiry()` where the vault address is some EOA owned by him. The asset is some whitelisted token, e.g. `USDC`. Amount can be any amount != 0.
    - This will set `_vaultAllowance[EOAHackerAddress][USDC] += amount`
  - The attacker then calls `transferAllowance` as the EOA Vault he has the keys to being the msg.sender. The function arguments are the following:
    - Router is the same router address in order to trigger the first if statement
    - newVault is some currently active vault (e.g. `Asgard1`)
    - amount is the previously deposited amount by him
    - this will allow `_adjustAllowances()` to succeed and emit `TransferAllowance` afterwards.
## Recommendation
- Consider protecting the function by a role or implementing whitelist check off-chain (e.g. in `GetTxInItem()` function or elsewhere which disregards the log if `oldVault` is not whitelisted)