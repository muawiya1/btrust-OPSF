# **My First Week Journey In The Btrust Open Source Fellowship Program**

The first two weeks came with some amazing experiences, which included the nerve-calming presentation from the Btrust team during the onboarding kickoff session. After that, the week’s materials were right on point to open the week and to introduce us to Bitcoin technology and how it operates. The program structure makes it exciting to learn alongside other fellows in the program, in which your progress is monitored to ensure progressive learning throughout the duration of the program and beyond.

### **The Kick-Off Session**

To write my experience for the first week, the onboarding session will be the bedrock of my first week. It helped me in a way to fully understand the structure of the program and gave me a sort of highlight on what my next 12 weeks will be. Starting  with week one, the session has given insight into how to highlight my working hours to separate my day job and the fellowship efficiently without overlapping one with the other.

### **The Learning Experience and the Modules**

**Week 1: Foundations & SegWit**  
Starting off with the famous Bitcoin Whitepaper published by Satoshi Nakamoto, in the paper there is detailed information on what Bitcoin is and how it operates. It was such a beautiful piece put together to introduce Bitcoin to the world, and for me it provided the good start I needed to refresh my mind on what Bitcoin is all about. The perfect explanation is a leap for me in understanding the technology further because I have never had the chance to fully read the whitepaper.

Further reading into the mandatory material, I got to learn about some more technical inventions before Bitcoin that the whitepaper mentioned, but I got to know the history of some of the inventions that were meant to be the internet currency but failed, and the reasons for their failure are well documented. Meanwhile, their failure was such a good start for Satoshi to put all those pieces together and invent the perfect internet currency where there’s no central authority, which was the most important factor in the failure of all the inventions before Bitcoin. 

Even though bitcoin achieves decentralization, one of the thing that made it different is the proof-of-work, which makes bitcoin to be tough and hard to break network providing the security assurance without central authority, the proof-of-work involves a consensus rule that was embedded in the heart of bitcoin where computers compete to solve a computational puzzle to to validate transactions and add new block to the network, the puzzle is the hardest part and after a computer solved the puzzles, it broadcast the new blockhash to the network and all other peers can easily validate that it’s correct, to solve this puzzle it an enormous computing power is needed which in turn requires investing more money in hardware and electricity. The proof-of-work can be said to be a Sybil-attack-resistant mechanism, which gives miners the power to outbid an attacker in the system.

Segregated Witness (Segwit), even though this was later introduced in Bitcoin, I can say that it is the one true update that really opened a new world in the Bitcoin network. Firstly, it fixes transaction ID (txid) malleability which is a situation where by a transaction ID can be slightly changed without invalidating the transaction, before Segwit, the transaction signature’s data was part of the data used to calculate transaction ID, which means that once a transaction is broadcast to the network a user can still alter transaction signature which in turn changes the transaction ID before it’s confirmed in the network. Segwit solves this problem by moving the transaction signature data outside the transaction data that is used to calculate the transaction ID, so even if you change the transaction signature data after the transaction was broadcast, it will not have any effect on the transaction ID; hence, the transaction ID stays the same no matter what.

Segwit introduces a new method of counting block size instead of raw bytes;, weight units are used instead, and since the witness data is discounted in the transaction data this helps Segwit transactions use block space more efficiently, thereby reducing transaction fees compared to a legacy transaction. 

Furthermore, Segwit provides backward compatibility between old nodes and new nodes in the Bitcoin network without requiring a hard fork. Legacy nodes that do not understand Segwit can still receive and validate the non-witness portion of the transaction as if it were a valid transaction, while upgraded Segwit nodes understand and validate both the transaction data and the witness data; this compatibility is one of the key reasons why Segwit was implemented as a soft fork rather than a hard fork.

Segwit made second-layer solutions like the Lightning Network possible in practice; by fixing the malleability bug, it helped set the groundwork for further innovations and scaling in the Bitcoin network while also making transactions cheaper.

