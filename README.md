# Helix2 Technical Design
![](https://raw.githubusercontent.com/helix-coupler/resources/master/logo/helix2-inverse-small.png)
#### Author: `sshmatrix`
#### Links: [[GitHub](https://github.com/helix-coupler)]  [[.ETH](https://helix2.eth.limo)]  [[.XYZ](https://helix2.xyz)]
###### tags: `specification` `design` `architecture` `link` `bond`
# Double Helix (Helix2)

Double Helix ([Helix2](https://helix2.xyz)) is an on-chain node-coupling protocol designed to link and/or bind names. Most blockchains have developed their versions of a naming system which allows representing addresses with human-readable names. Helix2 is designed as a possible next-generation successor of these name services.

For instance, Ethereum Name Service (ENS) is the first major on-chain name service on Ethereum in the form of a heirarchical Merkle tree-like data structure. Like all name services, it focuses on assigning names to the nodes (usually addresses) on the blockchain. Double Helix, or Helix2, now aims to standardise on-chain representation of links and bonds in addition to names.

### Extending nodes to links and bonds

While the set of nodes form a canonical and natural choice for labeling of addresses on any blockchain, the observation nonetheless is that most nodes do not interact with each other on chain. In fact, most wallets have a limited set of interactions with contracts and addresses. Keeping this in mind, we attempt to provide Ethereum with a next-generation 'link service', in addition to the name service already provided by ENS. The expected result of Helix2 service is a link-native naming ecosystem where an interaction between two names is representable on-chain similar to a human-readable name for an address. In addition, it comes with several structural features such as private payments, social graphs, DAOs etc. Note that while Helix2 has its own namespace, it does not replace ENS and is in fact intended to work alongside ENS as an extension. Lastly, Helix2 is not the only link service in the works; [Woolball](https://woolball.xyz) is another link service currently under development although the two implementations arguably have more differences than similarities.

#### Definitions

a) A 'link' is an open relationship between two entities where one entity (source) may set the link with or without explicit approval of the other (target),

b) A 'bond'  is a closed relationship between two entities that requires explicit approval of both the source and the target. In other words, a bond is a secure link. Alternatively, a link is an unsecure bond. All bonds are links but not all links are bonds.

# Helix2

Helix2 (Helix + 2) is motivated roughly by the double helix structure of DNA, where two polynucleotide chains are connected by bonds. The blockchain representation of this structure is two copies of a blockchain connected by secure bonds or unsecure links. All name services so far have been essentially on-chain scalar databases (e.g. ENS, LENS, LNR, CB.ID), meaning that names are simply isolated nodes representable by one label (see figure below)

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/ens.png)

Helix2, in comparison, is an on-chain vector database. In Helix2, names can bond (or link) with one another; bonds are vectors between names, pointing from one name to another. In succinct, the basic syntax for the namespace is as follows.

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/helix2.png)

### Global

All native objects (names, links, bonds etc) end with `.`, e.g. `alice.`, whereas `.` acts as a trailing marker. Consequently, `.` is one of the two reserved characters in Helix2 and cannot be used in any of the object labels (other than as a suffix)

## Names

<pre>
struct NAME {
  address owner;
  address controller;
  address resolver;
  uint256 expiry;
}
</pre>

Helix2 names are similar to ENS names, except that the suffix for them is `.` instead of `.eth`. Helix2 names are not heirarchical, meaning that they cannot have subdomains.

- All Helix2 names end with `.` and they have a Resolver and Controller. Note again that `.` is a reserved character and therefore forbidden. Additionally,`_` and `#` are also forbidden.

## Bonds

<pre>
struct BOND {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32 anion;
    bytes32 alias;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
</pre>

A directional bond between two names `alice.` → `bob.` is labeled by its alias `alias`. Bonds start with `_`, end with`.` and can be queried by their alias prefixed with `_`, e.g. `_alias.`

The source of the bond is called a cation (`alice.`) and the target is called an anion (`bob.`)

Further, `covalence` flag determines whether the bond is 'secure' or 'unsecure' (i.e. when the bond is in fact a link). To reiterate,

- a bond between alice and bob is insecure when it is uni-directional and requires only alice's approval, or a
- a bond between alice and bob is secure when it is mutual, bi-directional and requires both alice's and bob's approval

### Hooks & Rules

The most important feature of bonds are `hooks` and `rules`, which quantify the link between two names and give meaning to the `_` representation. Hooks are contractual definitions between two names:` mapping(uint8 => address)`, mediated by ordered one-to-one mapping inside rules: `uint8[] rules`

- To query a hook inside `hooks` for a bond, one needs its associated `rule`, which is a `uint8` identifier mapping to the contractual address `hook`. Hooks are thus queryable as `_alias#rule.`, e.g. `_alias#404.`
- A trivial application of a hook is a payment router, i.e. payment sent to `0` hook `alias_#0.` is routed to the address of `bob.`. More on hooks in upcoming sections.

## Molecules

<pre>
struct MOLECULE {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32[] anions;
    bytes32 alias;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
</pre>

Helix2 allows for multi-bonding such that a cation can bond with multiple anions within one data structure instead of creating individual (and costlier) bonds; this structure is called a 'molecule' (or 'moly' in short). In a molecule, all individual bonds share the same covalence.

Other features of a molecule are similar to that of a bond, e.g. a molecule can have an alias and hooks. To denote a bond with alias `alias`, we use two consecutive `__` characters, i.e. `__alias.` without refering to an anion since molecules are anion-agnostic and hook-independent. By using `__alias#rule.`, one can refer to a unique hook for a molecule.

## Polycules

<pre>
struct POLYCULE {
    uint8[] rules;
    mapping(uint8 => address) hooks;
    bytes32[] anions;
    bytes32 alias;
    address resolver;
    address controller;
    bool covalence;
    uint256 expiry;
}
</pre>

Lastly, we can define another useful abstraction in the form of a 'polycule', which is a molecule comprising of unique bonds between a cation and a set of anions. In a polycule, all individual bonds share the same covalence despite being unique. In short, `rules.length == anions.length`, and `rules` & `anions` are a one-to-one map.

Other features of a molecule are similar to that of a molecule. To denote a bond with alias `alias`, we use three consecutive `___` characters, i.e. `___alias.`  etc. By using `___alias#rule.`, one can refer to a unique hook for a molecule by its `rule`. Alternatively, one can refer to a unique anion in a molecule by its indexed `rule`, e.g. `___alias#anion[rule]`

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/schema/helix2-struct.png)
&nbsp;

### Remarks

The `molecule` structure is the topological superset of [`name`, `bond`, `polycule`], i.e. it is possible to derive polycules and bonds from molecules although that'll literally be a gas-guzzling mistake. The seemingly unecessary differentiation between the three is to optimise gas consumption.

Note that in a heirarchical namespace such as ENS, the labels of subnodes form the leaves of the Merkle tree. Helix2, on the other hand, is an "inverted" Merkle tree in the sense that names are the leaves and linking is the path toward root node from the leaves.
