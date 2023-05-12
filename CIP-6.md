---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 17 | Improved exit process for storage nodes | EldenYang | Draft | Standards | Core | 23/05/10 |


## Abstract
This proposal is to adapt the file recovery mechanism and redesign the exit process of the storage node. For some special cases, such as during the random challenge proposal period, the storage node performs an exit operation and a mechanism has been established. The exit of the storage node will go through a lock-in period and a transfer period, and then the storage node will autonomously call the transaction to redeem the pledge.
This proposal also imposes strict restrictions on the behavior of storage nodes during the lock-in period and transfer period. When in the lock-in period or transfer period, the storage node will display the corresponding status for user confirmation. During the transfer period, the storage node will become a recovery target for the entire network. By making it more valuable to store service data than idle data, it encourages storage nodes across the network to transfer or recover data from recovery targets. At the same time, in order to facilitate the storage node to redeem the pledge as soon as possible, after transferring a certain number of bytes of data, a part of the deposit can be redeemed until all are redeemed.
## Motivation
1. The current exit mechanism of the storage node is not rigorous enough, which allows dishonest storage nodes to use the exit mechanism to evade impending punishment. If the storage node exits immediately, it will have a certain impact on the economy of the CESS network.
2. One problem currently existing in the network is how to deal with data loss. When a storage node exits, in order to prevent data loss, it is necessary to recover or transfer the data stored on that node.
## Specification
### Overall Process
1. The storage node calls the miner_exit_pre transaction to initiate a pre-exit transaction. The storage node will enter a one-day lock-in period (recorded in block height), during which the status will change to lock. If a challenge is received during this period, it must be completed normally.
2. After the lock-in period ends, the chain will automatically trigger the miner_exit function to execute the formal exit process of the storage node. The storage node status will change to exit and be marked as a recovery target, entering the transfer period.
3. Storage nodes across the network, upon detecting the existence of recovery targets, will transfer or recover data stored by the target according to their current idle resource situation and choose how much data to transfer. When all service data of the recovery target has been transferred to other storage nodes for storage, the transfer period is completed and the storage node can redeem its pledge and exit the network.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22121693/1683719832312-0ad8ca45-4354-4aea-8738-263a651d9c6e.png#averageHue=%23f6f5f4&clientId=u15a326e0-8d8f-4&from=paste&height=713&id=u844e2c28&originHeight=784&originWidth=821&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=49256&status=done&style=none&taskId=u98a8614f-c7c2-4657-bde3-abe5db89eb0&title=&width=746.3636201866406)
### Blockchain Node Interface
- Storage node pre-exit interface, `FileBank.miner_exit_pre`. 
- Storage node exit interface, `FileBank.miner_exit`. 
- Storage node redeem pledge interface, `FileBank.withdraw`.
#### Data Structure

1. **Storage Node Information Storage**
Stores some basic information of StorageNode. The service_space field will be synchronously recorded in the recovery target storage pool when StorageNode exits.
```rust
pub(super) type MinerItems<T: Config> = CountedStorageMap<
		_,
		Blake2_128Concat,
		T::AccountId,
		MinerInfo<T::AccountId, BalanceOf<T>, BoundedVec<u8, T::ItemLimit>>,
	>;
```
Primary key: T::AccountId Storage node wallet address.
Value: MinerInfo Storage node information.
Storage node information structure
```rust
pub struct MinerInfo<AccountId, Balance, BoundedString> {
	pub(super) beneficiary: AccountId,
	pub(super) peer_id: [u8; 52],
	pub(super) collaterals: Balance,
	pub(super) debt: Balance,
	pub(super) state: BoundedString,
	pub(super) idle_space: u128,
	pub(super) service_space: u128,
	pub(super) lock_space: u128,
}
```
`beneficiary`: Revenue account wallet address.
`peer_id`: The peer_id of the storage node.
`collaterals`: Pledge amount.
`debt`: Debt.
`state`: Storage node status.
`idle_space`: The idle space currently certified by the storage node.
`service_space`: The service space stored by the storage node.
`lock_space`: The space locked by the storage node.

2. **Storage Node Lock-in Period Time Storage**

Used to store the expiration time of the lock-in period, and also to perform the timing task of formal exit after expiration.
```rust
pub(super) type MinerLock<T: Config> = 
		StorageMap<_, Blake2_128Concat, AccountOf<T>, BlockNumberOf<T>>;
```
Primary key: AccountOf Storage node account address.
Value: BlockNumberOf Lock-in period end block number.

