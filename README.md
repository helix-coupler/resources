# Helix2 Technical Design
#### <span style="color:grey">authors:</span> keccak256(sshmatrix)
###### tags: `specification` `design` `architecture`

# Double Helix (Helix2)

Double Helix (Helix2) is an **on-chain node coupler** protocol designed to link names. Most blockchains have developed their versions of a naming system which allows representing addresses with human-readable names. 

Ethereum Name Service (ENS) is the first major on-chain name service on Ethereum in the form of a heirarchical Merkle tree data structure. Like all name services, it focuses on assigning names to the nodes (usually addresses) on the blockchain. Double Helix, or Helix2, alternatively aims to standardise on-chain representation of 'links' in addition to names.

### Worthiness of links over nodes

While the set of nodes form a canonical and natural choice for labeling of addresses on any blockchain, the observation nonetheless is that most nodes do not interact with each other on-chain. In fact, most wallets have a limited set of interactions with contracts and addresses. Keeping this in mind, we attempt to provide Ethereum with a next-generation link service, in addition to the name service already provided by ENS. The expected result of Helix2 service is a link-native ecosystem where an interaction between two entities is representable on-chain similar to a name. Note that while Helix2 has its own namespace, it nonetheless does not replace ENS, but is in fact an extension intended to work alongside ENS.

### Helix2 basics

Helix2 (Helix + 2) is motivated roughly by the double helix structure of DNA, where two polynucleotide chains are connected by bonds. The blockchain representation of this structure is two copies of a blockchain connected by links. All name services so far have been essentially on-chain **scalar** databases (e.g. ENS, LENS, LNR, CB.ID), meaning that names are simply isolated nodes representable by one label (see below). 

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/ens.png)

Helix2, in comparison, is an on-chain **vector** database.  Very quickly, the basic syntax for the namespace is as follows:

#### Syntax

- All **names** end with `#`, e.g. `alice#`
- A directional **link** between two names alice → bob is written as `alice#bob`
- We denote a **hook** labeled `label` of link `alice#bob` with `alice#bob_label`

In Helix2, **names** can *bond* with one another. **Bonds** are directed vectors, pointing from one name to another. 

## Architecture

### Syntax

- All **names** end with `#`, e.g. `alice#`
- A directional **link** between two names alice → bob is written as `alice#bob`
- We denote a **hook** labeled `label` of link `alice#bob` with `alice#bob_label`

### Idea

The idea for the architecture is as follows:

#### Name

Names are represented by `namehash` such that
```
namehash ~ keccak256(alice)
``` 

#### Link (+ Hooks)

Each name can link to another name (`alice`  → `bob`). Links are represented by `linkhash` such that 
```
linkhash ~ keccak256(keccak256(alice), keccak256(bob))
```
A basic link structure then looks like:
<pre>
struct <b>LINK</b> {
    bytes32 _from;
    bytes32 _to;
    bytes32 _alias;
    address _resolver;
    address _controller;
}
</pre>

Each link can have multiple hooks. Hooks are enumerable and indexed by `n`. Hooks are labeled by `labelhash ~ keccak256(label)`.

Each hook is a merkle node with a unique `hookhash ~ keccak256(labelhash, linkhash)`, mapping to a contractual relationship `hookhash => contract` between two names (e.g. chat, loan, yield, social, multisig etc etc) 
```
mapping(bytes32 => address) hooks
```
such that the `_resolver` and `_controller` may now be placed inside hooks

<pre>
struct <b>LINK</b> {
    <i>mapping(bytes32 => address) _hooks;</i> <b>//contains _resolver & _controller</b>
    bytes32 _from;
    bytes32 _to;
    bytes32 _alias;
    <s>address _resolver;</s>
    <s>address _controller;</s>
}
</pre>

When unlinking a link, one must unhook every hook in the link.

#### Tree

Since each name will link to several others with similar configuration, it is meaningful to also define a **tree** structure that allows for memory-efficient multi-linking to multiple linkee:

