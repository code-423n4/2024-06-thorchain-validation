# Events-Access

The vulnerability identified as events-access in the Ownable.transferOwnership function of the USDT.sol contract is a missing event emission for the critical access control parameter 'owner = newOwner'. This means that when ownership of the contract is transferred to a new address, an event should be emitted to log this change. 
Without this event, there is a lack of transparency and auditability in the contract, as important access control actions are not being properly recorded. This could potentially lead to difficulties in tracking ownership changes and identifying unauthorized modifications to the contract's ownership.

**Guidance and correction advice :**

```solidity
    function transferOwnership(address newOwner) public onlyOwner {
        if (newOwner != address(0)) {
            owner = newOwner;
        }
    }

```

To rectify the vulnerability in the

```
transferOwnership
```

function, you should emit an event whenever the owner is updated to a new address. This helps in tracking critical parameter changes and enhances transparency within the smart contract.

```solidity
pragma solidity ^0.8.0;
contract A {
    address public owner;

    modifier onlyAdmin() {
        require(msg.sender == owner, "Not authorized");
        _;
    }

    event OwnerUpdated(address indexed oldOwner, address indexed newOwner);

    constructor() {
        owner = msg.sender;
    }

    function updateOwner(address newOwner) external onlyAdmin {
        require(newOwner != address(0), "Invalid destination address");
        address oldOwner = owner;
        owner = newOwner;
        emit OwnerUpdated(oldOwner, newOwner);
    }
}
```