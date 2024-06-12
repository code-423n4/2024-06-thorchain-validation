Summary:
The review of the THORChain_Router contract did not reveal any critical vulnerabilities that could be exploited by an attacker. However, several areas for improvement were identified, primarily related to best practices in Solidity development. Adopting these practices can enhance the security and robustness of the contract.

Use of send in returnVaultAssets
Severity: Low
Location: returnVaultAssets function
Issue: The function currently uses send for transferring ETH, which restricts the gas forwarded to 2300 gas units. This can be insufficient for more complex recipient contracts.
Impact: Limited flexibility and potential for ETH transfer failures to complex contracts.
Recommendation: Replace send with call for more flexible gas management and better error handling.
Suggested Code Change:

solidity
Copy code
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

    // Use call instead of send
    (bool success, ) = asgard.call{value: msg.value}("");
    require(success, "ETH transfer failed");
}

what I Found: The nonReentrant modifier is used to prevent reentrancy attacks, which is great.
Suggestion: Alongside the nonReentrant modifier, using the checks-effects-interactions pattern would make our defenses even stronger. This means updating our internal state before making external calls to minimize risks.
Allowance Adjustments

What I Found: Adjusting allowances looks solid.
Suggestion: Using SafeMath (or Solidity's built-in safe arithmetic) could add an extra layer of safety against overflows and underflows, though not critical in our current Solidity version (0.8.22).
ETH Balance Consistency

What I Found: There’s an assert to check if the ETH balance is non-negative.
Suggestion: This check is a bit redundant because ETH balances can’t be negative due to the nature of unsigned integers. We can probably remove it for a cleaner code.
Expiration Handling

What I Found: The contract checks for expiration using require.
Suggestion: This is already in good shape, nothing to add here. 👍
Valid Address Checks

What I Found: Addresses are validated to ensure they’re not zero.
Suggestion: This looks solid. Always good to avoid the zero address.
ERC-20 Transfer Success

What I Found: We’re using low-level calls to check the success of token transfers.
Suggestion: Switching to a library like OpenZeppelin’s SafeERC20 could make these operations more robust and less prone to errors. It’s like a safety net for token transfers and approvals.
Safe TransferFrom Implementation

What I Found: The safeTransferFrom function works but could handle some edge cases better.
Suggestion: Keep an eye out for tokens that charge fees on transfers. Using a library might help handle these cases more gracefully.
ETH Transfer Failure Handling

What I Found: We’re using send for ETH transfers and have a fallback in case it fails.
Suggestion: Switching to call instead of send for ETH transfers is usually better. It handles failures more gracefully and works well with modern contracts.
Payload Handling in TransferOutAndCallV5

What I Found: The payload handling seems fine, no immediate issues here.
Suggestion: Just keep making sure that payloads are secure and can’t introduce unexpected behavior or vulnerabilities.
Batch Transfer Consistency

What I Found: Batch operations are processed consistently.
Suggestion: Consider adding checks to make these operations atomic (all-or-nothing) or improve failure handling for robustness.
Safe Approval

What I Found: The approval process works but could be better.
Suggestion: Switching to SafeERC20 for approvals helps avoid race conditions and improper approvals.
Recommendations
Reentrancy: Keep using nonReentrant, but also think about adopting checks-effects-interactions for added safety.
Allowance Adjustments: Consider adding SafeMath for arithmetic operations.
ETH Balance Check: Remove redundant checks on ETH balance.
ERC-20 Interactions: Use OpenZeppelin’s SafeERC20 library for handling ERC-20 tokens.
ETH Transfers: Prefer call over send for better handling of ETH transfers.
Batch Operations: Improve batch handling to ensure consistency and atomicity.
Approvals: Use SafeERC20 for managing token approvals.
Conclusion:
Overall, the THORChain_Router contract looks solid with no glaring vulnerabilities. Implementing these best practices will make it even more robust and secure against potential issues. Let’s keep refining it to make sure we’re always a step ahead!