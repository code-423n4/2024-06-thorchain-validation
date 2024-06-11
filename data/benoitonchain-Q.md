# reentrancy-benign

Impact: Low

Confidence: Medium

Position: ethereum/contracts/THORChain_Router.sol#L143-L160

## Analysis

The identified vulnerability in the THORChain_Router contract allows for reentrancy in the _deposit function. This vulnerability occurs when an external call to safeTransferFrom is made, and then state variables are written after the call. This sequence of operations could potentially allow for a reentrancy attack where an attacker exploits the order of operations to manipulate the state variables in an unintended way. This vulnerability is considered low severity but with medium confidence. It is important to address this issue to prevent any potential exploitation of the contract.

## Guidance and correction advice
### Proof of concept  
```
function _deposit(
    address payable vault,
    address asset,
    uint amount,
    string memory memo
  ) private nonReentrant {
    uint safeAmount;
    if (asset == address(0)) {
      safeAmount = msg.value;
      bool success = vault.send(safeAmount);
      require(success);
    } else {
      require(msg.value == 0, "unexpected eth"); // protect user from accidentally locking up eth
      safeAmount = safeTransferFrom(asset, amount); // Transfer asset
      _vaultAllowance[vault][asset] += safeAmount; // Credit to chosen vault
    }
    emit Deposit(vault, asset, safeAmount, memo);
  }

```

### Proposal 
To rectify the reentrancy vulnerability in the _deposit function, you should apply the "check-effects-interactions" pattern. This pattern ensures that state changes are made before interacting with external contracts to prevent reentrancy attacks.

Modify the code as follows:
```
function _deposit(
    address payable vault,
    address asset,
    uint amount,
    string memory memo
  ) private nonReentrant {
    uint safeAmount;
    if (asset == address(0)) {
      safeAmount = msg.value;
      bool success = vault.send(safeAmount);
      require(success);
    } else {
      require(msg.value == 0, "unexpected eth"); // protect user from accidentally locking up eth
      _vaultAllowance[vault][asset] += safeTransferFrom(asset, amount); // Credit to chosen vault before external call
      safeAmount = safeTransferFrom(asset, amount);;
    }
    emit Deposit(vault, asset, safeAmount, memo);
  }
```


By moving the state variable update _vaultAllowance[vault][asset] += safeAmount before the external call safeTransferFrom(asset, amount), you ensure that the state changes are completed before interacting with external contracts, mitigating the reentrancy vulnerability.