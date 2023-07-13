# CIP-6： Introduce staking score for validator election

***

| CIP | Title              | Author | Status | Type      | Category | Created  |
| --- | ------------------ | ------ | ------ | --------- | -------- | -------- |
| 20  | Consensus Election | Shaka  | review | Standards | Core     | 23/07/05 |


### Abstract <a href="#abstract" id="abstract"></a>

The consensus node election mechanism introduces a staking score, where the amount of stake affects the probability of being elected as a round-robin node. This encourages stable operation of consensus nodes, attracts nominators, and increases the chances of being elected as a round-robin node. Stakeholders can also share in the consensus block rewards. Additionally, it reduces the entry barrier for new consensus nodes, increases the number of consensus nodes, and promotes decentralization of the CESS network.

### Motivation <a href="#motivation" id="motivation"></a>

Consensus Nodes: Reduce the cost of stake participation and distribute the risk of token price volatility.&#x20;

Users: Earn interest by staking their CESS tokens.&#x20;

Market: Reduce token liquidity on the trading market.

### Background <a href="#hhcye" id="hhcye"></a>

Currently, miners need to stake 1 million to become a consensus node, and only one account (address) is allowed to stake as a consensus node.&#x20;

Round-robin node elections select the top 11 consensus nodes with the highest final scores.&#x20;

Final score = 80% \* Reputation Score + 20% \* Random Number

### Specification <a href="#specification" id="specification"></a>

#### Overall Process <a href="#thvyi" id="thvyi"></a>

Staking Threshold:

1. Holders staking a minimum of 300,000 CESS can become candidate nodes. The root account calls the staking.setStakingConfigs transaction and sets minValidatorBond to 300,000.
2. Consensus nodes can choose to open or close node staking financing. Holders (individuals) can participate as nominators and can nominate only one consensus node. The total staked amount of the consensus node and its nominator(s) is considered the staked amount during the round-robin node election.
3. The staked amount threshold for competing as a round-robin node is 3 million CESS. The staked amount affects the staking score during the election, which in turn affects the final score. There is no upper limit on the staked amount for consensus nodes. Final score = 50% \* Reputation Score + 30% \* Staking Score + 20% \* Random Score

Staking Score

At the moment of election triggering, candidate nodes with a staked amount less than 3 million cannot compete. The staked amount of the node with the highest staked amount among all candidates is set as the benchmark with a staking score of 100. The staking score of other nodes is calculated as (node's staked amount / maximum staked amount) \* 100.

Example：

| Nodes | Staked Amount |  Staking Score            |
| ----- | ------------- | ------------------------- |
| A     | 1,000,000     | 0 (directly disqualified) |
| B     | 3,000,000     | 30                        |
| C     | 10,000,000    | 100                       |

#### Staking Reward Distribution

consensus nodes can set a commission rate to cover machine operation costs. The commission is set as a percentage of the block reward and is directly paid to the consensus node. The remaining portion, excluding the commission, is distributed propotionally based on (staking amount of the node / total staked amount of that node).

Example:&#x20;

A stakes 1 million CESS

Total staked amount for the consensus node is 4 million CESS (A represents 25% of this).&#x20;

Commission rate is 10%. Block reward is 100,000 CESS.

\-------------------------------------------------------

Reward for A is calculated as: 100,000 CESS x 90% x 25% = 22,500 CESS&#x20;

Commission for the consensus node is: 100,000 CESS x 10% = 10,000 CESS

### Copyright <a href="#copyright" id="copyright"></a>

Copyright and related rights waived via CC0.
