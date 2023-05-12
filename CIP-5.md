---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 5 | Optimization for PoDR2 validation phase | EldenYang | Draft | Standards | Core | 23/05/09 |

## Abstract
In order to solve the problem of too many transactions generated on the chain during challenges, this chapter proposes to optimize the PoDR2 algorithm to reduce the pressure on the chain during random challenges and ensure the reliability and availability of service data and idle data. Based on a trusted computing environment (TEE Worker), the proofs submitted by storage nodes are verified to ensure fairness and impartiality of the verifier. At the same time, in order to reduce the number of transactions, the data structure related to generating challenge information is optimized, and changes have also been made to the challenge scope and challenge trigger frequency. Most importantly, aggregate proof technology is adopted so that storage nodes can submit all proofs about local data at one time, greatly reducing the number of interactions between storage nodes and the chain.
## Motivation

1. The duration of random challenges is too long, resulting in excessive resource consumption of the entire network for a period of time.
2. As the amount of data increases, the pressure on the chain gradually increases and there is a risk of excessive accumulation of transactions in the short term.
3. The untrustworthiness of the verifier’s workflow poses certain security risks.
4. When a storage node itself stores too much data, the proportion of resources consumed by calculating proofs will be too large, causing other main business processes to lag behind.
## Specification
### Overall Process

1. First, the chain network determines whether to trigger a random challenge based on a certain probability. When a random challenge is triggered, several current verifier nodes will execute the generation work through the off-chain worker. The execution result recognized by most verifier nodes will be used as the challenge information to generate.
2. When the storage node listens to the generation of the challenge, it will check whether it is the target of this round of challenge. If it is, it will start calculating the proof. Finally, submit the idle aggregate proof and service aggregate proof to the chain. Because the amount of aggregate proof data is too large, it will not be stored on the chain in its entirety, only some core information of the aggregate proof will be stored for comparison by the verifier.
3. After receiving the proof from the storage node, the chain network will randomly assign a consensus node to be responsible for verification. After querying the allocation result, the storage node will send the proof to the designated consensus node for verification. After completing the verification task through TEE Worker, the consensus node submits a signature containing a trusted environment and verification results to the chain.
4. The chain network rewards or punishes storage nodes based on verification results.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22121693/1683635294423-c0d46f2a-242c-44ab-8f71-5fc9c989c01b.png#averageHue=%23fafaf9&clientId=uaaa443fb-feac-4&from=paste&height=684&id=ub61cf0ae&originHeight=752&originWidth=819&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=48734&status=done&style=none&taskId=u0722c848-a194-4b00-9d5a-bf0ff6b4e81&title=&width=744.5454384078668)

### Blockchain Node Interface
#### Data Structure

1. **Challenge snapshot information storage**

It is used to store the challenge information of this round and make it public to the whole network. At the same time, it also contains some necessary parameters for calculating proofs.
```rust
pub type ChallengeSnapShot<T: Config> = StorageValue<_, ChallengeInfo<T>>;
```
No primary key.<br />Value：ChallengeInfo <br />ChallengeInfo Struct
```rust
pub struct ChallengeInfo<T: pallet::Config> {
	pub(super) net_snap_shot: NetSnapShot<BlockNumberOf<T>>,
	pub(super) miner_snapshot_list: BoundedVec<MinerSnapShot<AccountOf<T>>, T::ChallengeMinerMax>,
}
```
`net_snap_shot`: Network snapshot.<br />`miner_snapshot_list`: Storage node snapshot.<br />Network snapshot information structure.
```rust
pub struct NetSnapShot<Block> {
	pub(super) start: Block,
	pub(super) life: Block,
	pub(super) total_reward: u128,
	pub(super) total_idle_space: u128,
	pub(super) total_service_space: u128,
	pub(super) random: [u8; 20],
}

```
`start`: Challenge generation time.<br />`life`: Challenge declaration cycle.<br />`total_reward`: Total reward for this round’s challenge prize pool.<br />`total_idle_space`: Total idle space for this round of challenge.<br />`total_service_space`: Total service space for this round of challenge.<br />`random`: Public random number for this round of challenge.<br />Storage node snapshot information structure.
```rust
pub struct MinerSnapShot<AccountId> {
	pub(super) miner: AccountId,
	pub(super) idle_space: u128,
	pub(super) service_space: u128,
}

```
`miner`: Storage node account address.<br />`idle_space`: Storage node idle space.<br />`service_space`: Storage node service space.

