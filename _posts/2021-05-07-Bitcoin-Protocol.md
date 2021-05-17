---
title: "Bitcoin & Lightning Protocol (Chaincode Seminar)"
date: 2021-05-07
categories:
  - Blockchain
tags:
  - Bitcoin
  - Lightning
---

# Week-1: Intro

### What do we (Bitcoin devs/rsearchers) care about?
- Values: No trust, privacy, fungibility, scalability, soundness
- Why not just credit cards? It requires a *trusted third party* with traceability and control (to prohibit some actions)
### PBFT vs Decentralized Protocol (Cryptocurrency e.g., Bitcoin)
- PBFT requires PKI (which requires *trusted parties* managing certifications of participants) and servers who know other participants' identities (i.e., permissioned)
- What if we allow *permissionless* joining? It raises *Sybil Attack*: attacker creates many different identities to overwhelm honest nodes
- How can we guarantee the values in decentralized systems? Bitcoin (using cryptography for authentication, PoW for TXs ordering)

### Bitcoin - How to prevent...
- *Forgery*? Transaction Chain: records history of a given coin
- *Forgery of TX chain*? Cryptography: use privateKey to generate signature, use publicKey to authenticate the signature. 
Each TX in the chain is authenticated by the publicKey of the original owner (that is, no one can forge a TX *w/o privateKey of the previous owner*)
Example: TX#N(A tries to pay coin x), how to verify that A has x? TX#N has the signature of previous owner of x, so we can authenticate the signature using the publicKey in TX#N-1 
- *Double Spending*?: Consensus on a **single** consistent TX chain for each coin. How? Bitcoin assumes *Byzantine environment*..
- PBFT? two limitations: 1) scalability - Bitcoin's huge number of particiapants makes the performance becomes 0; 2) cannot ensure 'malicious nodes < 1/3' (remember Sybil attack).
- Solution! Blockchain

### Blockchain
- Block: a set of TXs (why not just a single TX? generating a block is expensive, so batching)
- Chain: each Block contains hash(previous_block) implying order of Blocks 
- PoW Consensus: Make generating a block computatinally expensive, and accept the longest chain only. 
- For an attacker to modify a block in the Chain: recompute the block and all subsequent blocks == catch up with and surpass the work of honest nodes
== have more computational power than all honest nodes (very difficult)
- Even if attacker has more computational power than honest nodes, it makes more sense for the attacker to work on the legitimate block generation **(mining)**
to get the Bitcoin rewards from it

### Mining
- To work on generating a new block extending the last Block of the longest Chain (then, append it to the Chain and broadcast to everybody)
- Preparation: 1) get the longest Chain; 2) gather a set of ongoing TXs to include in the new Block 
- Which work? Find a *nonce* such that 'hash(Block) < threshold' using brute force (why? hashing is unpredictable)
### What is a Sybil Attack, and how has it been solved in the past? How does proof of work enable Sybil resistance for new nodes joining the Bitcoin network?
- Sybil Attack: attackers generate more identities than honest nodes 
- PoW requires computational power, not just identities, so resistant to Sybil Attack
- New nodes download the entire Blockchain history at the beginning, and start at the same phase as exisiting nodes.
### Partitioning Attack (using Network Sybil Attack)
- Isolate victim node(s) from the rest of network
- How? By occupying controls of all p2p connections of the victim
- Why? To arbitrarily disrubt the data transmission from and to the victim
- Result? Enable (or intensify) many attacks: 51%, selfish mining, censoring TXs, taking down entire cryptocurrencies, etc. 
- Why possible? Blocks are propagated through p2p network among participants, so the attacker at the middle of those connections can manipulate the message arbitrarily

What is network sybil attack and is bitcoin safe from it?
What can be done to prevent network sybil attack?


# Week-2: Segwit: Segregated Witness
### How Bitcoin TX works?
- You have one block (1MB) which contains multiple layers of data (two main layers: headers, TXs)
- Header of every TX has the sender's script (code) and the recipient's information 
- Script has the sender's signature and publicKey (necessary to verify TXs)
- However, the script is heavy in data weight so that the 1MB block is getting full and clogging down the network

### Segwit
- Scrape the script from the TX, and put it into a different block (called an extended block)
- Extended block reduces the size of a block, hence making it lighter, faster, and more scalable
- Now, TX header has only recipient's information, and the extended block is transmitted together

### Making changes on the rules of BTC network
- Consensus and governance is important in BTC community
- Sometimes, the *politics* cause major issues in applying certain BIPs
- Scaling BTC is a main topic, and changing BTC's **consensus rule** has been discussed as an approach
- Althought BTC's **decentralization** is a core value, the lack of a central decider has been an issue
- Why? No one can force new rules on the network. We need a way for the network to adapt or adjust new rulse, and make sure everyone agrees on that.

### Hard Forks and Soft Forks
- These happen whenever the rules of network are updated
- Soft: when the updates are backward compatible, and eventually only one blockchain remain valid
- Hard: when the updates are backward incompatible, and eventually both blockchain (before and after the updates) remain valid in parallel

### BIP9: Enabling easier upgrades to BTC
- Allowing updates making Soft Forks to be deployed at the same time 
- How? Change the way of nodes to interpret *version* field in blocks (using *version bits*)
- Version bits: allow miners to signal when they are ready to adopt the new rules, and allow users to set up a parallel soft fork 
- Now, several updates can be deployed in parallel without knowing which one is going to be activated first.
- Controversial. Why? Due to the 95% threshold, miners can enforce some rules that users don't want

### BIP148: 
- Adopt Segwit without consensus of miners as in BIP9

Additional Question: What should be the reasoning behind the 95% threshold of BIP 9? What would be implications of setting lower threshold?
Answer: 95% to minimize danger of chain split, but it makes a side-effect of giving too much power to miners. Reducing the threshold would relieve this side-effect, but should be set carefully since it will increase the danger of chain split 

```
List<String> words = Arrays.asList("A", "B", "C");
if (words.contains("C")) {
    System.out.println("C is in the list");
}
```
- Some data structure (e.g., Map) handles these issues using **hash tables**
- **hashcode()** calculate the hash value for a given **key** in hash tables (values can be accessed efficiently!)

## How hashCode() works?
- returns an integer value
- Equal Objects (by *equals()*) must return the same hash code
- Collisions can happen, but ensuring distinct results for unequal objects improves the performance of hash tables
- hashCode() defined by **class** Object returns distinct integers for distinct objects
- During a single execution of Java app, hashCode() invoked on the same object returns the same value (but the value can be inconsistent across executions)

## How to implement efficient hashCode()?
- Several standard options: IntelliJ, Eclipse, Lombok, etc.
- Those utilize number 31: prime and bitwise multication friendly
```
31 * i == (i << 5) - i
```

## How to handle collisions?
- Various methodologies with their own pros and cons
- ex. *HashMap* uses the *separate chaining method*: each bucket is maintained as a linked-list
- Java 8 improve it: if a bucket size over the certain threshold, replace it with tree map, i.e., O(n) to O(log n)


##### References
[1] Segwit: https://www.youtube.com/watch?v=OFfBRzh9HmU
[2] BIP9: https://bitcoinmagazine.com/technical/bip-enabling-easier-changes-and-upgrades-to-bitcoin-1453929816


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
