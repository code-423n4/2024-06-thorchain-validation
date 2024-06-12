## Title
Use of send function, leading to fail if gas to high.

## Lines of code
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L152
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L424

## Impact
Using the send function can lead to transaction failures if the recipient requires more than 2300 gas to process the received Ether. This could result in:

1. Loss of Funds: If the transaction fails, the Ether may not be sent to the intended recipient.
2. Blocked Operations: Operations dependent on successful Ether transfers could be blocked or reverted, causing disruptions in the protocol's functionality.
3. Vulnerability to Attacks: Malicious actors could exploit this limitation to disrupt the protocol by ensuring that their fallback or receive functions consume more than 2300 gas, leading to denial-of-service attacks.


## Proof-of-code
If the recipient's fallback or receive function requires more than 2300 gas, the send function will fail, and the transaction will revert.

    bool success = vault.send(safeAmount);


    bool success = asgard.send(msg.value);


## Recommendation
Use call function instead:

    (bool success, ) = recipient.call{value: amount}("");
    require(success, "Transfer failed.");



## Title
Costly operations inside a loop


## Lines of code
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L250-L252
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L400-L402
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L415-L417
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L420-L422


## Impact
Operations such as state variable updates inside a loop cost a lot of gas, are expensive and may lead to out-of-gas errors.


## Proof-of-concept
(https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop)


    for (uint i = 0; i < transferOutPayload.length; ++i) {
      _transferOutV5(transferOutPayload[i]);
    }


    for (uint i = 0; i < aggregationPayloads.length; ++i) {
      _transferOutAndCallV5(aggregationPayloads[i]);
    }


    for (uint i = 0; i < coins.length; i++) {
        _adjustAllowances(asgard, coins[i].asset, coins[i].amount);
      }


    for (uint i = 0; i < coins.length; i++) {
        _routerDeposit(router, asgard, coins[i].asset, coins[i].amount, memo);
      }


## Recommendations
Optimizations using local variables are preferred.



## Title
Missing events


## Lines of code
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L422
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol#L176


## Impact
The absence of events for important state-changing functions can lead to several issues:

1. Lack of Transparency: Without events, it is challenging to track the execution of critical functions, making it harder for users and developers to verify contract behavior.
2. Difficulty in Debugging: Missing events complicate the debugging process, as there is no logged data to analyze when an issue arises.
3. Reduced Monitoring Capability: External applications, such as monitoring tools and decentralized applications (dApps), rely on events to detect and react to changes in the contract. Without events, these tools may fail to function correctly.
4. Compliance and Auditing Issues: Regulatory compliance and security audits often require a clear record of contract operations. Missing events can hinder these processes and raise concerns about the contract's reliability and integrity.


## Proof-of-concept
Detect missing events for critical arithmetic parameters.
https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-arithmetic

      function returnVaultAssets(
    address router,
    address payable asgard,
    Coin[] memory coins,
    string memory memo
      ) external payable nonReentrant {
    if (router == address(this)) {
      for (uint i = 0; i < coins.length; i++) {
        _adjustAllowances(asgard, coins[i].asset, coins[i].amount);
      }
      emit VaultTransfer(msg.sender, asgard, coins, memo); // Does not include ETH.
    } else {
      for (uint i = 0; i < coins.length; i++) {
        _routerDeposit(router, asgard, coins[i].asset, coins[i].amount, memo);
      }
    }
    bool success = asgard.send(msg.value);
    require(success);
      }



      function transferAllowance(
    address router,
    address newVault,
    address asset,
    uint amount,
    string memory memo
      ) external nonReentrant {
    if (router == address(this)) {
      _adjustAllowances(newVault, asset, amount);
      emit TransferAllowance(msg.sender, newVault, asset, amount, memo);
    } else {
      _routerDeposit(router, newVault, asset, amount, memo);
    }
      }


## Recommendation
Emit an event


## Title
Protocol doesn't handle ERC20 tokens with decimals other than 18, leading tokens with many decimal values may cause issues due to overflow, while tokens with few decimal values may result in a loss of precision.


## Lines of code
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol


## Impact
The incorrect handling of tokens with different decimal places can result in several issues:

1. Overflow Errors: When handling tokens with more than 18 decimals, calculations may exceed the maximum value that can be stored in a uint256 variable, causing overflow errors and potentially leading to incorrect or unpredictable behavior.
2. Loss of Precision: Tokens with fewer than 18 decimals can result in a loss of precision in calculations, leading to inaccurate accounting and potentially significant financial discrepancies.
3. Incompatibility: The protocol may become incompatible with a wide range of ERC20 tokens, limiting its usability and reducing the number of tokens that can be supported.
4. User Frustration: Users interacting with the protocol may experience unexpected behavior or errors when using tokens with non-standard decimals, leading to a poor user experience and loss of trust in the protocol.


## Proof-of-concept
https://solodit.xyz/issues/potential-funds-locked-due-low-token-decimal-and-long-stream-duration-spearbit-locke-pdf


## Recommendations
To mitigate this issue, it is recommended that the decimal values of all tokens be checked, and appropriate handling measures implemented.
Maybe add a function like this:

    function getAdjustedAmount(IERC20 token, uint256 amount) internal view returns (uint256) {
        uint8 decimals = token.decimals();
        return amount * (10 ** (18 - decimals));
    }



## Title
Doesn't check for Handling Support For Fee on Transfer Tokens and Deflationary Tokens.


## Lines of code
https://github.com/code-423n4/2024-06-thorchain/blob/main/ethereum/contracts/THORChain_Router.sol


## Impact
Different ERC-20 token implementations behave differently regarding the actual amount received when transferring tokens. USDT on Ethereum, for example, can charge a fee when transferring ERC-20 tokens.
Deflationary tokens like STA, meanwhile, burn a certain percentage of the transferred amount, which subsequently decreases the token supply.
As a result, the transfer of STA and USDT, and other tokens like them, to a smart contract can result in incorrect token accounting.


## Recommendation
To mitigate this issue, the contract should check the balance before and after a transfer, updating the balance of the msg.sender based on the amount subtracted for fees or burnt tokens.

    function Deposit(uint256 amount) public {
    uint balance_before = token.balanceOf(address(this) 
    token.transferFrom(msg.sender, address(this), amount); 
    uint balance_after = token.balanceOf(address(this); 
    require(balance_after >= balance_before );
    uint amount_to_add = balance_after — balance_before; 
    balances[msg.sender] += amount_to_add;
    }