2. **Storage of proofs to be verified**

It stores the verification tasks that each consensus node is responsible for, as well as the proof information submitted by the storage node.
```rust
pub(super) type UnverifyProof<T: Config> = 
    StorageMap<_, Blake2_128Concat, AccountOf<T>, BoundedVec<ProveInfo<T>, T::VerifyMissionMax>, ValueQuery>;
```
Primary key: AccountOf consensus node wallet address.<br />Value: BoundedVec<ProveInfo<T>, T::VerifyMissionMax> list of proof information.  
Proof of Information Structure
```rust
pub struct ProveInfo<T: pallet::Config> {
	pub(super) snap_shot: MinerSnapShot<AccountOf<T>>,
	pub(super) idle_prove: BoundedVec<u8, T::SigmaMax>,
	pub(super) service_prove: BoundedVec<u8, T::SigmaMax>,
}
```
<br />`snap_shot`: Store node snapshot information.<br />`idle_prove`: Store node idle proof information (partial).<br />`service_prove`: Store node service proof information (partial).

3. **Challenge Deadline Storage**

Store the expiration time of the challenge
```rust
pub(super) type ChallengeDuration<T: Config> = StorageValue<_, BlockNumberOf<T>, ValueQuery>;
```
No primary key<br />Value: Block height of the deadline for BlockNumberOf

4. **Verification Deadline Storage**

The expiration time for consensus nodes to perform verification tasks
```rust
pub(super) type VerifyDuration<T: Config> = StorageValue<_, BlockNumberOf<T>, ValueQuery>;
```
No primary key<br />Block height of the deadline for BlockNumberOf.

5. **Idle Proof Consecutive Failure Counter**

Record the current number of consecutive idle proof failures for each storage node.
```rust
pub(super) type CountedIdleFailed<T: Config> = StorageMap<_, Blake2_128Concat, AccountOf<T>, u32, ValueQuery>;
```
Key: Wallet address of storage node for AccountOf.<br />Value: u32 failure count.

6. **Service Proof Consecutive Failure Counter**

Record the current number of consecutive service proof failures for each storage node.
```rust
pub(super) type CountedServiceFailed<T: Config> = StorageMap<_, Blake2_128Concat, AccountOf<T>, u32, ValueQuery>;
```
Key: Wallet address of storage node for AccountOf.<br />Value: u32 failure count.

7. **Consecutive Liquidation Counter**

Record the current number of consecutive liquidations for each storage node
```rust
pub(super) type CountedClear<T: Config> = StorageMap<_, Blake2_128Concat, AccountOf<T>, u8, ValueQuery>;
```
Key: Wallet address of storage node for AccountOf.<br />Value: u32 liquidation count.

#### Detail Design

1. **Generate Challenge**

**Function Overview**<br />In the absence of challenges and verification tasks in the current network, a challenge will be triggered at a random time point. The random number and block number for this challenge will be generated, the current state snapshot of the network will be captured, the bonus pool will be recorded, and one-tenth of the storage nodes in the current network will be drawn as challenge targets. Their computing power and the total computing power of this round of challenges will be recorded, and finally a timing task will be started. The above work will be completed in the off-chain worker.<br />**Trigger probability**: 1/2880, judged every 6s.<br />**Challenge proposal**: In order to prevent the results submitted by off-chain workers from being tampered with, the consensus node executes the results submitted by the off-chain worker to the chain. It will first be saved as a proposal. When the number of times the same proposal is submitted exceeds 2/3 of the executors in this round, the proposal takes effect and the challenge is officially generated.<br />**Challenge range**: Each time 10% of storage nodes in the entire network are drawn for challenge.<br />**Process Overview**

1. Confirm that there is no challenge in the current network.
2. Generate random results and determine whether to trigger a challenge based on the results.
3. Start the off-chain worker and determine whether this node is an executor. If not, end directly.
4. Determine whether the off-chain worker is in a locked state. If it is in a locked state, end this task.
5. Lock the off-chain worker.
6. Randomly draw the data segment number for this round of challenges, generate a random number for each drawn data segment number, record the current network bonus pool, and record the current block height.
7. Obtain a list of storage nodes in the entire network and randomly draw 10% of (non-lock and non-exit status) storage nodes as challenge targets for this round.
8. Traverse storage node information, accumulate total computing power as total computing power for this round of challenges, and save snapshots of each storage node.
9. Combine all storage node snapshots and network snapshots to send transactions to the chain and submit challenge proposals.
10. The same challenge proposal, when the number of submitters reaches 2/3 of the number of executors, the proposal is passed and officially generates a challenge. Interface definition

