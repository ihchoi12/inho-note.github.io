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


# Week-3: Mining & Network Block Propagation
### How do pools give work assignments to client miners?
- The pool gives a different template for a block to each miner
- Then, each miner can generate lots of blocks from the template
- That is, although the miners try the hashing with same nonce values, they end up trying different blocks due to the differences in other parts of block
(as long as the pool never give out the same block template twice)
- Which header fileds are changed to differentiate block template? Extra nonce in the Merkle root [4]
### Do all miners search the same block candidates?
- No, since the given block template is different across miners, they search different block even if they are trying the same nonce value
### Additional Questions
- When the pool gives out different block templates, which parts of the template are different across miners?
- If the pool can assign a specific range of nonce across miners, can it be more efficient to have miners work on the same block candidate with different range of nonces?


# Week-4: P2P Network
### Tried Table
- Store public IP addrs of peers that the node has successfully established an incoming or outgoing connection
- The timestamp for the most recent successful connection to each peer is stored together.
- The position in the table is decided by hash of the peer's 1) IP addr; 2) group (/16 IPv4 prefix containing it's IP addr);
- When the location for a new peer is already occupied, the tried table priorities the peer observed more recently

### New Table
- 256 bucket, 64 IPs in each bucket
- Store public IP addrs of peers that the node has not yet initiated a successfuly connection
- How to learn those IP addrs? From DNS seeders or ADDR msgs
- The inserted IP addr belongs to 1) group (/16 IPv4 prefix containing it's IP addr); and 2) source group (group of the peer sending the inserted IP)
- If a bucket is full, isTerrible func is called => evict 1) > 30 days old; 2) had too many failed connection attempts;

### How a node choose an IP to make an outgoing connection?
1) Choose a table between New and Tried
2) Select an IP from the chosen table (baised toward 'fresher' timestamps)
3) Attempt an outgoing connection to that IP

### Discussion Question
Q. What is the rationale behind the "new"/"tried" table design? Were there any prior inspirations within the field of distributed computing?
A1) To make the list of peers of a bitcoin node various in terms of their IP prefix group. Since normally an IP prefix group is owned by a single party, distributing these of the peers prevents a single malicious party from dominating all the connections of some nodes.
A2) To balance the p2p connections of a bitcoin node between new nodes and the other nodes it has been connected to before.

Additional 
Q1: Since Bitcoin v0.10.1, the Bitcoin banned other nodes from directly insert their IP addresses to the Tried table of a node. What would be the main issue this patch tried to solve? 
Q2: How does a fixed set of 4 outbound peers get chosen? In what circumstances would you evict or change them?


# Week-5: Script & Wallets
### Bitcoin Wallets [5]
- A program to send, receive, store, and monitor BTC balances (just like Gmail or Outlook managing emails)
- Wallets interface with the BTC blockchain to monitor BTC addresses on the blockchain and update their own balance with each TX
- What is BTC address? an address used to send or receive BTC (acts like an email address), and its generated from private_key of its wallet
- A wallet is defined by its private_key
- For privacy, each private_key should be used only once, so a new private_key needs to be generated for each TX (disadvantage)
- To sum up, the wallet's core functions: creation, storage, and use of private_key (i.e., it automates Bitcoin's complex cryptography for users)

### HD Wallet (Hierarchical Deterministic Wallet) [6]
- An evolved version of wallet which generates all of its keys and addresses from a single source
- **Hierarchical** means the keys and addresses can be organized into a tree
- **Deterministic** means the keys and addresses are always generated in the same way
- Most importantly, the user can generate new public_keys without knowing the private_key
- It generates an initial phrase (known as a seed), which is a string of common words, instead of the long confusing private_key
- If a wallet gets destroyed or stolen, the user can enter the seed to reconstruct the private_key
- Also, an HD wallet can generate many BTC addresses from the same seed, and all TXs sent to addresses created by the same seed become a part of the same wallet
- Since those private_key and seed have complete power over the user's BTCs, they must be kept secret and safe

### How It Works?
- A standard BTC wallet will create a *wallet.dat* file
- This file should be backed up by copying it to a safe location
- On the other hand, HD wallet will supply the user with a seed (up to 24 words) which should be written down in a safe place too


### Discussion Question
Q. Why is the internal chain not visible outside of the wallet if it uses public derivation?
A. In order to discuss this question, we need to understand the HD wallet first. 
HD wallet is an evolved version of BTC wallet. The BTC wallet is a program to manage the user's BTC balances through the asymetric key operations. 
The user can generate his private_key to verify his ownership of the wallet or manage the BTC balances in his wallet. 
Also the user can generate public_key (using the private_key) so that other users can generate some TXs involving him.
Here, the private_key and public_ket pair should be generated for each TX, because otherwise the consistent key information can be used to compromise the user's
information. 
However, managing the key pairs generated every time the user receives a new TX is burdensome. 
So, HD wallet enables generating a large number of keys in a hierarchical way, meaning that a newly generated key is used again to generate another new key.
Public derivation is the way of deriving the new keys (child keys) from the parent public_keys without access to the corresponding private_key.
Then, what is external and internal chain? The separation of them comes from BIP32.
External chain is used for addresses that are meant to be visible outside of the wallet (e.g. for receiving payments). Internal chain is used for addresses which are not meant to be visible outside of the wallet and is used for return transaction change.
So, using public derivation means that it is another user outside of the wallet who is trying to send some BTC to the wallet, so internal chain should not be visible to it (otherwise, the external user can see some inforamtion in the internal chain like BTC spending of the waller owner)








##### References
[1] Segwit: https://www.youtube.com/watch?v=OFfBRzh9HmU
[2] BIP9: https://bitcoinmagazine.com/technical/bip-enabling-easier-changes-and-upgrades-to-bitcoin-1453929816
[3] https://bitcoin.stackexchange.com/questions/66396/how-miners-on-the-same-pool-search-non-overlapping-sets-of-nonce-candidates-alre
[4] https://blog.bitmex.com/an-overview-of-the-covert-asicboost-allegation-2/
[5] https://www.youtube.com/watch?v=A1Pl5hYHXiI
[6] https://learnmeabitcoin.com/technical/hd-wallets

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
