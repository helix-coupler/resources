# Helix2 Technical Design
#### <span style="color:grey">authors:</span> keccak256(sshmatrix)
![](https://raw.githubusercontent.com/helix-coupler/resources/master/helix2-logo-small.png)
#### <span style="color:grey">repo:</span> https://github.com/helix-coupler
###### tags: `specification` `design` `architecture`
# Double Helix (Helix2)

Double Helix (Helix2) is an **on-chain node coupler** protocol designed to link names. Most blockchains have developed their versions of a naming system which allows representing addresses with human-readable names. 

Ethereum Name Service (ENS) is the first major on-chain name service on Ethereum in the form of a heirarchical Merkle tree data structure. Like all name services, it focuses on assigning names to the nodes (usually addresses) on the blockchain. Double Helix, or Helix2, alternatively aims to standardise on-chain representation of 'links' in addition to names.

### Worthiness of links over nodes

While the set of nodes form a canonical and natural choice for labeling of addresses on any blockchain, the observation nonetheless is that most nodes do not interact with each other on-chain. In fact, most wallets have a limited set of interactions with contracts and addresses. Keeping this in mind, we attempt to provide Ethereum with a next-generation 'link service', in addition to the name service already provided by ENS. The expected result of Helix2 service is a link-native ecosystem where an interaction between two entities is representable on-chain similar to a human-readable ENS name. Note that while Helix2 has its own namespace, it does not replace ENS and is in fact intended to work alongside ENS as an extension.

### Helix2 basics

Helix2 (Helix + 2) is motivated roughly by the double helix structure of DNA, where two polynucleotide chains are connected by bonds. The blockchain representation of this structure is two copies of a blockchain connected by links. All name services so far have been essentially on-chain *scalar* databases (e.g. ENS, LENS, LNR, CB.ID), meaning that names are simply isolated nodes representable by one label (see below). 

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/ens.png)

Helix2, in comparison, is an on-chain *vector* database.  In Helix2, **names** can bond with one another; **bonds** are directed links, pointing from one name to another. Very quickly, the basic syntax for the namespace is as follows:

1.  All native objects (names, bonds etc) end with `:`, e.g. `nick:`, whereas `:` acts as a trailing marker.
2. A directional **bond** between two names `nick:` → `bob:` is represented by `nick?bob:`.
3. Given a bond` nick?bob:`, the origin of the bond is called a **cation** (`nick:`) and the end is called an **anion** (`bob:`).
4. Bond ` nick?bob:` can possess features such as,
a) **polar** bond: when `nick?bob: != bob?nick:`, i.e. when the bond between nick and bob is uni-directional and requires only nick's approval, and
b)  **equal** bond: when `nick?bob: == bob?nick:`, i.e. when the bond between nick and bob is mutual, bi-directional and requires both nick's and bob's approval.
5. Helix2 allows for multi-bonding such that a cation can bond with multiple anions within one data structure instead of creating individual (and costlier) bonds; this structure is called an **molecule**. In a molecule, individual bonds between a cation and the set of anions may either be equal or polar but *not a mix* of the two.
6. Lastly, we can define the highest form of abstraction in the form of a **polycule**, which is an molecule comprising of *unique* bonds between a cation and a set of anions. In a polycule, individual bonds between the cation and anions may be equal or polar or a mix of the two.

