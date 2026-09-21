# **How Can Bitcoin Payments Happen Off-Chain**

Payments happen on the blockchain every second of the day, some days transaction volume spikes. For a transaction to receive its first confirmation it has to be recorded in a block on the blockchain. Each new block already has limited space and blocks arrive roughly every ten minutes on average.

If you look at it, we have limited block space, when demand exceeds available block space some transactions must wait a longer for confirmation which largely impacts the speed of transaction confirmation on the network. Due to the limited block space, miners generally prioritise transactions offering higher fees rates, which can make timely confirmations more expensive during busy periods.

Let’s look at this example, Alice wants to buy a bottle of water from Bob’s shop, this is a very small transaction. Alice is thirsty and she sends Bob 1,000sats. Bob would not give her the water until he received confirmation of her payment. Let's say after an average of ten minutes Alice's transaction gets confirmed and included in a block, on a normal day and the transaction fee is reasonable. Then what happens on a busy day when there’s more competition from other higher paying transactions. Alice may have to wait much longer in order to get her water from Bob. Or she has an option to pay a higher transaction fee which might end up being more than the cost of the water she is buying. This is a very inconvenient situation for Alice and even on a normal day she has to wait an average of ten minutes to get her water which is even more inconvenient.

At this point, you might wonder how practical bitcoin is for everyday purchases. Do you know that Alice can make this transaction cheaper, faster and without recording the payment separately on the blockchain? Welcome to the Lightning Network.

### **How Does Lightning Network Enables Off-Chain Payments**

The Lightning Network is a payment network built on top of Bitcoin. It allows people to send bitcoin through connected channels, usually quickly and with low fees without putting every payment on the blockchain.

Now that you know what a Lightning Network is, let’s talk about how it makes off-chain payments possible. The term off-chain basically means off the blockchain while on-chain means on the blockchain.

**What is a Payment channel?**

A standard Lightning payment channel is an arrangement backed by an on-chain funding output and signed commitment transactions. A simple payment channel appears only twice on the blockchain, when it was opened and when closing it, although other closures may require additional on-chain transactions.  The funding transaction represents the channel until it's spent. To open a payment channel, an initial transaction must first occur on the main blockchain, securing the funds within a 2-of-2 multisig address where both participants are required to sign off on any spending. Once open, Alice and Bob can send funds to each other by privately exchanging signed balance updates without broadcasting anything to the blockchain.&nbsp;

**How do you create a channel?**

Let’s discuss how to create a payment channel. Using Alice and Bob as our example, Alice begins by sending an `open_channel` message containing her proposed channel terms and funding public key. If Bob agrees, he responds with an `accept_channel` message containing his own funding public key and requirements.

Alice then prepares the funding transaction, which will lock bitcoin into the channel. Before broadcasting it, Alice and Bob exchange signatures on their commitment transactions. These transactions specify how the channel’s funds can be distributed between them.

Alice sends a `funding_created` message containing the funding transaction’s details and her signature for Bob’s commitment transaction. Bob replies with a `funding_signed` message containing his signature for Alice’s commitment transaction. Each participant now has the other’s signature needed to claim their share under the channel’s rules without requiring further cooperation.

Alice can then broadcast the funding transaction to the Bitcoin network. Once it receives the required confirmations, both participants exchange `channel_ready` messages, making the channel available for off-chain payments.

**How do you update a channel?**

To update their payment channel, Alice and Bob exchange fresh commitment transactions reflecting their updated balances.

For instance, assuming both parties hold an initial balance of 5 BTC within the channel and Alice intends to transfer 1 BTC to Bob, the updated arrangement allocates 4 BTC to Alice and 6 BTC to Bob (excluding network fees). Alice provides her signature for Bob’s commitment transaction while Bob signs Alice’s commitment transaction, guaranteeing that both individuals hold valid proof of the updated balances.

After securing the updated commitment transactions, Alice and Bob revoke their previous commitments by exchanging the corresponding per-commitment secrets. The old transactions remain valid under Bitcoin’s rules, but these secrets allow either participant to penalize the other for broadcasting a revoked commitment.

This step completes the balance adjustment entirely off the main ledger. The underlying funds stay secured within the multi-signature output while their latest signed commitments track the current balances.

Notice what has changed. Excluding any minor network costs, Alice holds 4 BTC while Bob holds 6 BTC, yet the initial 10 BTC remains secured within that identical multi-signature output. Through the private exchange of signed commitment transactions, both wallets simply track an updated distribution of those balances. This transfer effectively adjusts their claim over the underlying bitcoin, all without broadcasting a transaction to the main blockchain network.

These cryptographic signatures provide immediate security. Should Alice suddenly go offline, Bob already possesses her valid signature on the most recent commitment state. He can append his signature to publish the transaction on-chain and close the channel independently. Likewise, Alice can execute the exact same procedure if Bob becomes unresponsive. However, settling channel funds on the main network in this manner incurs transaction fees and requires blockchain confirmations and may involve additional delays before funds become spendable.

