# Ethereum Roadmap FAQ 

Non-exhaustive, if there's a question you'd like answered, please add as an issue/PR. Feel free to submit Q&As as a PR, too. 

## Merge

* What actually went on with the [Kiln merge](https://blog.ethereum.org/2022/03/14/kiln-merge-testnet/)?  
    * Like mainnet, Kiln was launched with separate PoW and PoS chains. It ran through The Merge and now operates fully under PoS. 
* Did Kiln actually follow the current spec for the mainnet merge?  By this I mean, was it equivalent to what will happen with the mainnet merge? i.e. does it follow the spec of the bellatrix fork?
    * Yes. We did make some minor changes to the spec after Kiln was live, but they were backwards compatible. The current specs for the network can be found [here](https://hackmd.io/@n0ble/kiln-spec)
    * We don't expect major spec changes now, and strongly recommend tooling/infra/application developers test on Kiln to ensure their products work as expected in a post-merge Ethereum context. 
* It seems like the Kiln testnet beacon chain has state and smart contracts. This runs contrary to what the http://ethereum.org suggests about the beacon chain.  Is this simply because I am confused by the block explorers?
    * Post-merge, Beacon blocks contain the transactional payload that current PoW blocks contain. We call this the `ExecutionPayload` in the specs. Here's a diagram which illustrates what it looks like: 
    * ![](https://i.imgur.com/ImOX35U.png)
    * On the above, the first two blocks on PoW and the Beacon Chain are pre-merge, and the last two are after The Merge. Once the last PoW block is produced, subsequent Beacon Chain blocks include the transactional data. Longer post about this [here])(https://hackmd.io/@timbeiko/acd/https%3A%2F%2Ftim.mirror.xyz%2FsR23jU02we6zXRgsF_oTUkttL83S3vyn05vJWnnp-Lc%3Fdisplay%3Diframe)
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

## Withdrawals

* How will staking withdraws actually work?  I have seen a couple specs floating around, but I don't see anything with obvious consensus on it.  If you look at the deposit contract, there is literally one write method: deposit().  There is simply no logic for withdraw
    * Withdrawals **will not** be enabled at The Merge, they will come in the fork after. Here is the current specification for them: https://notes.ethereum.org/@ralexstokes/Skp1mPSb9
        * Note: the approach for the execution layer is now confirmed to be [EIP-4895: Beacon chain push withdrawals as operations](https://eips.ethereum.org/EIPS/eip-4895)
    * Withdrawals don't "go through" the deposit contract: they are pushed by the Beacon Chain back to the EL and get credited in the same way as miner rewards currently do. This means that calclating circulating supply will be slightly more complex as the deposit contract's balance will not "decrease" with withdrawals. 
* How do withdraw keys work?  Using the eth2-deposit-cli defaults you don't automatically generate a BLS withdraw key.  How will this key work?  Will you just take the 0th index of the derivation path?
    *  EIP-2334 defines this [here](https://eips.ethereum.org/EIPS/eip-2334#eth2-specific-parameters)
*  If you specify an eth1 withdraw key, can you only withdraw to that key's address? Or is it simply used for signing a withdraw transaction, and the eth you can withdraw to any specified address?
    *  You can withdraw to the address directly
*  Suppose you use a BLS key to withdraw, where will the eth go?  My understanding is that the consensus layer will not have state or accounts.  As such, do you simply specify an execution layer address to send withdrawn eth to?  Or will the consensus layer actually have state?
    *  You specify an execution-layer address to withdraw to, and an amount to withdraw in Gwei, see [the spec](https://github.com/ethereum/consensus-specs/blob/dev/specs/capella/beacon-chain.md#withdrawal)
*  If you specify an eth1 withdraw key, can you only withdraw to that key's address? Or is it simply used for signing a withdraw transaction, and the eth you can withdraw to any specified address?
    * **TBA**


## Sharding

* Third topic: sharding.  Is there any consensus on what sharding will look like in the future?  It seems like execution sharding has been totally abandoned in favor of a rollup-centric future.
    * Execution sharding has been "deprecated" in favor of rollup-centric scaling of execution. Data sharding is now the main approach being researched and implemented. 
* How do rollups actually work post merge?  Do they still sit on top of ETH1 / the execution layer?  Or will they sit directly on top of the consensus layer?  
    * They keep working as they do today, deployed on the execution layer, but can leverage consensus layer finality.
* Where does data sharding currently sit? What actually is data sharding?  Is it just a way of solving the Data Availability problem as rollups become more popular?
    * The current plan is to first have it be exposed via a new transaction type, as specified in EIP-4844, which has a full website with an FAQ: https://www.eip4844.com/
    * This will also lay the groundwork for a full implementation of sharding on a separate p2p layer. The current proposed model for full sharding is explained [here](https://notes.ethereum.org/@dankrad/new_sharding)

