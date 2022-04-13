# Ethereum Roadmap FAQ 

Non-exhaustive, if there's a question you'd like answered, please add as an issue/PR. Feel free to submit Q&As as a PR, too. 

Full disclaimer: this current version was typed in <1h, and will likely have typos & minor inaccuracies.

## Merge

* What actually went on with the [Kiln merge](https://blog.ethereum.org/2022/03/14/kiln-merge-testnet/)?  
    * Like mainnet, Kiln was launched with separate PoW and PoS chains. It ran through The Merge and now operates fully under PoS. 
* Did Kiln actually follow the current spec for the mainnet merge?  By this I mean, was it equivalent to what will happen with the mainnet merge? i.e. does it follow the spec of the bellatrix fork?
    * Yes. We did make some minor changes to the spec after Kiln was live, but they were backwards compatible. The current specs for the network can be found [here](https://hackmd.io/@n0ble/kiln-spec)
    * We don't expect major spec changes now, and strongly recommend tooling/infra/application developers test on Kiln to ensure their products work as expected in a post-merge Ethereum context. 
* It seems like the Kiln testnet beacon chain has state and smart contracts. This runs contrary to what the http://ethereum.org suggests about the beacon chain.  Is this simply because I am confused by the block explorers?
    * Post-merge, Beacon blocks contain the transactional payload that current PoW blocks contain. We call this the `ExecutionPayload` in the specs. Here's a diagram which illustrates what it looks like: 
    * ![](https://i.imgur.com/ImOX35U.png)
    * On the above, the first two blocks on PoW and the Beacon Chain are pre-merge, and the last two are after The Merge. Once the last PoW block is produced, subsequent Beacon Chain blocks include the transactional data. Longer post about this [here](https://hackmd.io/@timbeiko/acd/https%3A%2F%2Ftim.mirror.xyz%2FsR23jU02we6zXRgsF_oTUkttL83S3vyn05vJWnnp-Lc%3Fdisplay%3Diframe)
* At a high level, what does the block processing flow look like, post-merge? 
    1. A validator is elected to propose a block
    2. This validator asks its execution layer (EL), via the [Engine API](https://github.com/ethereum/execution-apis/tree/main/src/engine), to send him an `ExecutionPayload`
    3. The EL returns the payload which contains the most profitable set of valid transactions to the consensus layer (CL).
    4. The CL proposes a block which includes this payload and propagates it on the Beacon Chain p2p network
        * Notes: 
            1. Individual transactions are still gossiped on the EL p2p network and the EL is solely in charge of maintaining the transaction pool. Full blocks get propagated on the CL p2p network.
            1. The validator specifies the address, on the execution layer, at which they want to receive fees. Transaction fees never get "locked" on the Beacon Chain like the validator stake and rewards do. 
    5. Other validators attest to the block and, if valid, propagate it on the Beacon Chain p2p network
* What is the testing process like for The Merge? 
    * We are running several testing efforts in parallel. A list is available [here](https://github.com/ethereum/pm/blob/master/Merge/mainnet-readiness.md#testing) 

### Shadow Forking

* What is Shadow Forking? 
    * **TL;DR: a shadow fork is a new devnet created by forking a live network with a small number of nodes. The shadow fork keeps the same state & history, and can therefore replay transactions from the main network.**  
    * Longer version: A shadow fork is when a small number of nodes are configured to fork off from an Ethereum network at a certain point. In the context of The Merge, we do this by launching nodes which are set to run through The Merge at an earlier point than the entire network. This allows us to test how the upgrade would have happened in similar conditions to the shadow-forked network, but without the vast majority of the nodes being aware this has happened. After the shadow fork, transaction valid on the main chain can get replayed on the forked chain as well, simulating the throughput of the original network. [@parithosh_j has a good tweet thread with more details](https://twitter.com/parithosh_j/status/1513129881927884801)
    * The diagram below, from the linked thread, shows what the network looks like after a shadow fork:   ![FP-tJYhXwAI57Cm](https://user-images.githubusercontent.com/9390255/162835363-d5b70366-c9a7-460e-b881-321cae2c72e6.png)
        * The rop row of Goerli blocks shows a node on the canonical chain, which are not aware of the shadow fork. 
        * The middle row of Goeli blocks shows a node on the shadow-forked chain, which has a modified configuration telling it to fork once the total terminal difficulty (TTD) is hit.
        * The bottom row shows a Beacon Chain which was launched for the purposes of the shadow fork only: it will provide consensus to the chain when TTD is hit. 
        * After TTD is hit, the nodes on the canonical chain continue producing blocks normally: nothing has happened "for them". 
        * After TTD is hit, nodes with the modified configuration fork off and run through The Merge. The first post-merge block is produced by the next validator in the Beacon Chain. While this block can contain any transaction seen on the canonical chain, the exact transactions included, or their ordering, is not necessarily the same as on the canonical chain. 
* Why are Shadow Forks useful?
    * Shadow Forks allow us to see how nodes react when The Merge happens using only a small number of nodes and without disrupting the canonical chain. Shadow Forks give us a more realistic environment to test in than launching new testnets, because existing testnets already have transactions happening organically on them, and a large state size & block history which put node under more stress than new testnets. They therefore allow us to get "real world" performance metrics on nodes, without potentially affecting the canonical network's operations. 
* Can you Shadow Fork mainnet? 
    * Yes, and we now have! Shadow Forking mainnet is incredibly useful because it shows us how nodes react in the harshest possible conditions: when their state and history is large and transactions are the most complex. After a mainnet shadow fork, we can also test how stable nodes are, how well they sync, etc. when trying to join the forked network. This gives us data not only on the transition itself, but also on how new nodes behave when they join the network in a post-merge state. 
* What stage in The Process™️ are Shadow Forks?    
    * Shadow forks help us increase our confidence that implementations work as expected. Once they go smoothly across all implementations, we can then confidently run existing testnets through The Merge. It is worth noting that the nodes in Shadow Forks are controlled by a small set of operators: some public testnets have much broader validator sets. Once testnets are upgraded and stable, then we can plan for The Merge to happen on mainnet.  

### Merge Timelines 

* When will The Merge happen? 
    * There is no official date for The Merge yet. A date will only be set once client teams are confident that the software implementations have been thoroughly tested and are bug-free. Any date announcement will be communicated on blog.ethereum.org 
* What needs to happen before The Merge?
    * As of April 2022, all client teams have in-progress implementations for The Merge, which have been tested by test suites, the launch of new testnets and shadow forks. 
    * Shadow forks, run against both existing testnets and the Ethereum mainnet, have revealed implementation issues in clients. Teams are now fixing these and regularly re-running shadow forks to test fixes. 
    * Once clients work without issues during shadow forks, then the existing Ethereum testnets (Ropsten, Goerli, etc.) will be run through The Merge. An announcement will be made on blog.ethereum.org at this point.
    * Once testnets have successfully upgraded, and remain stable, **then** a time will be set for the upgrade to happen on the Ethereum mainnet. 
        * The Merge, unlike previous Ethereum upgrades, will not be triggered by a block time. Instead, it will be triggered by a total difficulty value. Given these are harder to estimate than block times, the delay between choosing a time for The Merge and it going live on the network may be slightly shorter than prior Ethereum upgrades.
    * A list of tasks to be completed before the mainnet upgrade can be found [here](https://github.com/ethereum/pm/blob/master/Merge/mainnet-readiness.md). 
* How does the difficulty bomb affect the timeline?
    * The difficulty bomb is expected to start being noticeable on the Ethereum network around May, to start noticeably contributing to block times in June/July, and to make blocks unbearably (read 15-20 seconds) slow by August. Its progress is being tracked [here](https://ethresear.ch/t/blocks-per-week-as-an-indicator-of-the-difficulty-bomb/12120). 
    * If client developers do not think they can deploy The Merge to mainnet before block times are slowed too much, it will need to be delayed again. Two options are possible to delay the bomb:
        * **Combine Merge & Bomb Delay:** if we only need to delay the bomb a few weeks, we can combine a bomb delay with client releases for The Merge. The way this would work is that these releases would delay the bomb at a certain block, restoring 13s block times, and then activate The Merge shorly after. Because The Merge needs to happen fairly close to client releases, this scenario is only helpful if we want to delay the bomb by a few weeks.
        * **Separate Bomb delay:** if we do not expect to activate The Merge on the Ethereum mainnet before block times become unbearably slow, then a separate network upgrade, which only delays the difficulty bomb, will need to happen prior to The Merge. Note that the time by which we delay the difficulty bomb is independent of when The Merge happens. For example, if we delayed the bomb by 6 months, we could merge before that. Similarly, if we delayed the bomb 3 months and found a major issue right before it goes off again, another bomb delay would be considered. 
* How can I help The Merge happen quicker?
    * If you run a node, either as an infrastructure provider, validator, or hobbyist, make sure to test your current setup on [Kiln](https://blog.ethereum.org/2022/03/14/kiln-merge-testnet/) to ensure it works as expected. Trying the software in many environments, with many set of eyes is the best way to ensure that we catch bugs early in the process.



## Withdrawals

* How will staking withdraws actually work?  I have seen a couple specs floating around, but I don't see anything with obvious consensus on it.  If you look at the deposit contract, there is literally one write method: deposit().  There is simply no logic for withdraw
    * Withdrawals **will not** be enabled at The Merge, they will come in the fork after. Here is the current specification for them: https://notes.ethereum.org/@ralexstokes/Skp1mPSb9
        * Note: the approach for the execution layer is now confirmed to be [EIP-4895: Beacon chain push withdrawals as operations](https://eips.ethereum.org/EIPS/eip-4895)
    * Withdrawals don't "go through" the deposit contract: they are pushed by the Beacon Chain back to the EL and get credited in the same way as miner rewards currently do. This means that calclating circulating supply will be slightly more complex as the deposit contract's balance will not "decrease" with withdrawals. 
* How do withdraw keys work?  Using the eth2-deposit-cli defaults you don't automatically generate a BLS withdraw key.  How will this key work?  Will you just take the 0th index of the derivation path?
    *  EIP-2334 defines this [here](https://eips.ethereum.org/EIPS/eip-2334#eth2-specific-parameters)
*  If you specify an eth1 withdraw key, can you only withdraw to that key's address? Or is it simply used for signing a withdraw transaction, and the eth you can withdraw to any specified address?
    *  To withdraw ETH from the beacon chain, you **MUST** specify an eth1 address as the "target" recipient. you will not need to sign anything from this account as the withdraw will happen automatically from the execution layer perspective. When the withdrawal happens, it will look like the target account suddenly has the extra ETH in the account balance in the post-state of the block containing the withdraw.
*  Suppose you use a BLS key to withdraw, where will the eth go?  My understanding is that the consensus layer will not have state or accounts.  As such, do you simply specify an execution layer address to send withdrawn eth to?  Or will the consensus layer actually have state?
    *  You specify an execution-layer address to withdraw to, and an amount to withdraw in Gwei, see [the spec](https://github.com/ethereum/consensus-specs/blob/dev/specs/capella/beacon-chain.md#withdrawal). To be clear, you *cannot* withdraw with a BLS withdrawal key. We will have an operation to change your credentials on the consensus layer coming in the fork after the Merge.


## Sharding

* Is there any consensus on what sharding will look like in the future?  It seems like execution sharding has been totally abandoned in favor of a rollup-centric future.
    * Execution sharding has been "deprecated" in favor of rollup-centric scaling of execution. Data sharding is now the main approach being researched and implemented. 
* How do rollups actually work post merge?  Do they still sit on top of ETH1 / the execution layer?  Or will they sit directly on top of the consensus layer?  
    * They keep working as they do today, deployed on the execution layer, but can leverage consensus layer finality.
* Where does data sharding currently sit? What actually is data sharding?  Is it just a way of solving the Data Availability problem as rollups become more popular?
    * The current plan is to first have it be exposed via a new transaction type, as specified in EIP-4844, which has a full website with an FAQ: https://www.eip4844.com/
    * This will also lay the groundwork for a full implementation of sharding on a separate p2p layer. The current proposed model for full sharding is explained [here](https://notes.ethereum.org/@dankrad/new_sharding)

