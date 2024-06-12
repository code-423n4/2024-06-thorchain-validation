
## [L-1] Missing msg.value Check in ETH Deposit in \_deposit function

### Description:

The `THORChain_Router::_deposit` function is missing a check to ensure that `msg.value` is greater than `0` when depositing `ETH`. Without this check, a user can call the function with `msg.value` equal to `0`, potentially causing unexpected behavior.

```javascript
if (asset == address(0)) {
safeAmount = msg.value;
bool success = vault.send(safeAmount);
require(success);
}
```

### Impact:

Users might unintentionally call the function with `msg.value` set to 0, leading to confusion and potential misuse of the contract.

### Proof of Concept:

copy and past it in 1_Router.js contract under.

```javascript
it("should pass even user passes 0 Eth", async function () {
  await ROUTER1.depositWithExpiry(
    YGGDRASIL1,
    ETH,
    "0",
    "SEED ETH",
    currentTime,
    { from: accounts[10], value: 0 }
  );
});

Logs:  Router contract
    Upgrade contract
      ✔ should pass even user passes 0 Eth


  1 passing (3s)
```

### Recommended Mitigation:

Modify the code to include a check for `msg.value`:

```diff
if (asset == address(0)) {
+ require(msg.value > 0, "No ETH sent");
safeAmount = msg.value;
bool success = vault.send(safeAmount);
require(success);
}
```

## [L-2] Declaration Order Not Following Checks-Effects-Interactions Pattern

### Description:

The `THORChain_Router::_vaultAllowance` update should be done before calling `safeTransferFrom()` to adhere to the Checks-Effects-Interactions pattern, reducing the risk of reentrancy attacks.

```javascript
safeAmount = safeTransferFrom(asset, amount); // Transfer asset
_vaultAllowance[vault][asset] += safeAmount; // Credit to chosen vault
```

### Impact:

Although the function is marked nonReentrant, adhering to the Checks-Effects-Interactions pattern is a good practice to avoid potential reentrancy issues in future modifications.

### Proof of Concept:

Move the `THORChain_Router::_vaultAllowance` update before the `safeTransferFrom()` call. // to-do

### Recommended Mitigation:

Update the code to follow the Checks-Effects-Interactions pattern:

```diff
+ _vaultAllowance[vault][asset] += amount; // Credit to chosen vault
safeAmount = safeTransferFrom(asset, amount); // Transfer asset
- _vaultAllowance[vault][asset] += amount; // Credit to chosen vault
```

## [L-3] Missing Allowance Check in transferAllowance Function

### Description:

The `THORChain_Router::transferAllowance` function is supposed to update the `THORChain_Router::_vaultAllowances` map only if msg.sender already has the appropriate allowance for the asset. However, there is no check to ensure that msg.sender has the required allowance before updating the map.

```javascript
function transferAllowance(address router, address newVault, address asset, uint256 amount, string memory memo)
external
nonReentrant
{
if (router == address(this)) {
_adjustAllowances(newVault, asset, amount);
emit TransferAllowance(msg.sender, newVault, asset, amount, memo);
} else {
_routerDeposit(router, newVault, asset, amount, memo);
}
}
```

```javascript
    // Decrements and Increments Allowances between two vaults
    function _adjustAllowances(address _newVault, address _asset, uint256 _amount) internal {
        // q does order of Decrements and Increments Allowances between two vaults?
        _vaultAllowance[msg.sender][_asset] -= _amount;
        _vaultAllowance[_newVault][_asset] += _amount;
    }
```

```javascript
// Adjust allowance and forwards funds to new router, credits allowance to desired vault
    function _routerDeposit(address _router, address _vault, address _asset, uint256 _amount, string memory _memo)
        internal
    {
        _vaultAllowance[msg.sender][_asset] -= _amount;
        safeApprove(_asset, _router, _amount);

        iROUTER(_router).depositWithExpiry(
            _vault,
            _asset,
            _amount,
            _memo,
            type(uint256).max // q why expiry is max over here
        ); // Transfer by depositing
    }
```

### Impact:

Without this check, the function could incorrectly update allowances, potentially leading to inconsistencies in the allowance data. This might allow a vault to operate with an incorrect allowance, which could disrupt the intended flow of asset transfers between vaults.

### Proof of Concept:

The following code shows the missing check:

**There is no check to verify if msg.sender has the appropriate allowance for the asset before calling `THORChain_Router::\_adjustAllowances.**

### Recommended Mitigation:

Include a check to ensure that `msg.sender` has the necessary allowance for the asset before updating the `THORChain_Router::_vaultAllowances` map. For example:

```javascript
function transferAllowance(address router, address newVault, address asset, uint256 amount, string memory memo)
external
nonReentrant
{
require(_hasAllowance(msg.sender, asset, amount), "Insufficient allowance");

    if (router == address(this)) {
        _adjustAllowances(newVault, asset, amount);
        emit TransferAllowance(msg.sender, newVault, asset, amount, memo);
    } else {
        _routerDeposit(router, newVault, asset, amount, memo);
    }

}

function _hasAllowance(address owner, address asset, uint256 amount) internal view returns (bool) {
return _vaultAllowances[owner][asset] >= amount;
}
```

## [L-4] Missing Check for Length of Input Array

### Description:

The `THORChain_Router::batchTransferOutAndCallV5` function iterates over the `aggregationPayloads` array without checking whether the array is empty or not. This may lead to unexpected behavior if the array is empty.

```javascript
function batchTransferOutAndCallV5(TransferOutAndCallData[] calldata aggregationPayloads)
    external
    payable
    nonReentrant
{
    for (uint256 i = 0; i < aggregationPayloads.length; ++i) {
        _transferOutAndCallV5(aggregationPayloads[i]);
    }
}

```

### Impact:

If `aggregationPayloads` is an empty array, the loop will execute unnecessarily, consuming gas and potentially leading to inefficiency in contract execution.

### Recommended Mitigation:

Include a check at the beginning of the function to ensure that `aggregationPayloads` is not empty before proceeding with the loop. For example:

```diff
function batchTransferOutAndCallV5(TransferOutAndCallData[] calldata aggregationPayloads)
    external
    payable
    nonReentrant
{
+    require(aggregationPayloads.length > 0, "Empty array");

    for (uint256 i = 0; i < aggregationPayloads.length; ++i) {
        _transferOutAndCallV5(aggregationPayloads[i]);
    }
}

```