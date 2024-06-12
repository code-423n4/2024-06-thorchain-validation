### [L-01] Missing zero address checks in `THORChain_Router.sol`

**Description:**
Zero address checks are missing in many functions inside `THORChain_Router.sol`. This can lead to losing of the user's funds if incorrect parameters are passed.
Missing in functions: `THORChain_Router::_deposit`, `THORChain_Router::transferOut`, `THORChain_Router::_transferOutV5`, `THORChain_Router::_transferOutAndCallV5`,`THORChain_Router::_adjustAllowances`

**Impact:**
Users or Vaults may lose their funds by sending them to the zero address

**Recommended Mitigation:** Add zero address checks to the functions to prevent fund loss