3. **Recovery Target Information Storage**
```rust
pub(super) type RestoralTarget<T: Config> = 
		StorageMap< _, Blake2_128Concat, AccountOf<T>, RestoralInfo<BlockNumberOf<T>>>;
```
Primary key: AccountOf Storage node account address.
Value: RestoralInfo Recovery target information.
Recovery target information structure
```rust
pub struct RestoralInfo<Account, Block> {
    // Whether to add this field depends on the specific situation during development
    pub(super) miner: Account, 
	pub(super) service_space: u128,
	pub(super) restored_space: u128,
    // Whether to add this field depends on the specific situation during development
	pub(super) cooling_block: Block,
}
```
`miner`: Wallet address of storage node.
`service_space`: Service space to be restored.
`restored_space`: Space currently restored.
`cooling_block`: Block height of cooling period.
#### Detail Design

1. **Storage node pre-exit**

**Function overview**
This function is set up to prevent storage nodes from performing exit operations during the challenge proposal period, resulting in state inconsistencies. The purpose is to prevent storage nodes from exiting immediately. If a pre-exit is performed during the challenge proposal period, after the challenge is officially generated, the storage node may still be one of the challenge targets and needs to complete the challenge. It should be noted that if the challenge fails and the storage node is frozen, the subsequent exit process cannot be completed. The storage node needs to make up the deposit and re-lock the pre-exit.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22121693/1683709495129-201a81c9-4213-4dcc-846d-ae89b04f279d.png#averageHue=%23fcfcfc&clientId=ub287f5ab-9998-4&from=paste&height=437&id=ued54d921&originHeight=481&originWidth=1155&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=29984&status=done&style=none&taskId=ud163796b-f820-4865-9f4e-5ea674f381b&title=&width=1049.9999772418635)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22121693/1683709770013-643000ff-5c28-40dc-824f-650de561309f.png#averageHue=%23fcfcfc&clientId=ub287f5ab-9998-4&from=paste&height=457&id=uc752c223&originHeight=503&originWidth=1159&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=33413&status=done&style=none&taskId=u0f73dde2-21aa-4ca9-b745-ff961f46503&title=&width=1053.636340799411)

After initiating the pre-exit process, the storage node will enter a lock period of one day (corresponding to 28800 blocks) and its status will change to lock. After performing the pre-exit operation, the storage node cannot authenticate space, store service data, or receive rewards. At the same time, a timed task will be started. After the lock period ends, the formal exit process will be executed.
**Prerequisite status requirements**: Positive
**Lock status restricted behavior**: (1) Prohibition of space authentication. (2) Prohibition of storing service data. (3) Prohibition of receiving rewards. (4) Will not receive new challenges (the random selection process will not select storage nodes in Lock status).
**Matters that need to be reminded to storage nodes**: Before pre-exit, you need to ensure that you currently have no rewards to claim. After being locked, you will not be able to receive new rewards.
**Process overview**

1. Check whether the storage node status is Positive.
2. Check MinerLock Storage to determine whether the storage node is in the lock period.
3. Update the storage node status to lock.
4. Lock the storage node, create a lock period, and store it in MinerLock Storage.
5. Create a formal exit timed task.

**Interface**
FileBank.miner_exit_pre

| Interface parameters | Parameter type | Parameter description |
| --- | --- | --- |
| origin | OriginFor | Signature source |

2. **Storage node exit**

**Function overview**
Triggered by the timed task of the previous step, after the lock period of the storage node ends, the chain network begins to execute the formal exit process for it. After checking that the storage node that has passed the lock period has no abnormalities, it will be changed to the exit status and designated as a recovery target. Transfer all service data stored by it to other storage nodes. The restricted behavior of storage nodes in the exit state is the same as that in the lock state.
Finally, unclaimed and unissued bonuses will be returned to the bonus pool.
Note: The formal exit process is irreversible. If the storage node wants to rejoin the network, it needs to wait for the redemption of the deposit and re-register as a storage node.
**Process overview**

1. Check whether the storage node status is lock.
2. Determine whether the lock period has reached its deadline. If it has not reached its deadline, end this process.
3. Record the current service space of the storage node, create relevant information about the recovery target, and store it in RestoredTarget Storage.
4. Return the unissued rewards and unclaimed rewards of the storage node to the bonus pool (one of the reasons why the process is irreversible).
5. Change the storage node status to exit.
6. Clear the reward table of the storage node (one of the reasons why the process is irreversible).

**Interface**
FileBank.miner_exit（root权限调用）

| Interface parameters | Parameter type | Parameter description |
| --- | --- | --- |
| origin | OriginFor | Signature source |
| miner | AccountOf<T> | Wallet address of storage node. |

3. Storage node redemption pledge

**Function overview**
After the service data of the storage node is transferred, the deposit can be redeemed, but the storage node needs to actively call the transaction for redemption. After the deposit is redeemed, the metadata of the storage node will be cleared, which also means that the exit process is over.
**Process overview**

1. Determine whether the storage node status is exit status.
2. Check the transfer status of the storage node’s service data. If it is not completed, end the process.
3. Return the deposit of the storage node.

**Interface**
FileBank.miner_withdraw

| Interface parameters | Parameter type | Parameter description |
| --- | --- | --- |
| origin | OriginFor | Signature source |

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
