# Helix2 Technical Design
![](https://raw.githubusercontent.com/helix-coupler/resources/master/logo/helix2-inverse-small.png)
#### Author: `sshmatrix`
#### Links: [[GitHub](https://github.com/helix-coupler)]  [[.ETH](https://helix2.eth.limo)]  [[.XYZ](https://helix2.xyz)] [[Goerli](https://goerli.etherscan.io/address/0x608fbcbcf8e4d60830a97f116f7d004de48d7361)]
###### tags: `specification` `design` `architecture` `link` `bond`
# Double Helix (Helix2) 🧬🧬🧬

## Abstract

Double Helix or Helix2 is a next-generation link service on Ethereum, designed as a natural successor to generic name services. With Helix2 protocol, it is now possible to link names in several useful configurations on Ethereum blockchain. Helix2 infrastructure makes it possible to define interactions between names, categorise those interactions, assign them rules and labels, and most importantly authorise those interactions with on-chain records. Due to its unique design crafted to leverage interactions among names, Helix2 allows names to form organised on-chain structures and graphs.

## Introduction

Most blockchains have developed their own versions of naming systems which allow representing addresses with human-readable names. On Ethereum, [Ethereum Name Service](https://app.ens.domains) (ENS) is the first and most notable example, while similar services later became available on Tezos and Solana in the form of [Tezos Domains](https://tezos.domains/en) and [Solana Name Service](https://naming.bonfida.org/) by Bonfida. By construction, a name service assigns names to nodes in a network. In a classic web2 world, Domain Name Service (DNS) fulfils this requirement. Crypto-native name services are similar to DNS in the sense that they enable assigning names to addresses similar to how DNS assigns human-readable names to Internet Protocols (IP). There are however clear added benefits to crypto-native naming architecture over DNS since crypto-native services often double as a decent identity framework in their respective blockchain ecosystems. It goes without saying that the immutability and decentralisation properties of typical crypto-native systems add to their desirability owing to their censor-resistant and unruggable nature.

[Helix2](https://helix2.xyz) is designed as a next-generation successor of these name services. While the set of nodes form a canonical and natural choice for labeling in any distributed system (e.g. addresses on any blockchain), the observation nonetheless is that most nodes do not interact with each other. For instance, there are about 220 million Ethereum addresses whereas an average address will likely interact with no more than a few hundred other addresses its lifetime. This means that most wallets have a limited set of interactions with contracts and addresses in general and their interactions are classifiable to a very large degree. Keeping this in mind, Helix2 is an attempt to provide Ethereum with a next-generation link service, in addition to the name service already provided by ENS. The expected outcome of the Helix2 protocol is a link-native naming ecosystem, a 'linkspace', where an interaction between two names is representable on-chain similar, but not limited to, a human-readable name for an address. A shift in focus from nodes to links has a profound effect on the nature of structures that an ecosystem can support. Several features which are challenging to achieve with a name service alone become extremely convenient with Helix2. For instance, Helix2 comes with several fundamental and easily accessible structural features such as stealth payments, social graphs, DAOs etc.

![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/gif/helix2-schema.gif)

While Helix2 has its own namespace, it does not replace ENS and is in fact intended to work alongside ENS as an extension. Helix2 is able to import all `.` namespaces by design without any bridging or wrapping. Lastly, Helix2 is not the only link service in the works; [Woolball](https://woolball.xyz) is another link service currently under development although the two implementations arguably have more differences than similarities.

### Definitions

Helix2 (Helix + 2) is roughly motivated by the double helix structure of DNA, where two polynucleotide chains are connected by bonds. The blockchain representation of this structure is two identical copies of the same blockchain connected by secure or unsecure bonds. The nomeclature is therefore borrowed from fundamental chemistry. Below is a list of definitions that will appear in this document:

- A **name** is an on-chain label for a node in a distributed system. An example of this is `vitalik.eth` labeling the address of Vitalik on Ethereum.

- A **link** is an open relationship between two names where one name (source) may interact with another name (target) without latter's explicit approval. An example of a 'link' is  transfer of ERC20 or ERC721 tokens from one name to another, e.g. `vitalik.eth` and `virgil.eth` are linked via ERC interface.

- A source in Helix2 architecture is called a **cation**.

- A target in Helix2 architecture is called an **anion**.

- A **bond** is a closed relationship between two names that requires explicit approval of both the cation (source) and the anion (target). An example of a bond is granting of controller permissions to an operator by an owner using ERC721 `setApprovalForAll()` function. In this case, the interaction between the owner and controller is of closed nature<sup>[1](#1)</sup>. In other words, a bond is a secure link since it requires explicit approval. Alternatively, a link is an unsecure bond.

- **Covalence** is a flag in Helix2 architecture that defines whether an interaction or relationship is a link or a bond. Covalence is `true` for a secure bond and `false` for an unsecure bond (aka link).

- A **molecule** is a set of similar bonds between a cation and several anions. An example of a molecule is the set of links between a DAO governor contract (source) and the participating token holders (targets). This molecule is secure since such membership of a DAO requires explicit approval of both the engaging entities. The individual bonds in this molecule are similar in the sense that the anions share the same relationship with the contract. A multi-sig vault is also an example of a molecule. A public channel in a Discord server is another example of a molecule.

- A **polycule** is a special type of molecule in which each individual bond between the cation and its several anions is unique. An example of a polycule is the set of private channels in a Discord server, where each channel may have its unique requirements as well as unique participating members.

- A **hook** is a contractual (or non-contractual) relationship (a contract) between two names. Technically, a hook is a map consisting of two elements `rule → config`.

- A **config** is the address of the contract defining a hook between two names. For non-contractual relationships, `0x0` config is used.

- A **rule** is an identifier which encodes the relationship between two names and must accompany a hook as its calldata.

## Helix2 Design

### Ethereum Name Service (ENS)

All name services so far have essentially been on-chain scalar databases (e.g. ENS, LENS, LNR, CB.ID), meaning that names are simply isolated nodes representable by one label (see figure below). ENS is a good example of such a scalar architecture. Typical name services like ENS are also hierarchical namespaces in the sense that the set of subnodes and nodes form a merkle-tree with labels as leaves of the tree. In such an architecture, names are self-contained and isolated by construction.

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/ens.png)

### Helix2 Service

Helix2 is essentially two services under one protocol: a native namespace providing a name service and a link service for that underlying namespace. Helix2 is an on-chain vector database. In Helix2, names can bond (or link) with one another; bonds are vectors between names, pointing from one name to another - this is the core premise of Helix2 protocol and essentially what separates it from other services. Helix2 names are not hierarchical, meaning that they cannot have subdomains, i.e. Helix2 is a flat namespace. Subdomains are not necessary in Helix2 since linking provides the same features without constraining the relationships between nodes to within one parent node's hierarchy. This is the only significant divergence of Helix2 namespace from ENS. In addition to this flat namespace, Helix2 has an additional linkspace which is the core utility of the protocol.

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/helix2.png)

While the theoretical premise of a linkspace appears easy, in practise, this premise requires some crucial modifications to work in a resource-constrained environment such as Ethereum Virtual Machine (EVM). One of these modifications consist of allowing one name to link with multiple names within one object with common or unique relationships among them. This is highly consequential in reducing gas costs for serial linkers.

## Helix2 Protocol

All native objects (names, links, bonds etc) end with `.`, e.g. `alice.`, where `.` acts as a trailing marker. Consequently, `.` is one of the three reserved characters in Helix2 and cannot be used in any of the object labels (other than as a suffix)

### Names

```
struct NAME {
  address owner;
  address controller;
  address resolver;
  uint256 expiry;
}
```

Helix2 names are similar to ENS names, except that the suffix for them is `.` instead of `.eth`. Helix2 names are not hierarchical, meaning that they cannot have subdomains.

- All Helix2 names end with `.` and they have a Resolver and Controller. Note again that `.` is a reserved character and therefore forbidden. Additionally, `-` and `#` are also forbidden.

#### Importing ENS names

Helix2 is capable of importing all `.` namespaces by forbidding `.` in its native namespace. In the prototype implementation of Helix2, ENS name import is already implemented and ENS users can claim their ENS on Helix2. For ENS names, the representation of names then looks like `alice.eth.`, i.e. ENS name `alice.eth` followed by a `.` as the usual trailing marker.

### Bonds

```
struct BOND {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32 anion;
    bytes32 label;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
```

A directional bond between two names `alice.` → `bob.` is labeled by its label `label`. Bonds start with `-`, end with `.` and can be queried by their label prefixed with `-`, e.g. `-label.`

The source of the bond is called a cation (`alice.`) and the target is called an anion (`bob.`)

Further, `covalence` flag determines whether the bond is 'secure' or 'unsecure' (i.e. when the bond is in fact a link). To reiterate,

- a bond between alice and bob is insecure when it is uni-directional and requires only alice's approval, or a
- a bond between alice and bob is secure when it is mutual, bi-directional and requires both alice's and bob's approval

### Hooks, Rules & Config

The most important feature of bonds are `config` and `rule`,  combining to form a `hook`, which quantify the link between two names and gives meaning to the `-` representation. Hooks are contractual mappings` rule → config` between two names, mediated by ordered one-to-one mapping inside rules: `uint8[] rules`

- To query a hook's config inside `hooks` for a bond, one needs its associated `rule`, which is a `uint8` identifier mapping to the contractual address `config`. Hooks are thus queryable as `-label#rule.`, e.g. `-label#404.`
- A trivial application of a hook is a payment router, i.e. payment sent to `0` hook `-label#0.` is routed to the address of `bob.`; more on hooks in upcoming sections.

### Molecules

```
struct MOLECULE {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32[] anions;
    bytes32 label;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
```

Helix2 allows for multi-bonding such that a cation can bond with multiple anions within one data structure instead of creating individual (and costlier) bonds; this structure is called a 'molecule' (or 'moly' in short). In a molecule, all individual bonds share the same covalence.

Other features of a molecule are similar to that of a bond, e.g. a molecule can have an label and hooks. To denote a bond with label `label`, we use two consecutive `--` characters, i.e. `--label.` without refering to an anion since molecules are anion-agnostic and hook-independent. By using `--label#rule.`, one can refer to a unique hook for a molecule.

### Polycules

```
struct POLYCULE {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32[] anions;
    bytes32 label;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
```

Lastly, we can define another useful abstraction in the form of a 'polycule', which is a molecule comprising of unique bonds between a cation and a set of anions. In a polycule, all individual bonds share the same covalence despite being unique. In short, `rules.length == anions.length`, and `rules` & `anions` are a one-to-one map.

Other features of a molecule are similar to that of a molecule. To denote a bond with label `label`, we use three consecutive `---` characters, i.e. `---label.`  etc. By using `---label#rule.`, one can refer to a unique hook for a molecule by its `rule`. Alternatively, one can refer to a unique anion in a molecule by its indexed `rule`, e.g. `---label#anion[rule]`

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/helix2-struct.png)
&nbsp;

### Remarks

The `molecule` structure is the topological superset of [`name`, `bond`, `polycule`], i.e. it is possible to derive polycules and bonds from molecules although that'll literally be a gas-guzzling mistake. The seemingly unnecessary differentiation between the three is to optimise gas consumption.

## Contracts

Helix2 Protocol is a fairly large set of code divided into clean modules in a way that all submodules are replaceable and upgradeable without breaking any existing architecture or interfaces. This is achieved through a combination of basic logic & storage separation via proxies and [EIP-2535 Diamond Standard](https://eips.ethereum.org/EIPS/eip-2535).

There are four submodules in the genesis version of protocol - Name, Bond, Molecule and Polycule. Each submodule is composed of three contracts - Registrar, Registry & Storage. In each submodule, the storage contracts are proxied by registry contracts such that the logic and storage is seperated. This enables the protocol to replace the storage and logic in each submodule seemlessly through future protocol upgrades without compromising on-top services. Name submodule is somewhat more central to the architecture than the remaining three link modules since links depend on and derive from names.

The four submodules are connected at the core by the Helix2 Core manager which is a state-of-the-art [Multi-facet Proxy (EIP-2535)](https://eips.ethereum.org/EIPS/eip-2535). EIP-2535 compliant contracts are fully upgradeable and it allows Helix2 protocol to use one single address forever without compromising on future functionalities or breaking existing on-chain or off-chain functionalities. Helix2 manager is capable of excepting new submodules, replace existing submodules or update core configuration.


![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/helix2-contracts.png)

Helix2 `v0.0.2` contracts are now deployed for testing on Goerli:

Helix2 Core: [`0x0`]()

| - | `Registrar` | `Registry` |
| - | - | - |
| Name | [`0x0`](https://goerli.etherscan.io/address/0xB9BB951aaA33cd723c536BDac80691f87b1778f4) | [`0x0`](https://goerli.etherscan.io/address/0x74E71F8d9015881a98065E10Dc1DefFA81E46A22) |
| Bond | [`0x0`](https://goerli.etherscan.io/address/0x77932429a41521BE353EA8E4076507AF3619A08E) | [`0x0`](https://goerli.etherscan.io/address/0xa87F2075EC87b50C30Cff3D23F42034D0188941d) |
| Molecule | [`0x0`](https://goerli.etherscan.io/address/0xe5f2A931182fCbf9B31A6C7f5d525d9259CF4251) | [`0x0`](https://goerli.etherscan.io/address/0xD61C1CCf935AeF5D9B9449bBDf811d63592FE94e) |
| Polycule | [`0x0`](https://goerli.etherscan.io/address/0xBDeE64035fA5B85e91c6A178572A159149D521F8) | [`0x0`](https://goerli.etherscan.io/address/0xE1752F3A3579064284DB9d1C21DEDF95C1414d9D) |

The testing stage is expected to last some weeks at least to allow for proper sampling of feedback from the wider community.

## Utility

Helix2 is a core infrastructure which can be leveraged for perhaps countless utilities. Some of the notable few that come to mind are the following.

#### Multi-sig Safes

By design, Helix2 molecules are a perfect fit for multi-sig vault management. Molecule struct is readily mappable to a multi-sig safe architecture and formalise vaults as nameable linkspaces.

#### Stealth payments

Helix2 hooks can be configured for bonds to receive stealth payments to a name via a stealth protocol such as [Umbra Protocol](https://github.com/ScopeLift/umbra-protocol) or [Aztec](https://zk.money/). Note that this feature is also possible with ENS Resolver.

#### Web3 social graphs

Helix2 molecules are a natural fit for open social graphs and could aid in better decentralised social media protocols.

#### Messaging and chat apps

Helix2 molecules are a natural fit for web3 messaging apps and chat services. In particular, molecules are a canonical choice for organising group chat participants and their permissions whereas polycules could be utilised to manage private channels. Both in combination could help build the elusive web3 version of Discord.

#### DAOs

Helix2 molecules are a canonical fit for DAO governance and associated tooling where delegates can be viewed as members of the same molecule with common relationship among all members of the DAO. [Orca Protocol](https://orca.mirror.xyz/Y2xvPmB4cJH51srGqY6Mm_g38lV-7cwvtyDePnyzfAE) is a similar idea which has been implemented already albeit with limited focus on on-chain coordination.

#### Other use-cases

Other possible use-cases include

- a unified web3 individual and group reputation system,
- proof-of-humanness by defining human-specific hooks in molecules or polycules,
- and perhaps anything and everything that requires structuring in groups.

## Future progress

Progress should come thick and fast in the coming weeks or months while contract undergoes stress tests and improvements. Primary near-term objectives include:

- Off-chain hooks to reduce gas costs for large molecules,
- Helix2 Manager Client App,
- Helix2 Subgraph,
- Preset hooks for stealth payments, social graphs and DAOs,
- and a lot of other ideas

---

#### Footnotes

<a name="1">[1]</a> Although this example uses addresses as nodes, a similar implementation is nonetheless possible with names.


### Previous deployments

#### v0.0.1

Helix2 `v0.0.1` contracts are now deployed for testing on Goerli:

Helix2 Core: [`0x608fbcbcf8e4d60830a97f116f7d004de48d7361`](https://goerli.etherscan.io/address/0x608fbcbcf8e4d60830a97f116f7d004de48d7361)

| - | `Registrar` | `Registry` |
| - | - | - |
| Name | [`0xB9B...8f4`](https://goerli.etherscan.io/address/0xB9BB951aaA33cd723c536BDac80691f87b1778f4) | [`0x74E...A22`](https://goerli.etherscan.io/address/0x74E71F8d9015881a98065E10Dc1DefFA81E46A22) |
| Bond | [`0x779...08E`](https://goerli.etherscan.io/address/0x77932429a41521BE353EA8E4076507AF3619A08E) | [`0xa87...41d`](https://goerli.etherscan.io/address/0xa87F2075EC87b50C30Cff3D23F42034D0188941d) |
| Molecule | [`0xe5f...251`](https://goerli.etherscan.io/address/0xe5f2A931182fCbf9B31A6C7f5d525d9259CF4251) | [`0xD61...94e`](https://goerli.etherscan.io/address/0xD61C1CCf935AeF5D9B9449bBDf811d63592FE94e) |
| Polycule | [`0xBDe...1F8`](https://goerli.etherscan.io/address/0xBDeE64035fA5B85e91c6A178572A159149D521F8) | [`0xE17...d9D`](https://goerli.etherscan.io/address/0xE1752F3A3579064284DB9d1C21DEDF95C1414d9D) |
