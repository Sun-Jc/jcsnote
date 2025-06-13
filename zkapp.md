# ZKP: A Black Magic, But Not A Silver Bullet (Draft) 

by JCS, Jun 10, 2025

This note is for product designers and developers who are interested in applying ZKP.
It helps to identify whether ZKP technology is a good fit for the application.
Readers should know some basic concepts of ZKP or zk-SNARKs. 
Read the following callout to see if you are ready.

> [!NOTE] Term: ZKP
> In this note, by ZKP (Zero-Knowledge Proof), we usually mean zk-SNARKs, i.e. argument with succinctness, non-interactivity and zero-knowledge properties. 

Some friends asked if ZKP can help to solve their problems such as debugging without leaking clients' data.
If the answer is no, then what applications are ZKP good for?
I realized that we have gone so far and it is time to look back at why we started the journey of ZKP.
It feels like black magic—difficult to master and capable of solving problems that no other technology can address --- trustlessness.
Optimistic mechanisms also deal with trustlessness, but they rely on economic incentives and is likely to be more fragile than cryptographic assumptions.
ZKP is not a silver bullet, as it does not solve all problems and most problems does not even need this level of trustlessness.

> [!WARN] Draft Status 
> This note is in draft status and lacks some references.

## Asymmetry

There are two abstract roles in a ZKP application: the prover and the verifier. 
For example, in a ZK-RollUp, the prover is the Layer 2 node and the verifier is a smart contract on the main-net blockchain. 

ZKP applications fundamentally stem from the imbalance between the prover and the verifier. 

### Trust