<pre>
struct <b>TREE</b> {
    <i>mapping(bytes32 => address) _hooks;</i>
    bytes32 _from;
    <b>bytes32[] _to;</b>
    bytes32 _alias;
}
</pre>

#### Web

Further memory-efficient abstraction is possible by defining increasingly complex structures, if needed:

<pre>
struct <b>WEB</b> {
    <b><i>mapping(bytes32 => address[]) _hooks;</i></b>
    bytes32 _from;
    <b>bytes32[] _to;</b>
    bytes32 _alias;
}
</pre>

#### More on Hooks (and rules)

Since hooks may be owner-specific, we must flag all hooks that are **non-transferable**. When a name is transferred, all links with **non-transferable** rule must break. To accomodate this feature, the structure of hooks then needs an update to include the **rules** for them:

<pre>
mapping(bytes32 => <b>mapping(uint8 => address[])</b>) _hooks;
</pre>

where `uint8` keeps track of the rules for each hook. In the simplest implementation when there are only two rules (transferable or otherwise), `bool` is a more natural choice. In order to allow for multiple rules however, we choose `uint8` (`non-transferable = 0`; `0 < transferable <= 2^8`).


### Schematic


### Contract ABI

#### ownerOf()

*Function*: `ownerOf()` : *<span style="color:grey"> returns the owner of a name</span>*
*Syntax*: `function ownerOf(bytes32 namehash) returns (address);`

#### transfer()

*Function*: `transfer()` : *<span style="color:grey"> transfers name to new owner</span>*
*Syntax*: `function transfer(bytes32 namehash, address newOwner);`

~~#### resolver()~~

~~*Function*: `resolver()` : *<span style="color:grey"> returns the resolver for a name*</span>
*Syntax*: `function resolver(bytes32 namehash) returns (address);`~~

~~#### setResolver()~~

~~*Function*: `setResolver()` : *<span style="color:grey"> sets the resolver for a  name</span>*
*Syntax*: `function setResolver(bytes32 namehash, address resolver);`~~

#### getLinkCount()

*Function*: `getLinkCount()` : *<span style="color:grey"> returns the number of links in format `[in, out]` for a name</span>*
*Syntax*: `function getLinkCount(bytes32 namehash) returns (uint[](2));`

#### getLink()

*Function*: `getLink()` : *<span style="color:grey"> returns a link</span>*
*Syntax*: `function getLink(bytes32 linkhash) returns (bytes32);`

#### link()

*Function*: `link()` : *<span style="color:grey"> creates a new link and returns its hash</span>*
*Syntax*: `function link(bytes32 from, bytes32 to, bytes32 alias) return (bytes32);`

#### getHookCount()

*Function*: `getHookCount()` : *<span style="color:grey"> returns the number of hooks in a link</span>*
*Syntax*: `function getHookCount(bytes32 linkhash) returns (uint);`

#### getHook()

*Function*: `getHook()` : *<span style="color:grey"> returns a hook</span>*
*Syntax*: `function getHook(bytes32 hookhash) returns (address);`

#### hook()

*Function*: `hook()` : *<span style="color:grey"> creates a new hook in a link with label</span>*
*Syntax*: `function hook(bytes32 linkhash, bytes32 labelhash);`

#### unlink()

*Function*: `unlink()` : *<span style="color:grey"> closes a link</span>*
*Syntax*: `function unlink(bytes32 linkhash);`

#### rehook()

*Function*: `rehook()` : *<span style="color:grey">rehooks a hook to a new link</span>*
*Syntax*: `function rehook(bytes32 hookhash, bytes32 newLinkhash);`

#### unhook()

*Function*: `unhook()` : *<span style="color:grey"> returns `true` after unhooking a hook; returns `false` otherwise</span>*
*Syntax*: `function unhook(bytes32 hookhash) returns (bool);`

#### unhookAll()

*Function*: `unhookAll()` : *<span style="color:grey"> returns `true` after unhooking all hooks inside a link; returns `false` otherwise</span>*
*Syntax*: `function unhookAll(bytes32 linkhash) returns (bool);`