Lightning uses a penalty mechanism to discourage participants from broadcasting revoked channel states. Suppose Alice broadcasts an earlier revoked commitment transaction that allocates her more bitcoin than the current state. Bob already has the per-commitment secret Alice disclosed when she revoked that commitment. He can use it, together with his own secret material, to derive the revocation private key and claim Alice’s delayed funds as a penalty. Bob, or a watchtower acting on his behalf, must detect the transaction and respond in time to claim those funds before Alice spends them.

Revocation therefore makes publishing an old commitment transaction punishable; it does not make the transaction invalid under Bitcoin’s consensus rules.

**How do channels become a network?**

Now let’s bring Carol into the picture. Suppose Alice wants to pay Carol but does not share a channel with her. Bob has a channel with each of them, creating a possible route: Alice → Bob → Carol. Alice can decide to route the payment through Bob to Carol. This is a simple example, but let’s look at a larger picture where more participants join the network and connect through multiple channels, creating more possible routes across the Lightning Network. Routing nodes may charge fees for forwarding payments.

In the case of sending money from Alice to Carol, this is simple when all of you know each other or are friends, but what if you need to send the money across the lightning network that comprises several participants whom Alice does not know or trust.&nbsp;

Each channel along the network needs to have enough liquidity in order to be able to route a payment, for example if Alice wants to send 1BTC to Carol and Bob has only 0.1BTC available to send toward Carol. The network might have to reroute the payment to a different route with enough liquidity to forward the transaction.

When a payment travels through several channels, Alice needs assurance that the intended recipient can claim it. Bob and any other intermediary along the route needs assurance that forwarding the payment allows them to claim the corresponding incoming funds. If payment fails Alice including other participants also need a way in order to recover their locked funds.&nbsp;

This is where Hashed Time-Locked Contracts (HTLCs) come in. They connect conditional payments across the routes using a secret, its hash and deadlines, allowing participants to coordinate transactions without trusting each other.

**How Do HTLCs Enable Trustless Payments**

An HTLC is what protects the payment when the intermediaries cannot be trusted. Besides enabling trustless payment, HTLC specifies how the receiver can claim the funds and how the sender can recover them if for some reason the payment fails or remains unresolved.&nbsp;

An HTLC uses a random secret called a preimage, represented by `S`. Hashing this value produces a payment hash: `H = SHA256(S)`. The hash can be shared while the preimage remains a secret.&nbsp;

For simplicity, this example ignores routing fees. We will again use the link between Alice, Bob and Carol to demonstrate how HTLC operates. Alice again wants to pay Carol through Bob. Carol  first generates a preimage `S` and gives Alice its hash, `H`, through her invoice.&nbsp;

Alice now offers Bob an HTLC of 1BTC whose hash-lock condition requires the matching preimage. Bob then offers Carol an HTLC of 1BTC using the same hash and committing funds from his channel with Carol. These two payments become connected because they both need the matching preimage that initially only Carol knows.

Carol knows the preimage, S, and once she confirms the expected amount and conditions of the incoming payment, she reveals the preimage to Bob. Bob hashes the preimage and checks whether `SHA256(S)` matches H. If it does, Bob and Carol update their channel to settle her payment.

Bob then passes the preimage to Alice. She verifies it, and they update their channel to settle Bob’s incoming payment. This entire process happens off-chain.

If you notice, HTLC moves forward while the preimage travels backward. From Carol to Bob and to Alice last, because Carol’s ability to claim her payment gives Bob the required information needed to claim his corresponding payment from Alice.

To protect each intermediary, its outgoing HTLC must expire before its incoming HTLC. This gives the intermediaries time to claim their funds after learning the preimage from the intended receiver.&nbsp;

If the payment fails and the participants cooperate they can cancel the HTLC off-chain and settle all reserves balances. And if someone stops cooperating the affected channel might need to be closed in order to allow locked funds to be recovered through the blockchain’s timeout rules.&nbsp;

Also note that claiming an HTLC on-chain requires the specified signatures as well as the matching preimage. Expiry enables timeout recovery, it does not automatically refund funds. On-chain recovery takes time and incurs fees.

### **So, How Do Payments Happen Off-Chain?**

Let’s return to our example of Alice with her bottle of water. If Alice and Bob have a working channel or route on the network, Alice can pay Bob without having to wait for that particular payment to be recorded in a new Bitcoin block. This offers usually faster payments with low fees. Once the payment is complete, Bob can use his funds for further transactions on the network while his channel remains fully open.

Behind that simple purchase are the mechanisms that we have explored,  payment channels hold the funds, signed commitment transactions track the balances, and HTLCs connect conditional payments across the network. Bitcoin’s blockchain remains the foundation for funding channels, settling them, and enforcing their rules when cooperation breaks down.

Alice’s payments happen off-chain because the participants update their enforceable claims to funds without recording each payment on-chain. while the channel balance is being secured by the blockchain. This makes Alice’s quick purchase of a bottle of water possible without requiring a separate blockchain entry which in turn uses less block space than recording every payment separately.