The core asymmetry is about trust.
ZKP protocols allow the verifier to outsource the computation to untrusted provers in a **trustless** way, see [verifiable computing](https://en.wikipedia.org/wiki/Verifiable_computing).

The prover is untrusted: it lacks trustworthiness from anyone, and is willing to cheat.
So, the ZKP protocol enforces the prover to behave honestly in a trustless way: no one needs to trust the prover. 

The verifier, on the other hand, is open for anyone. 
It is permissionless: anyone can be the verifier and verify the proof on its own, and thus the verifier can be trusted in this sense. 

ZKP brings in trustlessness, which is revolutionary like the Bitcoin.
But it also introduces non-neglectable cost ([several orders of magnitude of overhead](https://a16zcrypto.com/posts/article/building-jolt/)).
On the other hand, traditional applications are built on the assumption that the prover is trusted.
This can be enforced either by trusted hardware such as TEEs or by a permissioned scheme such as an real-name system where a dishonest prover will be punished.
So, if the prover is not that untrustworthy, or the application is not that cost insensitive, trusted settings are preferred.
I believe that a lot of applications is being and will be built with non-drastic trust assumptions. Please reconsider.

### Resource

ZKP applications exploit the asymmetry in resources between the prover and the verifier in two key aspects.

#### Computation

Prover has sufficient computation power.
In some applications, it owns a large amount of computation power.
Take ZK-RollUp as an example, the prover (Layer 2 node) consists of powerful datacenters, equipped with powerful GPUs, FPGAs, ASICs, etc.
In some applications, the prover is weaker but still bears fair amount of computation.
Take ZK-ID as an example, the prover resides in an end-user's device, which is just a consumer-grade processor.

The verifier's computation power is not assumed to be strong.
In some applications, it is very restricted, such as the smart contract on the main-net blockchain (ZK-RollUp).
In some applications, it is less restricted such as the web server in a ZK-ID application.


#### Information

The prover has full information.
Examples: 
In ZK-ID, the prover owns the full ID document.
In ZK-RollUp, the prover owns the full execution trace of all transactions.

The verifier owns information no more than allowed.
The verifier of ZK-ID only knows the fact that the user is more than 18 years old, whereas the name and address are not revealed.

In zk-RollUp, the zk property is actually optional.
It does not harm if the verifier (smart contract) also learns some information of the transactions.
So, many people think validity rollup is a better name.


## Discussion

It is not easier to find a good use case for ZKP than to invent a new ZKP protocol.

### Perfect Scenarios

The above explains why the public blockchain is a perfect scenario for ZKP, as the verifier, which is a smart contract, runs in an "edge-device" such as Ethereum main-net, and the prover is a untrusted party.
Examples: ZK-Rollups, ZK-Bridge, zk-CoProcessors, etc.

Confidential transaction is also a perfect use case, which exploits the imbalance **both** in computation power and information.
The prover is strongly untrusted, which matches the goal of verifiable computing paradigm.

How will ZKP go beyond Web3?
Even in the Web3 realm, Ethereum is said to pivot to Layer 1 centric scaling.
What is the future for ZKP and VB's ideal?

### Computation: Relaxation on Resource Restriction

In the past, the focus of ZKP protocol design was on [doubly-efficiency](https://eprint.iacr.org/2017/1132.pdf), which renders a linear prover and a sub-linear verifier.
It is known as the "Succinctness" property of zk-**S**NArK protocols.
The evolution of ZKP protocols appears to be closely tailored to the needs of blockchain systems.

Web2 applications, such as ZK-ID, exploit more privacy-preserving features.
They relax the constraint on the verifier's computation power.
The verifier can be much more powerful than the prover.
Client-side ZKP imposes a stronger constraint on the prover, which could even be an embedded device.
This calls for a faster prover and slower verifier.
There could be a parameterized elastic scheme to balance the workload between the prover and the verifier.
More drastically, the prover just preprocesses the witness data to be semi-finished products which is blinded, and then the verifier will do the heavier computation: this is closer to (FHE) Fully Homomorphic Encryption.

### Privacy: Compliance and Moral Hazard

ToC privacy preserving applications including confidential transaction and ZK-ID enable anonymous services such as social networks/media and payment.
This, however, is highly controversial.
They protect the privacy, but also cause harm to people and violate the law in many cases.
They also contradict the trend of massive surveillance and censorship and are not mainstream applications.
For more insights, please refer to [Cypherpunk](https://en.wikipedia.org/wiki/Cypherpunk).

How to prevent illegal use of ZKP and how to find compliant applications?

If the activities after verification is very limited, and the verification requires too much information, then ZK-ID is a good fit.
For example, just buying one alcohol offline is not worth the effort to show a full ID document.
But for critical KYC, such as opening an account, conservative service providers will be cautious.

People need security, but do they really need privacy?
We have already yielded a lot of privacy but rarely complain and quit any service.

> [!TBA]
> Users do not need it; authorities do not allow it?


### Trust: Necessity?

The zk-SaaS / zk-cloud-computing / zk-DB use case also leverages the asymmetry in computation power, but the necessity on (prover) trustlessness of them is much less undebatable as in blockchain systems.

The use of ZKP to fight against disinformation is a good example.
ZK-Real/ZK-C2PA against deep-fake is necessary, as in the era of AI, the information sources are too many and people have too little time to verify.
The trust on social media or even news reports is low enough to adopt ZKP technology.
The data size of social media is much larger than payment, and is less controllable.
(How to prevent a camera to take a picture of deep faked image, though?)

Payment used to be a low trust scenario.
Just one or two decade(s) ago, counterfeit money is not a negligible problem.
Retailers will be very alerted when they receive a bill with large denomination from a stranger.
Now, in some highly surveilled countries, they are less alerted since facial recognition cameras will find the user soon after the case.
At the same time, people have grown more comfortable with digital payments.
Years of reliable, bug-free operation have earned digital payment service providers a high level of trust.
In these regions, people believe the authority find out and punish the dishonest ones.
The authorities also publish professional technical standards to build a reliable payment system.
This again challenges the trustlessness assumption.

People are also exploring B2B applications.
ZKP is a programmable digital signature.
If the computation can be replicated and verifier by auditors, then transaction parties can just make promises by digital signature and take legal responsibility.
You can trust the other party as it will be punished later if it cheats.
But if auditors cheat and law-enforcement can never get access to the data, instant privacy preserving auditing is necessary and ZKP is good to reduce the cost.

> [!TBA]
> Verifiable AI

### Non-Interactive: Relaxation

If the verifier does not need to be open players, the protocol can be interactive.
It just allows limited number of parties to perform verifiable computing.
As a result, the protocol can be more efficient and more secure.


## ZKP Applications List

I will leave the list to another note.
 


