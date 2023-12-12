# Penalty and reward mechanism adjustments for storage node
---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 10 | Penalty and reward mechanism adjustments for storage node | swowk | Review | Standards | Core | 2023/12/12 |

## Abstract
​		There is a significant number of storage nodes that have not received rewards but have been penalized in cess-testnet. This situation has led to widespread losses among storage nodes. To prevent the occurrence of the above situation, the penalty system for storage nodes will be modified, and the earning rate of storage nodes will be increased. Additionally, the bonus pool has accumulated due to TEE Workers being busy and unable to process storage node requests, resulting in an uneven distribution of rewards. It is necessary to modify the current storage node reward calculation method to address this issue.

## Motivation
​		The occurrence of losses in storage nodes will diminish their willingness to join the network and honestly store user data, leading to instability in the network. Therefore, it is necessary to adjust the rules for rewarding and penalizing storage nodes to make their earnings more reasonable and fair. This adjustment aims to create a positive incentive environment.

## Specification

### Adjustment Plan

1. Rewards will no longer be released in a linear manner; instead, they will be immediately released.
2. The penalty for miners who have not received rewards will be reduced and fixed at 500 CESS.
3. The base value of the bonus pool will be fixed.

### Details of the Plan

Changes to the Reward Release Mode:

After calculating the rewards for storage nodes, the rewards will be immediately transformed into pending rewards, awaiting acceptance by the storage nodes.

Modification to Penalty Rules:

New penalty rules have been introduced. When a storage node fails to submit storage proof for settlement, it will be assessed whether the storage node has received rewards.

(1) If rewards have been received, the existing penalty rules will be applied:
  - First failure results in a deduction of 5 times the pledged CESS amount (e.g., if miner A declares 4T, the pledged amount is 16000, so the penalty for the first failure is 1600 CESS).
  - Consecutive failures twice will result in a deduction of 5 times the pledged CESS amount.
  - Consecutive failures three times will result in a deduction of 5 times the pledged CESS amount, and the node will be forcibly removed from the network. The unpenalized portion can be reclaimed after the 180-day pledging period.

(2) If no rewards have been received, new penalty rules will be established:
   - First failure to submit results in a penalty of 500.
   - Second failure to submit results in a penalty of 500.
   - Third failure to submit results in a penalty of 500. Consecutive three failures to submit will lead to expulsion from the network.

Fixed Bonus Pool Base:

Currently, the challenge rules result in different completion times for each storage node challenge, leading to varying bonus pool amounts when calculating rewards. This disparity allows nodes with lower storage capacity but larger bonus pool amounts to receive substantial rewards, ultimately causing an unfair distribution of rewards.

Assuming the premise: In each Era (6 hours), the bonus pool amount increases by _R = 350,000 CESS_.
The reward base is fixed at _262,500 CESS (R * 75%)_.

(1) The tokens issued to the bonus pool in the first Era will be divided into two parts: a fixed bonus (_R * 75%_) and a fill bonus (_R * 25%_).

(2) In subsequent Eras, if the fixed bonus portion is less than _262,500 CESS_, it will be topped up to _262,500 CESS_. If the current fixed bonus quantity is sufficient, it will all be allocated to the fill bonus.

(3) When a storage node successfully completes a challenge and receives a reward _U_, the fixed bonus will be deducted by the corresponding _U_, and the fill bonus will be topped up by _U_ to the fixed bonus.

(4) At the end of each Era, any unreleased rewards will be deposited into the treasury.

Note: To prevent situations where the bonus pool funds are insufficient, an emergency function to transfer funds to the bonus pool will be added when necessary.


### Programs Involved in the Changes

1. **cess-node**
   
(1) Changes to the Reward Release Mode.

(2) Modification of Penalty Rules.

(3) Fixed Bonus Pool Base.

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
