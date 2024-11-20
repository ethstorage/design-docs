# Purpose

The purpose of this document is to introduce the rationale and proposed design for `soul gas token`(`SGT`), a *
*non-transferable** wrapped L2 native gas token. It provides a new way to subsidize and incentivize users, trying to
solve the problems of the current way.

# Problem Statement + Context

To promote mass adoption of web3 and draw in users from web2, project teams often apply strategies like fee subsidies
and airdrops.

- The practice of using fee subsidies typically involves employing account abstraction through EIP-4337, yet adopting
  smart contract wallets continues to pose difficulties. Despite ongoing discussions about EIP-3074 and EIP-4337, most
  users continue to prefer traditional wallets.
- Airdrops have not been effective in maintaining genuine user engagement, as there is a noticeable decline in on-chain
  activity post-redemption.

To tackle the challenges faced by chain operators, `SGT` is proposed as an innovative approach to subsidize and motivate
users.

# Proposed Solution

We introduce `SGT`, an ERC-20 token that mirrors the L2 native gas token with a 1:1 value peg.
SGT permits the L2 native gas token to act as collateral and serve as the gas token on L2, though it is not transferable
between users.
Both Chain operators and Governance foundations can leverage `SGT` to incentivize users, who in turn can utilize `SGT`
for paying transaction fees.

## Prerequisites

The prerequisite for using `SGT` is to use the [
`custom gas token`](https://specs.optimism.io/experimental/custom-gas-token.html) feature of the OP Stack as the native
gas token for L2.

## Implementation

### Contract

#### Non-transferable feature

Implement the non-transferable feature. Disallow the functions below:

```solidity

    function transfer(address _to, uint256 _value) public returns (bool success);

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success);

    function approve(address _spender, uint256 _value) public returns (bool success);

    function allowance(address _owner, address _spender) public view returns (uint256 remaining);

```

#### Deposit and withdrawal

Implement the deposit and withdrawal functions, inheriting
the [ERC20Wrapper](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Wrapper.sol)
and overriding
the `deposit` and `withdraw` functions. Chain operators or governance can mint and burn `SGT` by depositing and
withdrawing the L2 native gas token.

### State Transition Logic

#### Gas fee processing 

Modify the state transition logic to utilize `SGT` for gas fees. 

1. Adjust the `busGas` function logic: if the `SGT` contract is already deployed, in addition to fetching the user's balance, the contract's `balanceOf` method should be invoked to retrieve the user's `SGT` balance. Prioritize using `SGT` to purchase `Gas` first.
```go
 func (st *StateTransition) buyGas() error
```

2. Refund the user's `SGT` if it remains after the gas fee is paid.

3. Fee Rewards also uses `SGT` to reward sequencer.


# Challenges

1. How to reward fees to sequencers using `SGT`? 
- The functions of transferring `SGT` are disabled, so how can sequencers receive `SGT` as rewards?
- How to ensure that the 1:1 peg between `SGT` and the L2 native gas token is maintained? 

# Security Considerations

1. Risk of DoS Attacks
2. Sybil Attack Prevention