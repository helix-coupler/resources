# Helix2 Technical Design
#### <span style="color:grey">authors: keccak256(sshmatrix)
![](https://raw.githubusercontent.com/helix-coupler/resources/master/helix2-logo-small.png)
#### <span style="color:grey">repo: https://github.com/helix-coupler
###### tags: `specification` `design` `architecture` `link` `bond`
# Double Helix (Helix2)

Double Helix (Helix2) is an on-chain node-coupling protocol designed to link and/or bind names. Most blockchains have developed their versions of a naming system which allows representing addresses with human-readable names. Helix2 is designed as a possible next-generation successor of these name services.

For instance, Ethereum Name Service (ENS) is the first major on-chain name service on Ethereum in the form of a heirarchical Merkle tree-like data structure. Like all name services, it focuses on assigning names to the nodes (usually addresses) on the blockchain. Double Helix, or Helix2, now aims to standardise on-chain representation of links/bonds in addition to names.

### Extending nodes to links and bonds

While the set of nodes form a canonical and natural choice for labeling of addresses on any blockchain, the observation nonetheless is that most nodes do not interact with each other on-chain. In fact, most wallets have a limited set of interactions with contracts and addresses. Keeping this in mind, we attempt to provide Ethereum with a next-generation 'linking service', in addition to the name service already provided by ENS. The expected result of Helix2 service is a link-native naming ecosystem where an interaction between two names is representable on-chain similar to a human-readable name for an address. In addition, it comes with several structural features such as private payments, social graphs, DAOs etc. Note that while Helix2 has its own namespace, it does not replace ENS and is in fact intended to work alongside ENS as an extension.

#### Link vs Bond?

- A 'link' is an open relationship between two entities where one entity (source) may set the link with or without explicit approval of the other (target).

- A 'bond'  is a closed relationship between two entities that requires explicit approval of both the source and the target. In other words, a bond is a secure link. Alternatively, a link is an unsecure bond.

### Helix2 basics

Helix2 (Helix + 2) is motivated roughly by the double helix structure of DNA, where two polynucleotide chains are connected by bonds. The blockchain representation of this structure is two copies of a blockchain connected by secure or unsecure bonds (aka links). All name services so far have been essentially on-chain scalar databases (e.g. ENS, LENS, LNR, CB.ID), meaning that names are simply isolated nodes representable by one label (see figure below).

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/ens.png)

Helix2, in comparison, is an on-chain vector database.  In Helix2, names can bond (or link) with one another; bonds (or links) are vectors between names, pointing from one name to another. In succinct, the basic syntax for the namespace is as follows:

1. All native objects (names, links, bonds etc) end with `:`, e.g. `alice:`, whereas `:` acts as a trailing marker.

2. A directional bond between two names `alice:` → `bob:` is represented by `alice?bob:`.

3. Given a bond` alice?bob:`, the source of the bond is called a cation (`alice:`) and the target is called an anion (`bob:`).

4. Further, a bond ` alice?bob:` can be,

- Unsecure bond (link) → when `alice?bob: != bob?alice:`, i.e. when the bond between alice and bob is uni-directional and requires only alice's approval, and

- Secure bond → when `alice?bob: == bob?alice:`, i.e. when the bond between alice and bob is mutual, bi-directional and requires both alice's and bob's approval.

5. Helix2 allows for multi-bonding such that a cation can bond with multiple anions within one data structure instead of creating individual (and costlier) bonds; this structure is called a 'molecule'. In a molecule, individual bonds between a cation and the set of anions may either be secure or unsecure but not a mix of the two.

6. Lastly, we can define the highest form of abstraction in the form of a 'polycule', which is a molecule comprising of unique bonds between a cation and a set of anions. In a polycule, individual bonds between the cation and anions may be secure, unsecure or a mix of the two.

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/helix2.png)
&nbsp;

PS: Note that in a heirarchical namespace such as ENS, the labels of subnodes form the leaves of the Merkle tree. Helix2, on the other hand, is an "inverted" Merkle tree in the sense that names are the leaves and linking is the path toward root node from the leaves.

## Architecture

The idea for the architecture is as follows:

### Name

A 'name' is a generic structure in form of
<pre>
struct NAME {
  address owner;
  address resolver;
}
</pre>
mapped by `namehash ~ keccak256(alice)` such that
```
mapping (namehash => name) names;
```
Native Helix2 names do not have subdomain functionality like ENS since it is a flat namespace by choice although it is capable of importing heirarchical namespaces. The idea is essentially that if one can link names, then one shouldn't need subnodes.

### Bond & Hook

Each name can bond to another name (`alice:`  → `bob:`). Bonds are represented by `bondhash` such that for secure bonds:
```
bondhash ~ keccak256(keccak256(bob), keccak256(alice))
```
and for unsecure bonds:
```
bondhash ~ keccak256(keccak256(bob), keccak256(alice))
```
A basic bond structure then looks like:
<pre>
struct BOND {
    bytes32 cation;
    bytes32 anion;
    bytes32 alias;
    address resolver;
    address controller;
    bool secure;
}
</pre>