Unarguably, this week's materials further expanded my knowledge into some of the security architectures of the Bitcoin Network, like the checkpoint, assumevalid, assumeutxo, bloom filters, and compact block filter. Before this week, I never knew about these mechanisms set in place to provide the network with some efficiency and solve one of the biggest bottlenecks of full verification in the network, thereby helping to ease the cost of participation in the Bitcoin ecosystem; they make the use of Bitcoin on lightweight devices possible.  

#### **Week 2: Mining, P2P, Script & Wallets**

In this module, I learned more about Bitcoin. To start with, I learned that a Bitcoin block doesn’t appear out of thin air; it is mined. Miners create new blocks through a process called mining. What is mining, and how long does it take to mine a block? I know these questions might be on your mind right now, so let me explain.

Mining is the process of finding the next block on the blockchain. It involves solving a computational puzzle, or performing proof of work, to find a valid block hash. Solving this puzzle is difficult, but verifying the solution is easy. Once a miner finds a valid block hash, they broadcast the newly discovered block across the network. The nodes can then verify that the miner has performed the required proof of work and update their blockchain history.

How soon should we expect the next block to be mined? On average, I would say 10 minutes. However, this does not mean that a new block appears exactly every 10 minutes. If nine minutes have passed since the last block appeared, the expected waiting time for the next block is still another 10 minutes. Here, I learned that this process follows a Poisson distribution and that the waiting time is memoryless: at any point, the next block is expected to be mined approximately 10 minutes into the future.

After a block is mined, the next step is to send it across the network so that all connected nodes can discover it and update their blockchain history. This process is called block propagation. It sounds easy, but considering how large the network is, propagation needs to be fast enough to reduce the chance of a temporary chain split.

If a block does not propagate quickly enough, another miner might still be working on the previous block because they have not yet heard about the newly mined one. They might then find a competing block. Since both blocks are valid, the network temporarily has two competing branches. Eventually, one branch becomes the winning chain, while the block on the losing branch is left behind.

In this section, I also learned about compact blocks. Compact blocks improve propagation speed by reducing the amount of data sent across the network. There are two relay modes: low-bandwidth relay and high-bandwidth relay.

Since nodes in the network have nearly identical mempool contents, it is possible to reduce the bandwidth required to propagate a block. The sending peer sends a compact block “sketch,” and the receiving peer uses it to reconstruct the entire block from transactions already in its mempool. If any transactions are missing, the receiving peer asks the sending peer to send them.

In high-bandwidth relay mode, the transmitting peer relays a compact block to its peers as soon as it is found or validated, without waiting to be asked. This uses more bandwidth but improves block propagation across the network.

In low-bandwidth relay mode, the transmitting peer first sends a short announcement to its peers saying, “I have a new block.” If a receiving peer is interested, it requests the block in response to the announcement. This reduces redundancy because some peers may have already received the block.

Whichever relay method is used, the compact block sends short transaction IDs instead of the full transaction data. Since the receiving peer has already seen most of the transactions in its mempool, it can reconstruct the block, saving bandwidth compared with sending all the transaction data within a block over the network.

The Bitcoin network is where everything happens. In this module, I learned about the network and how it operates. It is a peer-to-peer (P2P) network comprising nodes that run Bitcoin software, communicate with one another, and collectively maintain, validate, and enforce the rules of the Bitcoin blockchain. These nodes can act as clients or servers. The network is decentralized, meaning that nodes communicate independently without a central authority coordinating them.

There are different types of nodes, including fully validating nodes (full nodes), archive nodes, mining nodes, and simplified payment verification (SPV) nodes. Let’s briefly go over my understanding of each.

A full node validates transactions and blocks. It downloads every block and transaction and independently checks them against all of Bitcoin’s consensus rules. In doing so, it enforces Bitcoin’s rules across the network. However, it does not always keep all historical block data forever. A full node can run in pruned mode, meaning that it downloads and validates blocks, then discards older block data to conserve space while continuing to validate incoming transactions and blocks.

An archive node keeps a record of the entire block history, from the genesis block to the present, and cannot run in pruned mode.

