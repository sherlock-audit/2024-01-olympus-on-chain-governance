
# Olympus On-Chain Governance contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
Mainnet
___

### Q: Which ERC20 tokens do you expect will interact with the smart contracts? 
gOHM
___

### Q: Which ERC721 tokens do you expect will interact with the smart contracts? 
None
___

### Q: Do you plan to support ERC1155?
No
___

### Q: Which ERC777 tokens do you expect will interact with the smart contracts? 
None
___

### Q: Are there any FEE-ON-TRANSFER tokens interacting with the smart contracts?

No
___

### Q: Are there any REBASING tokens interacting with the smart contracts?

No
___

### Q: Are the admins of the protocols your contracts integrate with (if any) TRUSTED or RESTRICTED?
TRUSTED
___

### Q: Is the admin/owner of the protocol/contracts TRUSTED or RESTRICTED?
TRUSTED
___

### Q: Are there any additional protocol roles? If yes, please explain in detail:
Veto Guardian: TRUSTED admin that can veto proposals in case of emergency
___

### Q: Is the code/contract expected to comply with any EIPs? Are there specific assumptions around adhering to those EIPs that Watsons should be aware of?
No
___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.
N/A
___

### Q: Please provide links to previous audits (if any).
It is based on GovernorBravo which has the following audit history:
- https://blog.openzeppelin.com/compound-governor-bravo-audit
- https://blog.openzeppelin.com/compound-alpha-governance-system-audit
- https://github.com/trailofbits/publications/blob/master/reviews/compound-governance.pdf
___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, input validation expectations, etc)?
We will have a proposal simulation framework, but it doesn't effect the actual potential use of the governance system
___

### Q: In case of external protocol integrations, are the risks of external contracts pausing or executing an emergency withdrawal acceptable? If not, Watsons will submit issues related to these situations that can harm your protocol's functionality.
No
___

### Q: Do you expect to use any of the following tokens with non-standard behaviour with the smart contracts?
No
___

### Q: Add links to relevant protocol resources
- V3 Repo: https://github.com/OlympusDAO/olympus-v3
- Docs: https://docs.olympusdao.finance/
___



# Audit scope


[bophades @ 3c4098ef9b2870f4ebd912b15466780676ba7db8](https://github.com/OlympusDAO/bophades/tree/3c4098ef9b2870f4ebd912b15466780676ba7db8)
- [bophades/src/external/governance/GovernorBravoDelegate.sol](bophades/src/external/governance/GovernorBravoDelegate.sol)
- [bophades/src/external/governance/GovernorBravoDelegator.sol](bophades/src/external/governance/GovernorBravoDelegator.sol)
- [bophades/src/external/governance/Timelock.sol](bophades/src/external/governance/Timelock.sol)
- [bophades/src/external/governance/abstracts/GovernorBravoStorage.sol](bophades/src/external/governance/abstracts/GovernorBravoStorage.sol)
- [bophades/src/external/governance/interfaces/IGovernorBravoEvents.sol](bophades/src/external/governance/interfaces/IGovernorBravoEvents.sol)
- [bophades/src/external/governance/interfaces/ITimelock.sol](bophades/src/external/governance/interfaces/ITimelock.sol)