No input parameters.

2. **Proof Submission**

**Function Overview**<br />The storage node submits two aggregated proofs, and the chain network randomly assigns the proof information of the storage node to any consensus node for verification. Clear the storage node snapshot, record the storage node proof information, representing that the storage node has submitted proof for this challenge.<br />**Process Overview**

1. Determine whether the snapshot exists. The existence of a snapshot means that the storage node needs to submit proof.
2. Combine the snapshot information and the two aggregated proofs submitted by the storage node into new information.
3. Randomly assign the current information to consensus nodes.
4. Zero out the number of times the storage node is cleared.

**Interface**<br />SegmentBook.submit_proof

| Interface Parameters | Parameter Type | Parameter Description |
| --- | --- | --- |
| origin | OriginFor | Signature Source |
| idle_prove | BoundedVec<u8, T::SigmaMax> | Aggregated Proof Type Parameter |
| service_prove | BoundedVec<u8, T::SigmaMax> | Aggregated Proof Type Parameter |

3. **Verification Result Submission**

**Function Overview**<br />The scheduling node submits the verification result of the proof. If both pass, the storage node can obtain rewards. If one fails, the storage node will not be able to obtain rewards for this round. When the proof fails, it will not be punished immediately. Only when the proof verification fails twice in a row (the two types of service and idle will be calculated separately for the number of failures), the storage node will be punished.<br />To prevent scheduling from deliberately doing evil, the chain network will verify the SGX signature to ensure that the verification work is carried out in SGX.<br />**Process Overview**

1. Check that the signature scheduling node has this task verification task.
2. Confirm the TEE worker signature.
3. If there is an error in the idle proof verification result, after accumulating one more idle proof consecutive failure time, if the consecutive failure times reach two or more, the storage node will be punished for idleness. If there is no error, reset the idle proof consecutive failure count to zero.
4. If there is an error in the service proof verification result, after accumulating one more idle proof consecutive failure time, if the consecutive failure times reach two or more, the storage node will be punished for service. If there is no error, reset the service proof consecutive failure count to zero.
5. Only when both aggregated proofs pass will rewards be calculated for the storage node. If one fails, the reward will be cancelled.
6. The scheduling verification task ends and cleans up this task record.

**Interface**<br />SegmentBook.submit_verify_result

| Interface Parameters | Parameter Type | Parameter Description |
| --- | --- | --- |
| origin | OriginFor | Signature Source |
| miner | AccountId | Storage Node Account ID |
| sgx_signature | Signature | SGX Signature |
| idle_result | bool | Idle Proof Verification Result |
| service_result | bool | Service Proof Verification Result |

4. **Challenge Clearing**

**Function Overview**<br />Executed once at the initialization of each block. After the challenge expires, storage nodes that have not completed the challenge will be cleared, and the chain network will clear relevant information to trigger the next challenge.<br />**Process Overview**

1. Determine whether the challenge has expired. If it has not expired, end directly.
2. Traverse storage nodes that have not completed the challenge.
3. Increase the number of consecutive clearings of the storage node.
4. Punish the storage node to different extents according to the number of consecutive clearings. When it reaches 3 consecutive times, the storage node will be forcibly kicked out of the network.
5. Clear the storage node snapshot.
6. After all traversals are completed, clear the network snapshot.

5. **Verification Clearing**

**Function Overview**<br />When the scheduling node fails to complete the verification task within the specified time, it will be punished. The unfinished task will be assigned to other scheduling nodes to continue to complete and extend the validity period.<br />**Process Overview**

1. Determine whether the current verification validity period has ended.
2. Traverse the current list of unfinished consensus.
3. Punish the consensus, obtain its current list of unfinished tasks, and randomly select one responsible for consensus.
4. Determine whether the random result is not the same as the current traversal consensus. If it is the same, randomly select again.
5. Assign tasks to new consensus.
6. Accumulate the number of unfinished tasks and continue to traverse the next one.
7. Calculate the waiting time for verification based on the total number of unfinished tasks.
8. If all tasks are completed, clear the network snapshot.

**Interface**<br />No parameters.
## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
