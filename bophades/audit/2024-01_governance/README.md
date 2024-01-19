# Olympus DAO: On-Chain Governance Audit

## Purpose

The purpose of this audit is to review the On-Chain Governance system that will power Olympus V3 moving forward. This system is an enhancement of [Compound's Governor Bravo](https://github.com/compound-finance/compound-protocol/tree/master/contracts/Governance) with minor logic changes to adapt its contracts to the Olympus codebase (there are also stylistic adjustments).

These contracts will be installed in the Olympus V3 "Bophades" system, based on the [Default Framework](https://docs.olympusdao.finance/main/contracts/overview).

## Scope

### In-Scope Contracts

The contracts in-scope for this audit are:

-   [governance](../../src/external/governance/)
    -   [abstracts](../../src/external/governance/abstracts/)
        -   [GovernorBravoStorage.sol](../../src/external/governance/abstracts/GovernorBravoStorage.sol)
    -   [interfaces](../../src/external/governance/interfaces/)
        -   [IGohm.sol](../../src/external/governance/interfaces/IComp.sol)
        -   [ITimelock.sol](../../src/external/governance/interfaces/ITimelock.sol)
        -   [IGovernorAlpha.sol](../../src/external/governance/interfaces/IGovernorAlpha.sol)
        -   [IGovernorBravoEvents.sol](../../src/external/governance/interfaces/IGovernorBravoEvents.sol)
    -   [Timelock.sol](../../src/external/governance/Timelock.sol)
    -   [GovernorBravoDelegate.sol](../../src/external/governance/GovernorBravoDelegate.sol)
    -   [GovernorBravoDelegator.sol](../../src/external/governance/GovernorBravoDelegator.sol)

Tests for the in-scope contracts are contained in the following locations:

-   [GovernorBravoDelegate.t.sol](../../src/test/external/GovernorBravoDelegate.t.sol)

## Previous Audits

The governance system has been forked from Compound's Governor Bravo, which has been previously audited by several firms:

-   [Open Zeppelin: Governor Bravo Audit](https://blog.openzeppelin.com/compound-governor-bravo-audit)
-   [Open Zeppelin: Governor Alpha Audit](https://blog.openzeppelin.com/compound-alpha-governance-system-audit)
-   [Trail of Bits: Governor Alpha Audit](https://github.com/trailofbits/publications/blob/master/reviews/compound-governance.pdf)

A comparison between the original Governor Bravo implementation (by Compound) and the adaptation made for Olympus can be found here:

-   [GovernorBravoDelegate/GovernorBravoDelegate](https://github.com/OlympusDAO/bophades/compare/docs/gov-bravo-og..governance-clean#diff-f8822168bdc78b46963cb47c126bb265c16f8a118829c5e26518cc9452016a13)
-   [GovernorBravoDelegate/GovernorBravoDelegator](https://github.com/OlympusDAO/bophades/compare/docs/gov-bravo-og..governance-clean#diff-43b818278518d0ec5220e069f896ad508cae8049f4d34798c76d89db35be8302)
-   [Timelock](https://github.com/OlympusDAO/bophades/compare/docs/gov-bravo-og..governance-clean#diff-985a4d65eddd182a4d5dcf4da18a4a6c6d7025b2cb730d5aca456aa83a281f28)

## Frequently-Asked Questions

### Q: Is the code/contract expected to comply with any EIPs? Are there specific assumptions around adhering to those EIPs that Watsons should be aware of?

A: None

## Getting Started

This repository uses Foundry as its development and testing environment. You must first [install Foundry](https://getfoundry.sh/) to build the contracts and run the test suite.

### Clone the repository into a local directory

```sh
git clone https://github.com/OlympusDAO/bophades
```

### Install dependencies

```sh
cd bophades
git checkout governance-clean
pnpm run install # install npm modules for linting and doc generation
forge build # installs git submodule dependencies when contracts are compiled
```

### Build

Compile the contracts with `forge build`.

### Tests

Run the full test suite with `pnpm run test`. However, there are some Fork tests for other parts of the protocol that can run into RPC rate limit issues. It is recommended to run the test suite without the fork tests for this audit. Specifically, you can run `pnpm run test:unit`.

Fuzz tests have been written to cover a range of inputs. Default number of runs is 256, more were used when troubleshooting edge cases.

### Linting

Pre-configured `solhint` and `prettier-plugin-solidity`. Can be run by

```sh
pnpm run lint
```

### Code Metrics

Code metrics have been calculated using the following command:

```shell
pnpm run metrics src/external/governance/**/*.sol
```

## Olympus Governor Bravo Implementation

The Olympus protocol is governed and upgraded by gOHM (Governance OHM) token holders, using three distinct components: the [gOHM](https://etherscan.io/token/0x0ab87046fBb341D058F17CBC4c1133F25a20a52f) token, the governance policy ([Governor Bravo](../../src/external/governance/GovernorBravoDelegate.sol)), and a [Timelock](../../src/external/governance/Timelock.sol). Together, these contracts enable the community to propose, vote on, and implement changes to the Olympus V3 system. Proposals can modify system parameters, activate or deactivate policies, and even install or upgrade modules, effectively allowing the addition of new features and the mutation of the protocol.

gOHM token holders can delegate their voting rights either to themselves or to an address of their choice. Due to the elasticity in the gOHM supply, and unlike the original implementation of Governor Bravo, the Olympus governance system relies on dynamic thresholds based on the total gOHM supply. This mechanism sets specific thresholds for each proposal, based on the current supply at that time, ensuring that the requirements (in absolute gOHM terms) for proposing and executing proposals scale with the token supply.

### Proposal Thresholds

When someone attempts to create a new governance proposal, the required thresholds for successfully submitting and executing the proposal are calculated. These thresholds determine:

-   The minimum amount of votes (either by direct voting power or delegated) required by the proposer to submit the proposal.
-   The minimum quorum required to pass the proposal (determined in favorable votes).

#### Proposal Submission Threshold

The proposal submission threshold is determined by calling the `getProposalThresholdVotes()` function:

```solidity
function getProposalThresholdVotes() public view returns (uint256) {
    return (gohm.totalSupply() * proposalThreshold) / 100_000;
}
```

#### Proposal Quorum Threshold

The minimum proposal quorum is determined in two steps. First, the contract checks whether the proposal attempts to perform a Kernel action that directly targets a high-risk module or attempts to (un)install a policy that requires permissions from a high-risk module.
If the proposal targets a high-risk module, the quorum is calculated using the `getHighRiskQuorumVotes()` function:

```solidity
function getHighRiskQuorumVotes() public view returns (uint256) {
    return (gohm.totalSupply() * highRiskQuorum) / 100_000;
}
```

_Note that a set of high-risk modules is tracked in the `isKeycodeHighRisk` mapping. The set is upgradable with the admin function `_setModuleRiskLevel()`._

Otherwise, it is calculated using the `getQuorumVotes()` function:

```solidity
function getQuorumVotes() public view returns (uint256) {
    return (gohm.totalSupply() * quorumPct) / 100_000;
}
```

### Proposal Timeline

When a governance proposal is created, it enters a X-day review period, after which voting weights are recorded and voting begins. Voting lasts for Y days. After the voting period concludes, if a majority of votes, along with a minimum quorum, are cast in favor of the proposal, it is queued in the `Timelock` and can be implemented Z days later.

Considerations:

-   The proposal can then be executed during a grace period of W days; afterwards, it will expire.
-   The proposal can be canceled at any time (before execution) by the proposer.
-   The proposal can be vetoed at any time (before execution) by the veto guardian. Initially, this role will belong to the DAO multisig. However, once the system matures, it could be set to the zero address.

![proposal-timeline-diagram](./proposal-timeline-diagram.svg)

### Vote Delegation

gOHM is an [ERC-20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) token that allows the owner to delegate voting rights to any address, including their own.

Considerations:

-   Users can delegate to only one address at a time.
-   The entire gOHM balance of the delegator is added to the delegatee’s vote count.
-   Changes to the delegator's token balance automatically adjust the voting rights of the delegatee.
-   Votes are delegated from the current block and onward, until the delegator delegates again or transfers all their gOHM.

Delegation can be achieved by calling the `delegate()` function or via a valid off-chain signature using `delegateBySig()`.

#### Delegate

```solidity
function delegate(address delegatee)
```

-   `delegatee`: The address in which the sender wishes to delegate their votes to.
-   `msg.sender`: The address of the COMP token holder that is attempting to delegate their votes.
-   `RETURN`: No return, reverts on error.

#### DelegateBySig

Delegate votes from the signatory to the delegatee. This method has the same purpose as Delegate but it instead enables off-chain signatures to participate in Olympus governance vote delegation. For more details on how to create an off-chain signature, review [EIP-712](https://eips.ethereum.org/EIPS/eip-712).

```solidity
function delegateBySig(address delegatee, uint nonce, uint expiry, uint8 v, bytes32 r, bytes32 s)
```

-   `delegatee`: The address in which the sender wishes to delegate their votes to.
-   `nonce`: The contract state required to match the signature. This can be retrieved from the contract’s public nonces mapping.
-   `expiry`: The time at which to expire the signature. A block timestamp as seconds since the unix epoch (uint).
-   `v`: The recovery byte of the signature.
-   `r`: Half of the ECDSA signature pair.
-   `s`: Half of the ECDSA signature pair.
-   `RETURN`: No return, reverts on error.

### Vote Casting

Olympus' implementation of Governor Bravo employs a pessimistic vote casting mechanism. This mechanism operates under the assumption that all governance proposals could potentially be malicious. Therefore, it consistently computes the most unfavorable vote cast numbers at different timestamps for each participant. The rationale behind this approach is to safeguard the system against scenarios where voters or proposers might attempt to gain or lose exposure during the lifetime of a proposal by trying to front-run the outcome of the vote.

This pessimistic vote casting approach aims to align each participants's interests with the long-term health and stability of the system, ensuring that decisions reflect the genuine consensus of the Olympus community, rather than short-term, opportunistic actions.

#### **Proposer**

As the participant introducing a (potentially malicious) change to the system, the proposer is required to gather enough votes to meet the minimum proposal submission threshold. Therefore, at every transition within the proposal timeline, the system checks that this requirement is met before altering the proposal's state. If not, the transaction will be reverted. This approach ensures that proposers (or their delegators) maintain skin in the game, preventing them from benefiting from malicious proposals that could depreciate the value of the gOHM token.

_Whitelisted proposers can bypass the minimum proposal submission threshold._

```solidity
function queue(uint256 proposalId) external {
    // Checks ...

    // Check that proposer has not fallen below proposal threshold since proposal creation
    // If proposer is whitelisted, they can queue regardless of threshold
    if (
        !isWhitelisted(proposal.proposer) &&
        gohm.getPriorVotes(proposal.proposer, block.number - 1) < proposal.proposalThreshold
    ) revert GovernorBravo_Queue_BelowThreshold();

    // Effects and interactions...
}
```

```solidity
function execute(uint256 proposalId) external payable {
    // Checks...

    // Check that proposer has not fallen below proposal threshold since proposal creation
    // If proposer is whitelisted, they can execute regardless of threshold
    if (
        !isWhitelisted(proposal.proposer) &&
        gohm.getPriorVotes(proposal.proposer, block.number - 1) < proposal.proposalThreshold
    ) revert GovernorBravo_Queue_BelowThreshold();

    // Effects and interactions...
}
```

#### **Voters**

As the participants responsible for approving changes within the system, voters play a crucial role in maintaining the integrity of the governance process. To ensure fairness, the system records the minimum amount of voting power held by a voter between the time a proposal is created and when the voter casts their vote. This mechanism is designed to prevent voters from altering their exposure level to the gOHM and influence the outcome of the vote. By doing so, it attempts to safeguard the voting process against strategies that manipulate voting power.

```solidity
function castVoteInternal(
    address voter,
    uint256 proposalId,
    uint8 support
) internal returns (uint256) {
    // Checks...

    // Get the user's votes at the start of the proposal and at the time of voting. Take the minimum.
    uint256 originalVotes = gohm.getPriorVotes(voter, proposal.startBlock);
    uint256 currentVotes = gohm.getPriorVotes(voter, block.number);
    uint256 votes = currentVotes > originalVotes ? originalVotes : currentVotes;

    // Effects...
}
```
