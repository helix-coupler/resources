![](https://raw.githubusercontent.com/helix-coupler/resources/master/logo/helix2-color-small.png)

# Helix2 🧬

## About

Double Helix or Helix2 is a next-generation link service and account abstraction protocol on Ethereum, originally designed as a natural successor to generic name services. Helix2 protocol allows names to link to each other in several useful configurations on Ethereum blockchain. Helix2 infrastructure codifies interactions between names, categorises those interactions, assigns them rules and labels, and in some cases, validates those interactions with on-chain records. Due to its unique design crafted to leverage interactions among names, Helix2 enables names to form organised on-chain structures and graphs. Wallets integrating Helix2 can leverage its properties to provide users with a smart contract interface to transact and interact with the Ethereum blockchain; this arguably makes Helix2 an account abstraction infrastructure.  

## Description

Most blockchains have developed their own versions of naming systems which allow representing addresses with human-readable names. On Ethereum, [Ethereum Name Service](https://app.ens.domains) (ENS) is the first and most notable example, while similar services later became available on Tezos and Solana in the form of [Tezos Domains](https://tezos.domains/en) and [Solana Name Service](https://naming.bonfida.org/) by Bonfida. By construction, a name service assigns names to nodes in a network. In a classic web2 world, Domain Name Service (DNS) fulfils this requirement. Crypto-native name services are similar to DNS in the sense that they enable assigning names to addresses similar to how DNS assigns human-readable names to Internet Protocols (IP). There are however clear added benefits to crypto-native naming architecture over DNS since crypto-native services often double as a decent identity framework in their respective blockchain ecosystems. It goes without saying that the immutability and decentralisation properties of typical crypto-native systems add to their desirability owing to their censor-resistant and unruggable nature.

[Helix2](https://helix2.xyz) is designed as a next-generation successor of these name services. While the set of nodes form a canonical and natural choice for labeling in any distributed system (e.g. addresses on any blockchain), the observation nonetheless is that most nodes do not interact with each other. For instance, there are about 220 million Ethereum addresses whereas an average address will likely interact with no more than a few hundred other addresses its lifetime. This means that most wallets have a limited set of interactions with contracts and addresses in general and their interactions are classifiable to a very large degree. Keeping this in mind, Helix2 is an attempt to provide Ethereum with a next-generation link service, in addition to the name service already provided by ENS. The expected outcome of the Helix2 protocol is a link-native naming ecosystem, a 'linkspace', where an interaction between two names is representable on-chain similar, but not limited to, a human-readable name for an address. A shift in focus from nodes to links has a profound effect on the nature of structures that an ecosystem can support. Several features which are challenging to achieve with a name service alone become extremely convenient with Helix2. For instance, Helix2 comes with several fundamental and easily accessible structural features such as stealth payments, social graphs, DAOs etc.

## Contracts

Helix2 Protocol is divided into clean modules in a way that all submodules are replaceable and upgradeable without breaking any existing architecture or interfaces. This is achieved through a combination of basic logic & storage separation via proxies and [EIP-2535 Diamond Standard](https://eips.ethereum.org/EIPS/eip-2535).

There are four submodules in the genesis version of protocol - Name, Bond, Molecule and Polycule. Each submodule is composed of three contracts - Registrar, Registry & Storage. In each submodule, the storage contracts are proxied by registry contracts such that the logic and storage is seperated. This enables the protocol to replace the storage and logic in each submodule seemlessly through future protocol upgrades without compromising on-top services. Name submodule is somewhat more central to the architecture than the remaining three link modules since links depend on and derive from names.

The four submodules are connected at the core by the Helix2 Core manager which is a state-of-the-art [Multi-facet Proxy (EIP-2535)](https://eips.ethereum.org/EIPS/eip-2535). EIP-2535 compliant contracts are fully upgradeable and it allows Helix2 protocol to use one single address forever without compromising on future functionalities or breaking existing on-chain or off-chain functionalities. Helix2 manager is capable of excepting new submodules, replace existing submodules or update core configuration.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/contracts.png)

## Account Abstraction

Helix2 is a core infrastructure which can be leveraged for perhaps countless utilities. One of the biggest utilities of Helix2 arises in account abstraction. **A**ccount **A**bstraction (AA) is an umbrella term first defined in [EIP-2938](https://eips.ethereum.org/EIPS/eip-2938), to categorise infrastructures that allow the use of smart contracts as wallets instead of the typical logic-free 'externally owned accounts' (EOA) that are currently in use as wallets. The core ideology behind account abstraction is to allow users to arbitrarily customise their handling of transactions, funds and interactions with the Ethereum blockchain in general with smart contract code.

Several decentralised applications (dApps) on Ethereum already allow specific smart control over blockchain interactions (e.g. [Gnosis Vault](https://safe.global/), [Umbra Cash](https://app.umbra.cash), [Tornado Cash](https://tornadocash.eth.link/), [Gas Station Network](https://opengsn.org/) but this list is rather small and the utility of them rather limited to only a few aspects. These services may additionally require users to pay additional fees on top of the (un)optimised gas costs.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/abstraction.png)

EIP-2938 proposed a new set of `OPCODES` that would allow account abstraction at core level but this required significant non-trivial changes to Ethereum consensus layer. [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337) proposed another infrastructure that uses alternative mempools ('alt mempools') for bundling transactions and then routing them to the consensus layer upon validation (akin to [Flashbots](https://www.flashbots.net/); the alt mempools would then be responsible for validating the smart wallet transaction calls and thereby such a validation & routing environment remains independent of the primary mempool and wouldn't require changes to the consensus layer itself. Due to the independent nature of the alt mempools, there is also room for mempool-level optimisations (e.g. of bundlers) to further minimise gas costs which is usually not the case for typical dApps.

Helix2 is also an account abstraction infrastructure that uses a namespace and its associated linkspaces, i.e. data structures containining pointers to smart contracts from names, to act as a router of transactions for any name (see figure above). Since the name- and linkspace are disjoint from the underlying EOA, the user pays no other gas costs than the standard network transaction fees, excluding the one-time cost of writing the pointers in the Helix2 data structure. Wallets connecting users to the blockchain via their names are therefore abstracted by Helix2 since the user can insert arbitrary logic to their name- and linkspace. ENS already provides a fraction of this functionality but loses out on the ability to enforce dynamic arbitrary front-facing logic due to its heirarchical nature. The several specific usecases of Helix2 mentioned in the upcoming sections are all examples of account abstraction.

#### Multi-sig Safes

By design, Helix2 molecules are a perfect fit for multi-sig vault management. Molecule struct is readily mappable to a multi-sig safe architecture and it readily formalises vaults as nameable linkspaces. In fact, Helix2 can store multiple vault agreements among a set of parties inside a single standard struct.

#### Stealth payments

Helix2 hooks can be configured for bonds to receive stealth payments to a name via a stealth protocol such as [Umbra Protocol](https://github.com/ScopeLift/umbra-protocol) or [Aztec](https://zk.money/). Note that this feature is also possible with ENS Resolver. In addition however, Helix2 allows performing stealth payments enmasse and with better control over sensitive data streams.

#### Web3 social graphs

Helix2 molecules are a natural fit for open social graphs and could aid in better decentralised social media protocols. In the current implementation, molecules are an appropriate representation of social groupings, although a complete version requires one or more submodules that can link molecules and polycules among each other. This requirement may vary from one platform to another, and is therefore not part of the Helix2 core protocol.

#### Messaging and chat apps

Helix2 molecules are a natural fit for web3 messaging apps and chat services. In particular, molecules are a canonical choice for organising group chat participants and their permissions whereas polycules could be utilised to manage private channels. Both in combination could help build the elusive web3 version of Discord.

#### DAOs

Helix2 molecules are a canonical fit for DAO governance and associated tooling where delegates can be viewed as members of the same molecule with common relationship among all members of the DAO. [Orca Protocol](https://orca.mirror.xyz/Y2xvPmB4cJH51srGqY6Mm_g38lV-7cwvtyDePnyzfAE) is a similar idea which has been implemented already albeit with limited focus on on-chain coordination. Helix2 in comparison makes the same task significantly easier at protocol level and allows formation of highly complex pods at much lower architectural cost.

#### Other use-cases

Other possible use-cases include

- a unified web3 individual and group reputation system,
- proof-of-humanness by defining human-specific hooks in molecules or polycules,
- and perhaps anything and everything that requires structuring in groups.

## Future

In near future, Helix2 expects to integrate off-chain lookups for extremely large link structures using Chainlink's [CCIP](https://chain.link/cross-chain) Protocol, integrate presets for stealth payments (via [Umbra](https://github.com/ScopeLift/umbra-protocol)) and a bridge to Dostr - Bitcoin's [Nostr](https://nostr.com) client. Upon completion of these tasks, full Helix2 Protocol will be deployed on Ethereum Mainnet L1, with CCIP bridges to all primary L2 chains such as Optimism, zkSync, Starknet etc following soon after. Coordinates of the WIP codebase are at the end of the document.

## Roadmap

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/png/roadmap.png)

🧬🧬🧬
