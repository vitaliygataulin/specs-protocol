`GEO Protocol / 2018` &nbsp;&nbsp;&nbsp;  `Release Candidate` **`Audit Pending`**

![Twin Spark Logo](https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/twin_spark.png)

Fast, safe, double-spending resistant, quantum-resistant, multi-attendee  
crypto-protocol for atomic and consistent assets transfers via p2p networks.
<br/>
<br/>
<br/>
[`Max Demiyan`](https://github.com/MaxDemyan)
[`Dima Chizhevsky`](https://github.com/haysaycheese/) 
[`Mykola Ilashchuk`](https://github.com/MukolaIlashchuk) 
<br/>
<br/>
_Contributors:_
<br/>
[`Denis Vasilov`](https://github.com/Vasilov345)
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
# Abstract
This specification describes transactions processing algorithm _("Algorithm"_ further in the doc.) for the [GEO Network](https://github.com/GEO-Project). Proposed solution provides ability for up to several hundreds participants _[※ 1]_ to achieve **100% consensus** via communications through potentially unstable and untrusted environment, under active fraud attempts of malicious participants.

<br/>

---

* _[1]_ — Up to `(2 ** 16)-1` in theory.


<br/>
<br/>
<br/>
<br/>

# Overview
The objectives of this document are:
1. to provide comprehensive info about proposed method of consensus;
1. to describe [guaranteed double spending impossibility](https://github.com/GEO-Project/specs-protocol/blob/master/transactions/transactions.md#double-spending-prevention), and distributed locks mechanics;
1. to describe the [cryptographic primitives used](https://github.com/GEO-Project/specs-protocol/blob/master/transactions/transactions.md#cryptographic-primitives) and the provide motivation for including each of them into the protocol;
1. to describe the method of mathematical and cryptographic confirmation of operations (consensus checking);
1. to prodive mathematical confirmation of the impossibility (or extreme complexity) of the operations compromising [todo];
1. to provide a list of possible edge cases and to describe the ways to avoid/resovle them, as well as possible outcomes of operations.

<br/>
<br/>

# Source Conditions and Requirements
This section lists the functional and design requirements for the proposed algorithm — the metrics, that were used to analyze all found and potentially applicable solutions for their applicability in to the final protocol.

**1. Requirements for cryptographic primitives:**
  1. _Quantum-resistant cryptography._  
  Algorithm must be avare of usage of cryptographic solutions, which are potentially [easily compromised in quantum-based environment](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography) (RSA / ECDSA, and other solutions that are based on similar mathematical problems).  
  
  1. _Strict minimum of crypto-primitives._  
  Algorithm must use strict minimum of the crypto systems. The role of each of them must be strictly defined and clearly motivated. 
  
**2. Requirements for operations:**
   1. _Strict 100% consensus._  
   Algorithm must enforce achievement of 100% consensus (with possibility of subsequent verification of it) between all  participants involved: the operation itself must be considered as conducted only if all participants (nodes) has approved it. Otherwise - operation must be cancelled also on all participants devices. Achieved consensus must be mathematically approvable and verifieble by any participant of the operation.
   
  1. _Deferred atomicity._  
  For each one operations launched in the network, all nodes involved in it, must achieve final state, or discard the operation at all. Intermediate states are acceptable only in short period of time, but are unacceptable to be permanent. Achieving atomicity should be:
    1. time-predictable (ideally about a few seconds).
    1. safe towards other operations (launched in parallel).
    1. agnostic towards nodes internal code modifications (hacking) and destructive motivation of the involved participants (fraud attempts).
    
  1. _Predicted time window._    
  Transaction time window — `1-15` seconds for non-fraud operations, and up to `~20 minutes` [#6] for processing problematic operations. It is expected, that more than 90% of all operations in the network would be non-fraud. This assumption is only related to the estimated network throughput.

  1. _Routes agnostic._
  Transactions must be able to use up to several hundreds of different payment paths. [# todo: provide link to the Routing specification].

  1. _Token / Commisions independence._  
  Transactions must be able to proceed without any internal token. Participants must not be enforced to charge any commissions for the middle ware operations processing.


**3. Requirements for end-point devices (nodes):**
  1. _Applicability for modern smartphones._  
  Algorithm should be usable in environments with limited computing resources and memory. Operations that consume a large amount of resources (for example, operations with significant number of participants), should be easy to process partially (pipelining).
  
  1. _Computational efficiency._  
  Algorithm should have low computational complexity. The mechanism for achieving consensus must avoid frequent calling of complex cryptographic operations (as, for example, Proof Of Work mechanics assumes in some blockchain-based solutions).
  
**4. Requirements for anonymity of operations:**
  1. _Network addressation agnostic._  
  Algorithm must be agnostic on addressation mechanics, so the participants might be present by any kind of addresses (IP / any kind of name resolutions services / prosy services, etc). As a consequence - algorithm should support anonimization via tor and other similar networks.
  
  1. _Lack of a common ledger and common operations history._
  The outcome of the operation must be determined and distributed only within the participants of this operation. The remaining nodes of the network, who do not participate in the operation, must not be able to retrieve the data about operation, and/or its content.
  
**5. Requirements for network resources:**
  1. _Resistance to unstable networks._  
  Algorithm is resistant to network interference, packet loss and/or even whole messages loss. In the worst case, if consensus can't be reached, the operation must be canceled on all devices participating in it.
  
  1. _Transport protocol agnostic._  
  Algorithm must not require a permanent connection and must not base its own mechanics on the guarantees provided by different protocols, starting with the transport layer of the OSI model (for example, TCP). The reference implementation of the algorithm is based on UDP (see network protocol specifications for the details) [※ 1].
  
**6. Requirements for fault tolerance:**
  1. _Strict operation completion guarantee_  
  Algorithm in conjunction with the rest of the GEO protocol ecosystem solutions, must be ablr to ensure the finality of any running operation on the network. Scenarios under which the operation was started, but can not be completed (dead locks) must be striclty documnted and excluded.
  
  1. _Automatic synchronization._  
  Algorithm must be able to automatically resolve possible conflicts between participants of the network, leading them back to the synchronization state, via strit logic of cryptographically-based solutions.
  
  
**7. Requirements for the speed of operations:**
  1. _Sub-second processing time for one participant._  
  This point depends very much on the configuration of the separately taken node, but in general, the GEO Protocol team aims to create an algorithm that is as effective in time as possible. This approach is reflected in a number of optimizations adopted in the algorithm, and continues to find a place in the new solutions.
  
**8. Requirements for portability:**
  1. The algorithm must not contain platform-dependent components.
  1. The algorithm must not be hardware specific or dependent.
  
  
※ To increase the probabilty of achieving consensus, and / or shorten the time of the algorithm execution, various traffic optimisation and correction techniques can be used as well. This protocol does not assumes any of them to be present, but obviously works better in the environments with stable network connections.

<br/>
<br/>

# Double Spending Prevention
Due to proposed [network topology, based on trust lines / channels]() [#todo: add link to trust lines protocol] — there is no common balance (or ledger) of the node. There are only relations of the node with its neighbours (other nodes in the network), and balances of this relations. There is no single point of assets storing. There are only relative balances of the node. As a result — there is no possibility to perform so called "double spending" towards several neighbours or remote nodes at a time. There is only a possibility to try to cheat against some neighbour and to try to force it to use some debt twice or more. But, there is no way to do it without approval (cryptographic sign) of that node, and obviously remote node would not be agree to approve such kind of malicious operation. 

To help nodes prevent double spending attempts — so called "distributed lock" is used. It is a separated component with a  simple map of type `node` → `total reserved amount` in its core. This map **must** be permanent and **must** be atomically updated and restored between each one node restarts, so each one operation on the node would have amount reservations related to it. 

On each one transaction, node **must** check this component and enshure that current `total reserved amount` on the requested trust line / channel is **less** than `(total available amount) - (reservation request)`. In other words, this check **must** enshure node would not reserve more amount than is available by the trust line / channel.

This check is **required part of each one operation.**  
(See [Stage 1 — Amount collecting and reservation](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-1--amount-collecting-and-reservation) for the details). 

While node follows this specification and it's internal behaviour was not modified (hacked) — there is no way to force remote node to accept more debts, than it was envisaged by the trust line / channel.

<br/>
<br/>


# Cryptographic Primitives
This section describes the used cryptographic primitives and the motivation for their inclusion in the protocol.

## Lamport Signature
[_Lamport signature_](https://en.wikipedia.org/wiki/Lamport_signature) or _Lamport one-time signature scheme_ is a method for constructing a digital signature. Lamport signatures can be built from any cryptographically secure one-way function; usually a cryptographic hash function is used _[※ 1]_. In Twin Spark, Lamport's signature is based on [_BLAKE2b_](https://blake2.net/).

<br/>

---

_[1]_ — For each hash function that generates an n-bit digest, the ideal resiliency to restore the prototype and to restore the second prototype implies `2n` operations and `2n` bits of memory for each one execution of the hash function in the classical computational model. Using the Grover algorithm, restoring the pre-image of the ideal hash value is bounded from above by `O (2n / 2)` operations in the quantum computational model. At the moment, Lamport's signature is considered one of the most reliable digital signature algorithms, proved to be resistant to attacks using quantum computing.



### Limitations Related to Lamport Signature
Lamport Signature is a one-time scheme (each one signing procedure discloses part of the private key). Because of that `n` key pairs are needed for `n` operations processing. Thus, each one pair of network members, who has common trust line(s), or channel(s), must generate a pool of keys, to be able to sign operations, before any other operations would be possible to process. By default, this pool contains [`1024`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#keys-pool-size) key pairs. Detailed descriptions of the key exchange mechanics and channels establishment can be found in the trust lines specifications `[todo #5: provide link to TL specifications]`.

### Difficulties of Applicability in Existing Solutions
A pair of Lamport keys (Private key and Public key) takes `32 kB`:
* `16 kB` — [Private key](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#particpant-private-key).
* `16 kB` — [Public key](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#particpant-public-key).
* `8 kB` — [Signature](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participant-signature).

The size of signatures and related keys genarated via the Lamport Signature scheme, are a way more expensive than corresponding sizes of signatures and keys, which are based on classic async. cryptograpty solutions (RSA/ECDSA/etc). The relatively large size of signatures makes the Lamport crypto system a very unlikely candidate for the inclusion in existing blockchain solutions in the foreseeable future due to a significant increase in the ledgers growth rate. Also, Lamport's signature, has no practical sense in systems whose cryptographic strength directly depends on components built on classical approaches and are known to be vulnerable against quantum-based attacks. For example, the inclusion of the Lamport cryptosystem in solutions built on top of ethereum / bitcoin, or other popular blockchain solution, has no practical meaning due to the underlying potential vulnerability of the central ledgers of these systems.


### Motivation for inclusion in the protocol
Using of the Lamport signature significantly improves the cryptographic strength of the proposed solution in the [face of potentiall attacks using quantum computations](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography). This kind of advantage is very important in the conditions of active development of computer technologies and constant increasing of the probability of the threat of classical cryptographic-systems with a public key. 


## BLAKE2b
[BLAKE2](https://blake2.net/) is a cryptographic hash function faster than MD5, SHA-1, SHA-2, and SHA-3, yet is at least as secure as the latest standard SHA-3. BLAKE2 has been adopted by many projects due to its high speed, security, and simplicity. 

### Motivation for inclusion in the protocol
Along with other finalists of the NIST SHA-3 contest, it has proven and reliable cryptographic stability, but it is much faster than SHA-3 (keccak). Unlike keccak, blake2 is built on to of classical approaches for hash-functions, so its internal structure is more studied for various kinds of vulnerabilities.

<br/>
<br/>

# Assumptions
1. Algorithm expects that neighbor nodes (nodes that are connected via trust line / channel) have secure communication channel between each other (common shared secret). Please, see [Trust Lines;]() [#todo: add link] specifications for the details about achieving this.

##### Related specs
* [Trust Lines;]() [#todo: add link]
* [Economic model;]() [#todo: add link]

<br/>
<br/>


# Protocol Decription
## Overview
Lets assume that there is a network with 4 attendees `{A, B, C, D}` involved into the next topology:

<img width=400 src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart1.svg">

```mermaid
graph LR;
    A-- 50 -->B;
    B-- 100 -->C;
    C-- 200 -->D;
```

* _`A` trusts / opens channels with `B` for `50` (accounting units of some value);_
* _`B` trusts / opens channels with `C` for `100` (accounting units of some value);_
* _`C` trusts / opens channels with `D` for `200` (accounting units of some value);_

The base task looks simple:
* to provide ability for the `D` to send `50` accounting units to the `A` through nodes `{C, B}`;
* to provide strong guaranties for the `A` and `D` about transaction's state: trunsaction **must** be finally approved or rejected by all participants, and **must not** change it's state any more in future, so `A` would be able to react on it (for example, ship some goods to `D`).

For nodes `{B, C}` it is very important to be shure that transaction would not be partially committed (committed on some trust lines, and not committed on the whole path).

The challenge itself is: 

---

_to provide information about **final** state of the transaction to all the participant in **secure Byzantine-resistant manner, and in predictable time window**_.

---

## Roles
#### Coordinator 
_Coordinator_ — node, that initiates the transaction (node `(D)` in example scheme).  
Coordinates all other nodes, involved into the operation during its life cycle.  
_Coordinator_ utilizes significantly more network traffic in comparison with other participants, but it has economic motivation to do so — it wants to transfer assets to some node and is ready to "pay for it by its resources";
</br>
</br>

#### Receiver
_Receiver_ — node that receives the transfer (node `A` in the example scheme).
</br>
</br>

#### Middleware node
_Middleware node_ — node that takes part into the operation as common participant, and provides its trust lines / channels as transport for assets (nodes `C` and `B` in the example scheme);
</br>
</br>

#### Observers
_Observers_ or _Observers chain_ — is special institute for disputes resolving. Please, see the [Observers]() [#todo: add specification] specification for the details.

## Terms

#### Nodes involved
_Nodes involved_ — `nodes_inv` — list of nodes, that are involved into the transaction. 
Depending from the context, might represent (1) _all nodes of the transaction_, or (2) _all other nodes of the transaction_.

#### Neighbors involved
_Neighbors involved_ — `neighbors_inv` — list of neighbor nodes, that are involved into the operation, is a subset of [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved)

#### Transaction amount
_Transaction Amount_ — `tr. amount` — amount of accounting units that [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) tries to send to the [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver);

#### Least common amount
_Least common amount_ — common reservation amount, that might be reserved by _all_ nodes in the path.  
Least common amount == max. payment flow of the path. Please, see [Maximum flow problem](https://en.wikipedia.org/wiki/Maximum_flow_problem) for the details.

#### Network Path
_Network Path_ — vector _V_ of nodes, that are chained by the trust lines / channels into one path.  
Path is considered as valid only in case if there is a possibility to traverse all nodes from the source node to the destination node (one by one).  

</br>

_V = [sourceNode, node1, node2, ..., destinationNode]_

<img width=400 src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004170830.svg">

_Example of network path: {A, B, C, D} and its subsets {A, B, C}, {A, B}, ..._

<img width=400 src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004171342.svg">

_Example of payment network path_

</br>

#### Paths map
_Paths map_ — coordinator-specific internal data structure for storing information about all reservations created on all paths used. During amount reservation, each one newly created amount reservation always would be less (or equal) to previously created reservation on the same path. This structure helps maintain path topology and help nodes to achieve _least common amount_ reserved.

#### Signatures list
_Signatures list_ — list of all participants of the operation and their signatures.  
Presence of all signatures approves 100% consensus and must be interpret as finalised operation. 

<br/>
<br/>

# Stage 1 — Amount Collecting and Reservation
#### Paths discovering
[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) in cooperation with [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver) **must** discover all (or some part of) possible [network paths](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-path) `{Coordinator -> Receiver}`. In case if no paths are found — _Algorithm_ execution **must** stop with the error code [`No routes`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#no-routes). Please, see [Routing]() `[#todo: link]` for the details on paths discovering.

#### Optimisations
* [Paths Processing] Routing algorithm [todo: link] migh return first portion of available paths much faster, than it needs for discovering whole list of all possible paths. For the transaction processing, only several paths might be needed, and, depending on the transaction amount, there is a non-zero probability, that several first portion(s) of paths would be enough for the operation to finish. Transactions Algorithm should not wait for discovering all the paths, but should begin even if one (or several) paths are available. In case if discovered paths would be not enough — Algorithm should pause for some time, until next portion of paths would be available.


#### Paths processing
[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** attempt to reserve [`tr. amount`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-amount) to the [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver) on several (or all, if needed, but at least one) discovered paths.

## Algorithm A: Parrallel reservations collecting 
_Advantages — high reservations speed._  
_Disadvantages — high liquidity locks._  

##### [Coordinator](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator)
1. ∀{`path` ∈ `paths_discovered`}:
    1. ∀{`node` ∈ `path`} :
        1. In case if `node` != [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver) — [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** ask `node` to reserve max. available _common_ amount between 2 neighbors and return it (max. common amount) back.  
        In case if `node` == [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver) — [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** ask it to reserve max. available amount to the neighbour _before_ it (penultimate тщву), and also return reserved amount back to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).
    1. **Must** collect _all_ reponses from _all_ nodes. In case if some node has not reponded during [2 network timeouts](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout) — `path` **must** be omitted. Node, that has not responded, must be considered as "unavailable". All paths with this node included **must** be dropped from the processing. [Optionall] [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) should send reservation cancel to _all_ nodes in this path (except node, that has not sent it's response).
    1. **Must** calculate [least common amount](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#least-common-amount) between all participants in the `path`.
    1. **Must** provide [least common amount](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#least-common-amount) to all nodes in `path` (except itself), and ask them to shortage their reserves in accroding to it. 

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181003154119.svg">

```mermaid
sequenceDiagram
        Coordinator (D)->>B: Reservation Request (D, C)
        Coordinator (D)->>C: Reservation Request (B, A)
        Coordinator (D)->>Receiver (A): Reservation Request (C)
```

##### All [middleware nodes](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) and [Receiver](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver)

1. On reservation request received, node **must** calculate [least common amount](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#least-common-amount) against/between participant(s), specified in the request.

1. **Must** send calculated least common amount back to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181003154552.svg">

```mermaid
sequenceDiagram
        B->>Coordinator (D): Least Common Amount
        C->>Coordinator (D): Least Common Amount
        Receiver (A)->>Coordinator (D): Least Common Amount
```

In the rest mechanics of this approach is similar to the "Algorithm B", provided further.

## Algorithm B: Sequential reservations collecting
_Advantages — low liquidity locks._  
_Disadvanteges — moderate reservations speed._  

##### [Coordinator](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator)
1. ∀{`path` ∈ `paths_discovered`}:
    1. _FN_ — first neighbour — second node in the path.
    **Must** calculate max. possible amount reservation on the trust line with _FN_;
    1. **Must** send Reservation Request to the _FN_;
    1. **Must** wait for the reponse from _FN_ for no more than [2 network timeouts](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout). In case if no response has been received during this time window — remote node **must** be considered as "unavailable". **All paths** with this node included **must** be dropped from the processing. If there are nodes, that has confirmed reservation request, then this nodes should receive reservation cancel request from the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).

##### All [middleware nodes](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) and [Receiver](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver)

Each middle-ware node `{C, B}` and `Receiver (A)`, on reservation request received, **must** check possibility to approve the request and **must** do one of 3 things possible:

1. If there is enough free amount on requested trust line — **must accept reservation fully**:
    1. **Atomically** create [amount reservation](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#double-spending-prevention) on the trust line with specified neighbour and for the specified amount;
    1. Set timeout for the created reservation to [default reservation timeout](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#amount-reservation-timeout).  
    Created timeout avoids eternal reservation and is used for canceling the operation, in case of occurred unpredictability.
    1. Send [reservation response](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#response-amount-reservation) with approved amound to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator);
  
1. If there is some free amount on requested trust line present, but it is less, than required — **must accept reservation partially**:
      1. **Atomically** create [amount reservation](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#double-spending-prevention) on the trust line with specified neighbour for the **available** amount.
      1. Similarly, to the previous case, sets timeout for the created reservation to [30 seconds](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#amount-reservation-timeout).
      1. Sends [reservation response](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#response-amount-reservation) to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator), but with amount reserved (less than was requested);

1. In **all** other cases — **must reject reservation** — send [reservation response](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#response-amount-reservation) with 0 to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator). Several, _but not all_ cases possible, are:
    1. no more/any reservation is possible;
    1. node received reservation request towards some other node, that is not listed as it's neighbour;
    1. node received reservation request that contains info about reservations, that was not done by this node;

#### Notes
* It is possible, that some node would be requested by the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) several times, to create several different reservations towards several different neighbours. This case is probable, when some middle ware node is present on several concurrent payment paths at the same time.
    <img width=400 src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart10.svg">

* Created reserves are only temporary locks on the trust lines, and must not be considered as committed changes (debts). This locks are important mechanism for preventing usage of greater amount of trust line / channel, that was initially granted, so it is expected that each one node would very carefully account this reservations.

#### Warnings
* Trust lines reservations and related transactions must be implemented in [**ACID** manner](https://en.wikipedia.org/wiki/ACID_(computer_science)).  
Transactions and related {trust lines, channels} locks must be restored after each one unexpected node failure and/or exit.

* Each one payment transaction, that was restored after node failure, must be automatically continued from [Stage Z: Recover](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-z-recover). 

```mermaid
sequenceDiagram
    Coordinator (D)->>C: Reserve 200
    C->>B: Reserve 200
    B->>C: OK, only 100 reserved
    C->>Coordinator (D): OK, only 100 reserved

    Coordinator (D)->>B: Reserve 100
    B->> Receiver (A): Reserve 100
    Receiver (A)->>B: OK, only 50 reserved
    B->>Coordinator (D): OK, only 50 reserved
```
<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart2.svg">

From [middle-ware nodes](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) perspective, processing of the amount reservation requests could be schematically explained in the next way:
   
<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart6.svg">
   
```mermaid
sequenceDiagram
      Coordinator (D)->>C: Reserve 200
      C->>B: Reserve 200
      B->>C: OK, reserved 100 (without sign yet)
      C->>Coordinator (D): OK, only 100 reserved
```   

1. Node `C` attempts to reserve required amount on its side first. 
  1. If no required amount may be reserved — node responds with reject.
  1. If reservation is possible and was done successfully — node suspects for successful reservation on the neighbour node as well (node `B` in the example), and sends the appropriate request to it. In case of received confirmation response from the neighbour node — `C` reports "success" to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).

### Amount reservations processing
1. On response from node, `Coordinator (D)` **must** check the response:
  * In case if reservation was "approved" — `Coordinator (D)`:
      1. Updates it's internal `paths map`;
      1. (re)calculates (probably shortated) least common reservations;
      1. In case if node, that sent `approve` to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) is the last node in the path — then [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) sends current reservations configuration (list of neighbours nodes and amount reservations towards them) to all nodes, in this path, including `Receiver`. This info is very important, because there is a non-zero probability of the case, when some path from a collection of discovered paths, would be dropped during amount reservation (for example, because some trust line on it has no enough amount). In this case, it is very probable, that some amount reservations towards some nodes would not be needed any more, but the others would stay necessary. It is even possible, that this configuration changes would take place on the same node. That's why [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) reports whole new amounts configuration state to all nodes.
    * In case of `reject` — `Coordinator (D)` drops this path at all from payment, and starts collecting amount on one of other available paths. During path dropping — it also informs all the nodes, that are involved into it, about dropped reservations, via sending  current reservations configuration to them.
    
1. In case if all paths are processed, but no required amount was collected — [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** suspend current operation and (re)try to collect more paths from the network. `[todo: describe paths complementing process]``[todo: routing]`.

1. In case if `collected amount == needed amount` — stage 1 is considered as complete.


### Reservations shortening
**Note:** All middleware nodes (`{(C), (D)}`) and `Receiver` (`(R)`) are motivated to update their reservations as quickly, as possible, because each one reservation present freezes some part of their liquidity. It is expected that some part of nodes, from to time, during operation processing, would be asked by the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) (`D`) to update their reservations to achieve _least common reservation amount_. From [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) prespective this process is sequntial, so it might take some time to update all nodes and find _least common reservation amount_. From the nodes perspective — this process requires some timeout after initial reservation creation and furter updates.

### Reservations prolongation
During processing of long paths, or significant amount of paths in total — there is a non-zero probability, that some nodes one, or several times would fall in state, when their reservation timeouts would be burned up, but the transaction itself would be still in amount collecting stage. In this case, to prevent transaction disruption —it is reccomended for the this nodes to ask [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) about current transactino state to be able to know what to do with their active reservations. 

#### Case 1: Transaction is still in progress
* [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** report transaction state via [Transaction State Message](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#response-transaction-state).
* `Node`, that requested the state **must** prolong **all related** reservations for up to [30 seconds](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#amount-reservation-timeout)

```mermaid
sequenceDiagram
    B->>Coordinator (D): Is TA (ID) still alive?
    C->>Coordinator (D): Is TA (ID) still alive?
    Receiver (A)->>Coordinator (D): Is TA (ID) still alive?

    Coordinator (D)->>B: Yes, wait some more time
    Coordinator (D)->>C: Yes, wait some more time
    Coordinator (D)->>Receiver (A): Yes, wait some more time
```
<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart3.svg">
   
In case if [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) responds with "active" state — it is recommended for the node to keep processing of the transaction. In this case node simply reinitialize it's internal timeout and keeps waiting. This stage might be repeated several more times. It is important for middle-ware nodes to wait as long as [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) asks to, otherwise - whole transaction would be dropped. [#8](https://github.com/GEO-Protocol/specs-protocol/issues/8)

**WARN:** There is an ability for the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) to hang reservation for a long time and harm network liquidity. To avoid this — node must keep counting of prolongations done. In case if next one prolongation begins to be uncomfortable for the node — it might simply reject the transaction `[Stage B]`. Max amount of prolongations should be decided by each one node for itself, based on it's internal trust to the coordinator. Recommended amount of prolongations is `6` (3 minutes).  

#### Case 2: Transaction has not collected required amount

* [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** report transaction state via [Transaction State Message](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#response-transaction-state).
* `Node`, that requested the state, **must** process the reponse and in case if [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) responds with state that differs from "active" (for example, because required amount could not be collected on discovered paths) — `node` must reject the transaction [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject). Amount reservations might be dropped safely at this stage.

```mermaid
sequenceDiagram
    B->>Coordinator (D): Is TA (ID) still alive?
    C->>Coordinator (D): Is TA (ID) still alive?
    Receiver (A)->>Coordinator (D): Is TA (ID) still alive?

    Coordinator (D)->>B: No, abort it!
    Coordinator (D)->>C: No, abort it!
    Coordinator (D)->>Receiver (A): No, abort it!
```
<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart4.svg">

#### Case 3: No coordinator response at all
In case if no response was received from the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) — one of 3 things might take place:
  1. [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) goes offline unexpectedly and/or is unable to proceed.
  1. Network segmentation takes place and no network packets are delivered/received to/from the node/[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).
  1. [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) behaves destructively and doesn't responds to the nodes requests.

```mermaid
sequenceDiagram
    B->>Coordinator (D): Is TA still alive?
    C->>Coordinator (D): Is TA still alive?
    Receiver (A)->>Coordinator (D): Is TA (ID) still alive?

    Coordinator (D)-->>B: (?)
    Coordinator (D)-->>C: (?)
    Coordinator (D)-->>Receiver (A): (?)
```
<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart5.svg">

In all cases, it is safe for any node to drop the transaction and it's related amounts reserves [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).  
  
<br/>
<br/>
  
# Stage 2 — Final Res. Conf. (coordinator)
##### Definitions
[Final Reservations Configuration]() [#todo: add structure] — _FCR_ — structure that represents relations and their states (reservations). Separated _FCR_ must be generated for each one node involved for the syncronisation purposes. 

##### Flow
1. **Must** finalize it's paths map and generate _FCR_  for ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved).
1. **Must** send _FRC_ to ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved). 

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004130140.svg">

<br/>
<br/>

# Stage 2 — Final Res. Conf. (nodes)
##### Definitions
* Internal Trust Lines — _ITL_ — list of trust lines (or channels), involved into the transaction. 

##### Flow
1. **Must** wait at leat 2 [network hops](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout).
1. **Must** receive Final Reservations Configuration Message (_FRC_) [#todo: add description] from the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).    
  * If no _FRC_ was received — node **must** reject the operation [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
  * If _FRC_ was received — node **must** validate it throught the checks provided further. 
      * If any of this checks fails — **must** reject the operation [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
      * IF all checks pass — **must** forward to Stage 2.1.

##### Checks for _FRC_
1. ∀`trust_line` ∈ `FRC.trustLines`:
  * `trust_line.amount` = _RA_,  
    where _RA_ — reserved amount on related trust line _on the node_.  
    If `trust_line.amount` < _RA_ — node **must** shortage reservation on related trust line (decrease _RA_).
1. ∀`trust_line` ∈ `ITL`:
  * If `trust_line` is not present in `FRC` — reservations on `trust_line` **must** be dropped ([`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) decided to not to use this channel).  

<br/>
<br/>

# Stage 2.1 — Signed debts receipts exchange
1. On each final configuration received, `node` must:
  * ∀`neighbor` ∈ [`neighbors_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#neighbors-involved)): 
    * `node` **must** create debt receipt (#todo: link to struct) for the `neighbor` with amount according to the reserved amount on the trust line with the `neighbour`.
    * `node` **must** sign it with one of it's public keys from pool of public keys with `neighbour`.
    * `node` **must** send signed debt receipt to the `neighbor`. 
      ```mermaid
      sequenceDiagram
          C->>D: [Signed debt receipt]
          B->>C: [Signed debt receipt]
          A->>B: [Signed debt receipt]
      ```
      <img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chartDebtsReceptExchange.svg">

**Note:** Signed debt receipt is only valid in case if whole transaction is signed (separate message with separate signature), so node might sign and send it to the neighbour without any doubt. In case if any other node would not sign the operation — no one debt receipt would be valid and must not be demanded.

∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved)+ [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator)):
  * `node` **must** receive signed debt receipts from **all** neighbours. In case if even one signed debt receipt wasn't received — node **must** reject the operation [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
  * `node` **must** check **all** received signed debt receipts for the next reuirements:
    
Stage 2.1 is considered as completed when all nodes would exchange signed debt receipts with each of it's neighbours involved into the operation.

#### Checks for the signed debts receipt
1. `receipt.amount`**must** be equal to reservations, created on the related trust line.
1. `receipt.transactionID` **must** be equal to current Transaction ID.
1. `receipt.equivalentID` **must** be eqaul to the equivalent ID of the reservations, on the related trust line.
1. Signature of the receipt **must** be used from common pull of PubKeys, that was established between the nodes prveiosly (see [Trust Lines]() [#todo: provide link] specification for the details).
1. There are no signed debt receipts for this operation already present on the node.

<br/>
<br/>

# Stage 2.2 — Public keys exchange (node)

##### Definitions
* [PublicKeyMessage]() [#todo: add structure] — _PKM_ — message, that contains Public Key of the node, which would be used by the node for the transaction signing.

* [PublicKeyHashMessage]() [#todo: add structure] — _PKHM_ — message, that contains **hash** of Public Key of the node, which would be used by the node for the transaction signing. This message is used for network traffic optimisations purposes. Content for the message **must** be generated by [HASH](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#blake2b)(`node.PubKey`).

##### Flow
1. **Must** send _PKM_ to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).
1. **Must** send _PKHM_ to ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved).

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004134734.svg">

```mermaid
sequenceDiagram
    B->>Coordinator (D): PubKey
    B->>Receiver (A): PubKey Hash
    B->>C: PubKey Hash
```

---

</br>

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004135048.svg">

```mermaid
sequenceDiagram
    C->>Coordinator (D): PubKey
    C->>B: PubKey Hash
    C->>Receiver (A): PubKey Hash
```

---

</br>

<img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/20181004134923.svg">

```mermaid
sequenceDiagram
    Receiver (A)->>Coordinator (D): PubKey
    Receiver (A)->>B: PubKey Hash
    Receiver (A)->>C: PubKey Hash
```

##### Optimisations
* ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved) might ommit sending of "PubKey Hash" to the neighbors nodes in this stage, but include the hash into the [SignedDebtReceiptMessage]() [#todo: add structure] on [Stage 2.1](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-21--signed-debts-receipts-exchange). This makes it possible to save one message round trip for each neighbor involved and several bytes of messages headers for each message.

<br/>
<br/>

# Stage 2.2 — Public keys exchange (coordinator)
##### Flow
1. **Must** wait at least 2 [network hops](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout).
1. **Must** receive [PublicKeyMessage]() (_PKM_) [#todo: describe the structure] from all [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved). If not _PKM_ was received _even from one_ node — **must** reject the transaction [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
1. **Must** check _PKM_ through the checks, provided further. 
  If _even one_ check doesn't pass — **must** reject the transaction [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
1. **Must** generate [PublicKeysListMessage]() [#todo: add structure] (_PKLM_) and send it to ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved).

##### Checks for PKM
1. _PKM_ contains public keys of _**all** neighbours nodes, of the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator), that are involved into the operation; (`neig. PKl`);_
1. [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** check **each one** key from `neig. PKl` for validity through next checks:
  * Key length **must** be `16Kb`;
  * Key must be included

##### Optimisations
* [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) should not inlude into _PKLM_ the pub. key of the _PKLM_ addressee (remote node knows its pub. key). 

<br/>
<br/>

# Stage 2.3 — Trust context checking (node)
##### Flow
1. **Must** receive [PublicKeysListMessage]() (_PKLM_) [#todo: describe the structure] during 2 [network hops](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout). If _PKLM_ was not received — node **must** reject the operation ([Stage B](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject)). 
1. **Must** check _PKLM_ through checks provided further.  
  If even one check fails — **must** reject the operation ([Stage B](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject))

##### Checks for PKLM
1. _PKLM_ contains all nodes from [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved) (except pub. key of current node). 
1. ∀ `record` ∈ _PKLM_:
    1. [_HASH_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#blake2b)(`record.pubKey`) == _H_,  
    where _H_ — corresponding pub. key hash, received on Stages [[2.1](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-21--signed-debts-receipts-exchange) / [2.2](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-22--public-keys-exchange-node)]

<br/>
<br/>


# Stage 3.1 — Signing prepearing (coordinator)
1. **Must** generate [Participants Public Keys List _(PPKL)_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participants-public-keys-list) for each one participant included into the operation;
1. **Must** send related _PPKL_ to all participant involved.
    ```mermaid
    sequenceDiagram
        Coordinator (D)->>B: PPKL
        Coordinator (D)->>C: PPKL
        Coordinator (D)->>Receiver (A): PPKL
    ```
    <img src="https://github.com/GEO-Project/specs-protocol/blob/master/transactions/resources/chart8.svg">

##### Optimisations
* [Traffic optimisation] [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) might exlude addressee from the [_PPKL_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participants-public-keys-list).  
  For example, in case if nodes {[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator), [`B`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node), [`C`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node), [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver)} take part into the operation, and [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) now performs [_PPKL_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participants-public-keys-list) for the [`A`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) — then [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) might exclude [`A`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) from the [_PPKL_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participants-public-keys-list) and _save network traffic for itself and for the [`A`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node)_.

<br/>
<br/>

# Stage 3.1 — Signing prepearing (node)
1. **Must** receive [Participants Public Keys List _(PPKL)_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#participants-public-keys-list) from the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).  
In case if no _PPKL_ was received — **must** reject the operation [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
1. **Must** check received _PPKL_ through the checks provided further;  
In case if _even one_ check failed — **must** reject operation [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject)
1. **Must** [sign the operation](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-32--signing-node) in case if _all checks passed_;

### Checks for PPKL:
1. ∀`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved):
    1. `node.memberID` is present in _PPKL_; 
    1. _H_ == [HASH](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#blake2b)(`n.PubKey`), where  
    _H_ — [HASH](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#blake2b)(`node.pubKey`, received on [Stage 2](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-2--trust-context-establishing-nodes)),  
    _n_ — node from _PPKL_, related by "memberID".
    
    
<br/>
<br/>

# Stage 3.2 — Signing (node)
1. **Must** serialize the operation to stable storage. It is neccesary in case if node would crash unexpectedly.
1. **Must** create [_Transaction Signature_](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature);
1. **Must** send it to the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator);
1. **Must** move to 


1. **Must** [commit](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-33--commiting) the operation.

<br/>
<br/>

# Stage 3.2 — Signs collecting (coordinator)

### Transaction Signatures
_TS_ collects transactions signatures from participants and stores received info realted to the sender. _TS_ = {
[`B`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node): [`TransactionSignatureB`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature), 
[`C`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node):[`TransactionSignatureC`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature), ... , 
[`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver): [`TransactionSignatureReceiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature)}.

### Flow
1. **Must** collect [Transaction Signatures](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature) from _all_ participants into _TS_.  
If not _all_ transaction signatures was collected during 3 [network hops](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout) _(2 hops for keys/signatures transfer and 1 for remote node proessing)_ — [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** reject the operation [Stage B](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).

1. **Must** check all received [Transaction Signatures](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature) through checks provided further.  
If even one check fails — [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must** reject the operation [Stage B](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).
1. **Must** generate TransactionConsensusMessage and send it to each one participant involved.
1. **Must** [Commit](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-34--commiting).


### Checks for TS:
* ∀(`record`, `sender` ∈ _TS_):
    1. `record.transactionID` == current [`Transaction ID`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transactionid);
    1. `record.signature` == SIG(`record`, PubK), where _PubK_ — Public Key of `sender`; 
    1. ∀(`member` ∈ `record.members`):
        1. `member.memberID` == `participant.memberID` **AND**
        1. `member.address` == `participant.address` **AND**  
          where `participant` = ∀(`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved));

<br/>
<br/>

# Stage 3.3 — Сonsensus expectation (node)
Only [middleware nodes](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node) and [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver) might fall into this stage.  
[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) is responsible for collecting transactions signatures and is able to check consensus achievent directly on its side.

##### Definitions
* Consensus Timeout — _CT_ — max. time window, node should wait for the [TransactionConsensusMessage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-consensus-message).  
By default, should be at least 2 [network timeouts](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout) long.

* [TransactionConsensusMessage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-consensus-message) — _TCM_ — message with signatures (approves) of all [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved).

##### Flow
1. **Must** wait for the _CT_;

1. **Must** receive _TCM_ from the [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator).  
In case if no _TCM_ was received — **must** move to [Recover stage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-z--recover-nodes).

1. **Must** check received _TCM_ through the checks provided further. 
    * In case if _even one_ check fails — **must** move to [Recover stage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-z--recover-nodes).
    * In case of _all_ checks has been passed — **must** [Commit](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-34--commiting).

##### Checks for the _TCM_
1. `TCM.transactionID` **must** be equal to current [Transaction ID](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transactionid);

1. ∀{`member` ∈ `TCM.members`}:
    1. `member` has corresponding by `memberID` node in [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved) (_Corresponding Node, CN_);
    1. _LamportSignatureCheck_(`member.signature`, `CN.pubKey`) -> `True`;

<br/>
<br/>

# Stage 3.4 — Commiting
Both, [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) and [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver):
1. **Must** serialize next data to the stable storage:
    * [Transaction ID](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transactionid) 
    * ∀(`node` ∈ [`neighbors_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#neighbors-involved):
        * `node.incoming_debt_receipt` (if present);
        * `node.outgoing_debt_receipt` (if present);
        
    * ∀(`node` ∈ [`nodes_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved)):
        * `node.memberID`;
        * `node.publicKey`;
        * `node.signature`;
     
1. **Must** turn all reserves into balances on all trust lines / channels, that has reservations related to the transaction.
1. **Must** drop all reserves, related to the transaction.
1. **Must** drop related serialized transaction from stable storage to prevent it restoring on node restart.

**WARN:** all operations provided in this section **must** be performed in atomic manner, otherwise — there is a non-zero probability that operation would be committed, but the serialized transaction would not be dropped, that would lead to errorneus transaction recover attempt and balances corruption as a result.

<br/>
<br/>



# Consensus pushing
There is a very small, but still non-zero probability of a case, when 100% positive consensus would appear somewhere in the network, and would not be available to the `Delegate`. As a result, `Delegate` would force operation reject and because of highest priority possible, this decision would be accepted by the nodes, even if operation is already committed on them. That leads to the operation cancellation, and to avoid some fraud attempts — this decision should be delivered to the nodes as fast as possible.

To do so, delegate will notify each of them once in a 3-5 seconds, during first 20 minutes. After that, if no notification acquiring received from the node - during next week, final decision should be propagated with exponential timeout, starting from 10 seconds up to 10 minutes between sending attempts. After that period of time delegate stops notifications attempts for participants, that are unreachable.

All decisions made by the `Delegate`:
* must be archived;
* must be available publicly. It is safe for the participants privacy to share this information, because votes list only contains final decision on the operation, but no any additional context is present in it. It even has not operation amount included, because it is unnecessary here. `todo: link to votes list structure explanation`.

# Forced consensus propagation
In cases, when some pair of nodes, that are using common trust line, has different balances on it (for example, as a result of partial data lost by some node, or data manipulation) — there is a special mechanism is envisaged — Trust Line Revise — protocol, that is based on top of collected `votes lists` and provides strong decisions for each one controversial situations possible. `todo: add link to the TL Revise`. As a result of this operation — 100% consensus might be achieved by both nodes in 100% cases.

Being applied to several neighbours, this mechanism has cascade influence and as a result — forces fair balances propagation through the network even in case of fraud manipulation with trust lines data on nodes sides.

`todo: describe it detailed`



[#todo: describe secure channel between neighbors]


# Stage A — Reject (Coordinator)
1. **Must** rollback all changes, drop all reservations and instantly forget about the operation.
1. **Must** respond "know nothing about the operation" for all requests about the operation state (it is enough for the other nodes to drop their pending reservations too after reservation timeouts).
1. Should notify othe participants about the operation canceling. [#todo: provide message structure].

</br>
</br>

# Stage B — Reject (Nodes)
1. **Must** rollback all changes, drop all reservations and instantly forget about the operation.

</br>
</br>

# Stage X — Observers Arbitration Requesting (nodes)
##### Definitions

* [ObserversRequestMessage]() [#todo: add structure] — _ORM_ — request for the arbitration, that is generated by the node in case if it has signed the transaction and has sent this signature into the network, but hasn't received [TranactionConsensusMessage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-consensus-message) and no one node responds to the request for the [TranactionConsensusMessage](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-consensus-message).

##### Flow
1. **Must** generate _ORM_ and include the next info into it:
    * [Transaction ID](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transactionid);
    * All [members IDs](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-signature);
    * All members [Public Keys]()https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#particpant-public-key.
    * All members Public Keys Hashes.
    * **must NOT** include transaction amount.
1. **Must** send _ORM_ to the [Observers]() [#todo: link];
1. **Must** check that _ORM_ is present in Observers chain. In case if _ORM_ is absent — **must** send _ORM_ once more (maybe via other observer node), until it'll appear in [Observers Chain]() [#todo: lnk].
1. **Must** check [Observers]() [#todo: link] for the TransactionConsensusMessage once in [todo: timeout]. If TransactionConsensusMessage appears in [Observers]() chain — **must** fetch it, and repeat [Stage 3.3](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-33--%D0%A1onsensus-expectation-node). If not TransactionConsensusMessage appeared during [#todo: timeout] — **must** reject the transaction [(Stage B)](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-b-middle-wares-node-behaviour-after-transaction-reject).


Delegate arbitration is used in case when there is an uncertainty about the operation processing, but some (or several) node(s) has been signed it already and sent their signs into the network. In this case, this node(s) has no ability to change it's vote(s), because:
    * nodes can't perform any reject, otherwise - it would be ambiguous. In case if operation was approved, there is no way to revert it.
    * propagating of negative sign would lead into the further consensus complexity, but in long term perspective is ineffective because of [Forced Consensus Propagation]() [#todo: link] (please, see [Trust Lines]() [#todo: link to specification] for the details ).

</br>
</br>

# Stage Z — Recover (Nodes)
During operation processing, there is non zero-probability, that some node(s) involved (or even all of them) would enter inoperable state for some reason (unexpected power off, network segmentation, etc). In this case, when operability of the node would be restored, such node(s) must proceed each one transaction that was initialized, but has no final decision yet.

##### Assertations
* [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) **must never enter** into recover state. It is responsible for collecting transactions signatures and is able to check consensus achievent directly on its side.
* Node **must** enter "Recover" state only if it has been signed the operation in the past. 
* Node **must not** enter "Recover" state if it restores it's state after crash, but the transaction itself was not signed and sent to the other participants.

##### Flow
1. **Must** restore transaction internal state (all reservations, stage identificator, members structures, etc).
1. Sequentially ∀`node` ∈ [`neighbors_inv`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#nodes-involved):
    1. **Must** ask `node` for the [`TransactionConsensusMessage`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#transaction-consensus-message) (_TCM_).
    1. **Must** wait for at least 2 [network hops](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout).
    1. **Must** receive _TCM_ from `node`.  
        In case if not _TCM_ was received — **must**:
            1. Wait 1 [network hop](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#network-hop-timeout).
            1. Skip to the next `node`.  
        In case if last `node` was processed and no _TCM_ was received — **must** generate [Observers Arbitration Request]([#todo: comment]).  
        In case if _TCM_ was received well — **must** [Commit](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#stage-34--commiting).

##### Optimisations
* Recommended order of nodes in Flow/2 should be the next:  
  [[`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator), [`Receiver`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#receiver), [`neighborNode1`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node), ..., [`neighborNodeN`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node), ..., [`remoteNode1`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node), ... [`lastRemoteNode`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#middleware-node)]


[#todo node must permanently check observers for the requests]

<br/>
<br/>

# Data types used
This section provides explanation of used data structures in developers friendly format.

### TransactionID
```c++
// It is expected that TAID
// would be randomly generated 24 bytes long sequence.
using TransactionID = byte[24];
```
<br/>


### Particpant Private Key
```c++
const uint16 kPKeyLength = 1024 * 16;
using PrivateKey = byte[kPKeyLength];
```
<br/>


### Particpant Public Key
```c++
const uint16 kPubKeyLength = 1024 * 16;
using PubKey = byte[kPubKeyLength];
```
<br/>


### Participant Signature
```c++
const uint16 kSignatureLength = 1024 * 8;
using Sign = byte[kSignatureLength];
```
<br/>

### Transaction Signature
_Transaction Signature_ is used by the nodes to approve the transaction.

```c++
struct TransactionSignatureMember {
    Address address;
    uint16 transactionMemberID;
}

struct TransactionSignature {
    TransactionID transactionID;
    uint16 membersCount;
    TransactionSignatureMember members[membersCount];
}
```
<br/>

### Particpant Address
[todo: move this into the addressation doc]

```c++
struct Address {
    byte typeID;
    byte length;
    byte[1..256] data;
}
```
<br/>

### Amount
_Amount_ specifies transaction amount or trust line amount, or channel amount.
"length" field makes it possible to transfer through the network only significant bytes, and to ignore zeroed part of the amount.


```c++
struct Amount {
    uint8 lenth;
    byte[1..32] value;
}
```
<br/>

<br/>

# Messages

#### Request: Amount reservation 
```c++
struct RequestAmountReservation {
    TransactionID transactionID;
    
    // Specifies amount, that should be reserved.
    Amount amount;
    
    // Specifies address of the node, towards which reservation must be done.
    Address neighbourAddress;
}
```

#### Response: Amount reservation
```c++
struct ResponseAmountReservation {
    TransactionID transactionID;
    
    // Specifies amount, that was reserved.
    Amount amount;
}
```

#### Request: Transaction State
[#todo: add declaration]

#### Response Transaction State
```c++
struct ResponseTransactionState {
  TransactionID transactionID;
  
  // 0 — Active — Transaction is still in progress;
  // 1 — Committed — Transaction is has bin processed now; 
  // 2 — Unknown — node knows nothing about transaction with such id;
  byte state;
}
```

#### Signed Debt Receipt
```c++
struct SignedDebtReceipt {
  TransactionID transactionID;

  // Amount of the receipt. 
  // MUST be equal to previous reservations.
  Amount amount; 

  // Index of Public Key, that was used by the node for signing current receipt;
  byte[4] transactionPubKeyIndex;

  // Crytographic sign of the receipt,
  // generated by the Public Key with positional number ==
  // (transactionPubKeyIndex - 1).
  // Please, see scheme further for the details.
  Sign sign;
}
```

#### Transaction Consensus Message
```c++
struct TransactionConsensusRecord {
    MemberID memberID;
    Signature signature;
}
```

```c++
struct TransactionConsensusMessage {
    TransactionID transactionID;

    // Total participants count, including the Coordinator.
    uint16 participantsCount;

    TransactionConsensusRecord members[participantsCount];
}
```

# Operation result codes
This section describes result codes, that are erported on the coordinator node to its interfaces (UI, API interface, etc)

#### OK
`code: 0`  
Operation was committed well. 

#### No routes
`code: 400`  
There is no any route discovered between [`Coordinator`](https://github.com/GEO-Protocol/specs-protocol/blob/master/transactions/transactions.md#coordinator) and `Receiver`;

#### Not Enough Amount
`code: 401`
There is no enough amount can be collected on all paths discovered.

#### No Consensus
`code: 402`
Some node, that participated in the operation did not provided its signature for it.

<br/>
<br/>

# Timeouts
#### Amount reservation timeout 
`30 seconds`

<br/>
<br/>

# Constants
### Keys pool size
`1024`

### Network Hop Timeout
`hop time` — time range, expected time that is needed for the network packet to be delivered to the destination node. By default should be set to `2 sec`.

### Observers verdict checking interval
_Observers verdict checking interval_ — time interval that specifies how often node should check the observers for TransactionConsensusMessage appearing. By default is set to `30 seconds`.

### Observers request checking interval
_Observers request checking interval_ — time interval that specifies how often nodes should check observers for new arbitration requests from other nodes. By default is set to `30 minutes`.

### Observers verdict timeout
_Observers verdict timeout_ — max. time that observers might wait for the TransactionConsensusMessage arriving from the network. By default is set to `2 hours`.

<br/>
<br/>

# License
[<img src="https://opensource.org/files/osi_keyhole_300X300_90ppi_0.png" height=90 style="float: left; padding: 20px">]()

**[MIT License](https://opensource.org/licenses/MIT)** (GEO Protocol worldwide team, 2018)


Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

