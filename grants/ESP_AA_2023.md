# HELIX2 🧬

## Abstract
Helix2 (or Double Helix) is a next-generation link service and abstraction protocol on Ethereum that allows names to link among each other, host smart contracts and route payments. Due to its unique design crafted to leverage interactions among names instead of addresses, Helix2 can provide users with a front-facing smart contract interface to transact and interact with the Ethereum blockchain without an explicit need for alt mempools.

## Objectives
Helix2 is a next-generation link service and account abstraction protocol on Ethereum designed as a successor to existing name services. Helix2 aims to provide Ethereum with a link-native naming ecosystem, a 'linkspace', where interactions between two names can be represented on-chain. The practical outcomes of Helix2 are several fundamental and easily accessible structural features such as stealth payments, on- and off-chain networks, and enabling the use of front-facing custom logic in smart contract wallets instead of the typical externally owned accounts that are currently in use as wallets. Helix2 due to its unique design can be leveraged for countless utilities, including account abstraction. Helix2 intends to accomplish the aforementioned goals without an explicit need for alt mempools or bundlers, although it is versatile enough to work independently of the local consensus logic.

## Outcomes

EIP-2938 initially proposed a set of `OPCODES` that would allow account abstraction at core level but this required significant non-trivial changes to Ethereum consensus layer. EIP-4337 proposed a different infrastructure that uses alt mempools for bundling transactions and then routing them to the consensus layer upon validation akin to Flashbots; the alt mempools would then be responsible for validating the smart wallet transaction calls and thereby such a validation & routing environment remains independent of the primary mempool and wouldn't require changes to the consensus layer itself. Due to the independent nature of the alt mempools, there is also room for mempool-level optimisations (e.g. of bundlers) to further minimise gas costs which is usually not the case for typical dApps.

Helix2 is a unique account abstraction infrastructure that uses a namespace and its associated linkspaces, i.e. data structures containining pointers to smart contracts from names, to act as a router of transactions for any name (see figure above). Since the name- and linkspace are disjoint from the underlying EOA, the user pays no other gas costs than the standard network transaction fees, excluding the one-time cost of writing the pointers in the Helix2 data structure. Wallets connecting users to the blockchain via their names are therefore abstracted by Helix2 since the user can insert arbitrary logic to their name- and linkspace. ENS already provides a fraction of this functionality but loses out on the ability to enforce dynamic arbitrary front-facing logic due to its heirarchical nature.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/abstraction.png)

## Grant Scope
The grant covers development work on Helix2 from its current state at v0.5-alpha to v0.9-beta. This explicitly includes:

- Stealth payments in all sub-modules,
- Networking protocols in all sub-modules,
- v0.9-beta pre-release codebase completed and open for audit, and
- EIP-4337 compatibility checks to allow functionality at base abstraction layer.

## Project Team
The Helix2 team currently comprises of a single degen developer based out of India (`sshmatrix.eth`)

## Background
[`sshmatrix.eth`](https://sshmatrix.eth.limo) is an astrophysicist-turned-programmer with 10+ academic papers and several crypto projects under his belt.

## Methodology
Helix2 aims to build a decentralized and immutable infrastructure that provides a layered vector linkspace for Ethereum ecosystem and can be leveraged for countless utilities. The protocol is divided into four submodules - Name, Bond, Molecule, and Polycule, each composed of three contracts - Registrar, Registry, and Storage. The storage contracts are proxied by registry contracts, enabling seamless protocol upgrades and replacement of storage and logic in each submodule. The core manager, a Multi-facet Proxy (EIP-2535) compliant contract, connects the four submodules, allowing the addition of new submodules, replacement of existing ones, or updates to the core configuration. EIP-2535 compliance allows Helix2 protocol to use one single address forever without compromising on future functionalities or breaking existing on-chain or off-chain functionalities. Helix2 manager is capable of excepting new submodules, replace existing submodules or update core configuration.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/contracts.png)

## Timeline
Prototype v0.5-alpha codebase of Helix2 is already deployed on Goerli. The expected timeline for the project is as follows:

- Q2/Q3 2023: Develop Helix2 frontend from scratch and host it on [Helix2.xyz](https://helix2.xyz) and [Helix2.eth](https://helix2.eth.limo). Simultaneously integrate stealth payments and networking protocols in submodules' configurations; this should take Helix2 closer to v0.8-alpha.

- Q4 2023: Deploy final Helix2 v0.9-beta on Goerli and open the codebase for audit.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/roadmap.png)

## Budget
**Amount Requested:** USD 25,000

The costs associated with the grant will be used for:

- **Principle Developers' Costs**, covering research and development of Helix2 codebase and Helix2 frontend, and
- **Contract Deployment Costs**, for contract deployments on mainnet; especially expensive due to large codebase of Helix2.

# Links

- [README](https://helix-coupler.github.io/resources/)
- [White Paper](https://github.com/helix-coupler/resources/blob/master/yellow-paper/helix2.pdf)
- [GitHub](https://github.com/helix-coupler/)
- [Contracts](https://github.com/helix-coupler/helix2-contracts)