A mining node is a full node with mining capabilities. It validates transactions like any other full node, but it can also assemble new blocks from pending transactions in its mempool and compete to solve the proof-of-work puzzle to mine a new block and update the blockchain history.

An SPV node, also called a lightweight node, does not have the network’s full block history. Instead, it depends on full nodes to validate the rules that it cannot check itself. It may have enough data to verify its own transactions, but it does not fully validate blocks.

New nodes first connect to peers that provide information they do not yet have, such as the current state of the network. A node usually connects to multiple peers to update its block history, requesting information from different peers rather than relying on just one.

Remember, this is a trustless network. If a node connects to a dishonest peer, that peer might keep feeding it incorrect information. This is why it is important to connect to multiple peers to help avoid an eclipse attack, in which a node is surrounded by malicious peers that prevent it from receiving accurate information about the network’s state.

In this section, I learned about Script. Script is a small programming language used as a locking mechanism for outputs in Bitcoin transactions. The scriptPubKey is the locking script for an output, while the scriptSig or witness provides the unlocking information needed to spend it.

The Script language has two major components: opcodes and data. Opcodes perform operations on the provided data to define or satisfy spending conditions. To validate a transaction, the receiving node executes the relevant scripts. If OP\_1—and nothing else—remains on the stack after execution, the transaction is valid; otherwise, it is invalid.

I also learned about Miniscript, which makes scripting easier for complex spending conditions. Bitcoin nodes do not execute Miniscript directly. Instead, software converts it into Bitcoin scripts, which the nodes then execute.

A hierarchical deterministic (HD) wallet generates all its keys and addresses from a single source, called a seed. It can generate billions of private keys from one seed. As long as you retain the seed, you can recover the corresponding keys and addresses. The keys and addresses are generated deterministically and can be organized into a tree, hence the name “hierarchical deterministic.”

This section also helped me understand Taproot, which allows people to create complex spending conditions while revealing as little information as possible on the blockchain. It is an upgrade to the Bitcoin network that became active in November 2021 and introduced a new output type called Pay-to-Taproot.

For example, you can identify a multisig transaction on the blockchain by its revealed spending conditions. With Taproot, only the spending condition used to spend the output is revealed, while the remaining conditions stay hidden.

I learned about Schnorr signatures, especially their security proof, non-malleability, and linearity. The linearity of Schnorr signatures particularly caught my attention. It allows multiple signers in a multisig transaction to combine their public keys into one aggregated key representing the group, a property called key aggregation. This aggregated key is included in the output, making it harder for blockchain observers to distinguish it from an ordinary transaction on the network.

### **Discussions Questions**

These questions are the most interesting part after reading through the modules, it tests your understanding of what you have read and help you gain a deeper understanding, and if you don’t, most times it prompt me to go back and read the material again, depending on the section that I cannot answer clearly or had no clear understanding of the section.

### **The Discussion with a Paired Fellow**

Another part of the journey is for me to get to interact with other fellows outside my homegroup, it is a good experience as it gives me an avenue to learn differently, get to share my perspective and learn from my paired fellow perspective, actually there were things that might have skipped my mind on some questions but when my paired fellow tries to explain from his views it literally sometimes gives me more idea about that answer or topic. 

We had a very good talk with my paired fellows, where we shared ideas and learned from each other’s view of the solution. I had two different partners each week, it was really a great experience getting to learn from other fellows and share from their experience.

### **Homegroup Session**

For me, that’s refreshing, because you get to hear from your homegroup, get to know the progress within the group, and ultimately, if there’s anyone with a blocker it will be discussed and solutions will be offered or guidelines to follow, and we get to share more amazing experiences through the learning process.

The faculty members are amazing; they encouraged us to ask questions and outline any problem that we might have so that they might guide us to achieve the best out of this fellowship.

### **Feelings Friday**

As the name implies, it’s indeed a feelings Friday. This is where everyone from different home groups and the entire faculty and Btrust staff gather to share their own overall feelings of the week. It’s an open space for everyone to share their exciting moments of the week, the fun activities, and many other beautiful experiences. 

It was indeed a moment to keep looking forward to, always.
