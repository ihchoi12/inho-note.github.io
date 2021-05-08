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
https://www.baeldung.com/java-hashcode

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