&nbsp;
![](https://raw.githubusercontent.com/helix-coupler/resources/master/helix2.png)

## Architecture

The idea for the architecture is as follows:

#### Name

Names are represented by `namehash` such that
```
namehash ~ keccak256(alice)
``` 

#### Bond (+ Hooks)

Each name can bond to another name (`nick:`  → `bob:`). Bonds are represented by `bondhash` such that for equal bonds:
```
bondhash ~ keccak256(keccak256(bob), keccak256(unicode"↔"), keccak256(nick))
```
and for polar bonds:
```
bondhash ~ keccak256(keccak256(bob), keccak256(unicode"←"), keccak256(nick))
```
A basicbond structure then looks like:
<pre>
struct <b>BOND</b> {
    bytes32 _from;
    bytes32 _to;
    bytes32 _alias;
    address _resolver;
    address _controller;
}
</pre>

Each bond can have multiple hooks. Hooks are enumerable and indexed by `n`. Hooks are labeled by `labelhash ~ keccak256(label)`.

Each hook is a merkle node with a unique `hookhash ~ keccak256(labelhash, bondhash)`, mapping to a contractual relationship `hookhash => contract` between two names (e.g. chat, loan, yield, social, multisig etc etc) 
```
mapping(bytes32 => address) hooks
```
such that

<pre>
struct <b>BOND</b> {
    <b>mapping(bytes32 => address) _hooks;</b>
    bytes32 _from;
    bytes32 _to;
    bytes32 _alias;
    address _resolver;
    address _controller;
}
</pre>

When unbonding a bond, one must attempt unhooking every hook in the bond.

#### MOLECULE

Since each name will bond to several others with similar configuration, it is meaningful to also define a **molecule** structure that allows for memory-efficient bonding to multiple anions:

<pre>
struct <b>MOLECULE</b> {
    mapping(bytes32 => address) _hooks;
    bytes32 _from;
    <b>bytes32[] _to;</b>
    bytes32 _alias;
    address _resolver;
    address _controller;
}
</pre>

#### POLYCULE

Further memory-efficient abstraction is possible by defining increasingly complex structures, if needed:

<pre>
struct <b>POLYCULE</b> {
    <b>mapping(bytes32 => address[]) _hooks;</b>
    bytes32 _from;
    <b>bytes32[] _to;</b>
    bytes32 _alias;
    address _resolver;
    address _controller;
}
</pre>

### More on Hooks (and rules)

Since hooks may be owner-specific, we must flag all hooks that are **non-transferable**. When a name is transferred, all bonds with **non-transferable** rule must break. To accomodate this feature, the structure of hooks needs an update to include the **rules** for them:

<pre>
mapping(bytes32 => <b>mapping(uint8 => address[])</b>) _hooks;
</pre>

where `uint8` keeps track of the rules for each hook. In the simplest implementation when there are only two rules (transferable or otherwise), `bool` is a more natural choice. In order to allow for multiple rules however, we choose `uint8` (`non-transferable = 0`; `0 < transferable <= 2^8`).


## Contract ABI

#### ownerOf()

Function: `ownerOf()` : <span style="color:blue"> returns the owner of a name</span>
Syntax: `function ownerOf(bytes32 namehash) returns (address);`

#### transfer()

Function: `transfer()` : <span style="color:blue"> transfers name to new owner</span>
Syntax: `function transfer(bytes32 namehash, address newOwner);`

#### resolver()

Function: `resolver()` : <span style="color:blue"> returns the resolver for a name</span>
Syntax: `function resolver(bytes32 namehash) returns (address);`

#### setResolver()

Function: `setResolver()` : <span style="color:blue"> sets the resolver for a  name</span>
Syntax: `function setResolver(bytes32 namehash, address resolver);`

#### getBondage()

Function: `getBonds()` : <span style="color:blue"> returns the number of bonds in format `[in, out]` for a name</span>
Syntax: `function getBondage(bytes32 namehash) returns (uint[](2));`

#### getBond()

Function: `getBond()` : <span style="color:blue"> returns a bond</span>
Syntax: `function getBond(bytes32 bondhash) returns (bytes32);`

#### bond()

Function: `bond()` : <span style="color:blue"> creates a new bond and returns its hash</span>
Syntax: `function bond(bytes32 from, bytes32 to, bytes32 alias) return (bytes32);`

#### getHookCount()

Function: `getHooks()` : <span style="color:blue"> returns the number of hooks in a bond</span>
Syntax: `function getHookCount(bytes32 bondhash) returns (uint);`

#### getHook()

Function: `getHook()` : <span style="color:blue"> returns a hook</span>
Syntax: `function getHook(bytes32 hookhash) returns (address);`

#### hook()

Function: `hook()` : <span style="color:blue"> creates a new hook in a link with label</span>
Syntax: `function hook(bytes32 bondhash, bytes32 labelhash);`

#### unbond()

Function: `unbond()` : <span style="color:blue"> breaks a bond</span>
Syntax: `function unbond(bytes32 bondhash);`

#### rehook()

Function: `rehook()` : <span style="color:blue">rehooks a hook to a new link</span>
Syntax: `function rehook(bytes32 hookhash, bytes32 newBondhash);`

#### unhook()

Function: `unhook()` : <span style="color:blue"> returns `true` after unhooking a hook; returns `false` otherwise</span>
Syntax: `function unhook(bytes32 hookhash) returns (bool);`

#### unhookAll()

Function: `unhookAll()` : <span style="color:blue"> returns `true` after unhooking all hooks inside a bond; returns `false` otherwise</span>
Syntax: `function unhookAll(bytes32 bondhash) returns (bool);`