Each bond can have multiple hooks. Hooks are enumerable and indexed by `n`. Hooks are labeled by `labelhash ~ keccak256(label)`.

Each hook is a merkle node with a unique `hookhash ~ keccak256(labelhash, bondhash)`, mapping to a contractual relationship `hookhash => contract` between two names (e.g. chat, loan, yield, social, multisig etc etc)
```
mapping(bytes32 => address) hooks
```
such that

<pre>
struct BOND {
    mapping(bytes32 => address) hooks; <b>←</b>
    bytes32 cation;
    bytes32 anion;
    bytes32 alias;
    address resolver;
    address controller;
    bool secure;
}
</pre>

When unbonding a bond, one must attempt unhooking every hook in the bond.

### Molecule

Since each name will bond to several others with similar configuration, it is meaningful to also define a molecule structure that allows for memory-efficient bonding to multiple anions:

<pre>
struct MOLECULE {
    mapping(bytes32 => address) hooks;
    bytes32 cation;
    bytes32[] anions; <b>←</b>
    bytes32 alias;
    address resolver;
    address controller;
    bool secure;
}
</pre>

### Polycule

Further memory-efficient abstraction is possible by defining increasingly complex structures, if needed:

<pre>
struct POLYCULE {
    mapping(bytes32 => address[]) hooks; <b>←</b>
    bytes32 cation;
    bytes32[] anions; <b>←</b>
    bytes32 alias;
    address resolver;
    address controller;
    bool[] secure; <b>←</b>
}
</pre>

The polycule structure is of course the topological superset of [bond, molecule, polycule], i.e. it is possible to derive molecules and bonds from polycules although that'll literally be a gas-guzzling mistake. The seemingly unecessary differentiation between the three is to optimise gas consumption.

#### More on Hooks (and Rules)

Since hooks may be owner-specific, we must flag all hooks that are non-transferable. When a name is transferred, all bonds with non-transferable rule must break. To accomodate this feature, the structure of hooks needs an update to include the rules for them:

<pre>
mapping(bytes32 => mapping(uint8 => address[])) hooks;
</pre>

where `uint8` keeps track of the rules for each hook. In the simplest implementation when there are only two rules (transferable or otherwise), `bool` is a more natural choice. In order to allow for multiple rules however, we choose `uint8` (`non-transferable = 0`; `0 < transferable <= 2^8`).


## Contract ABI

### Registrar

#### register()

Function: `register()` : <span style="color:blue">registers a name for an owner

Syntax: `function register(bytes32 namehash, address owner)`

### Registry



#### ownerOf()

Function: `ownerOf()` : returns the owner of a name

Syntax: `function ownerOf(bytes32 namehash) returns (address);`

#### transfer()

Function: `transfer()` : transfers name to new owner

Syntax: `function transfer(bytes32 namehash, address newOwner);`

#### resolver()

Function: `resolver()` : returns the resolver for a name

Syntax: `function resolver(bytes32 namehash) returns (address);`

#### setResolver()

Function: `setResolver()` : sets the resolver for a  name

Syntax: `function setResolver(bytes32 namehash, address resolver);`

#### getBonds()

Function: `getBonds()` : returns the number of bonds in format `[in, out]` for a name

Syntax: `function getBonds(bytes32 namehash) returns (uint[](2));`

#### getBond()

Function: `getBond()` : returns a bond

Syntax: `function getBond(bytes32 bondhash) returns (bytes32);`

#### bond()

Function: `bond()` : creates a new bond and returns its hash

Syntax: `function bond(bytes32 from, bytes32 to, bytes32 alias) return (bytes32);`

#### getHooks()

Function: `getHooks()` : returns the number of hooks in a bond

Syntax: `function getHookCount(bytes32 bondhash) returns (uint);`

#### getHook()

Function: `getHook()` : returns a hook

Syntax: `function getHook(bytes32 hookhash) returns (address);`

#### hook()

Function: `hook()` : creates a new hook in a link with label

Syntax: `function hook(bytes32 bondhash, bytes32 labelhash);`

#### unbond()

Function: `unbond()` : breaks a bond

Syntax: `function unbond(bytes32 bondhash);`

#### rehook()

Function: `rehook()` : <span style="color:blue">rehooks a hook to a new link

Syntax: `function rehook(bytes32 hookhash, bytes32 newBondhash);`

#### unhook()

Function: `unhook()` : returns `true` after unhooking a hook; returns `false` otherwise

Syntax: `function unhook(bytes32 hookhash) returns (bool);`

#### unhookAll()

Function: `unhookAll()` : returns `true` after unhooking all hooks inside a bond; returns `false` otherwise

Syntax: `function unhookAll(bytes32 bondhash) returns (bool);`
