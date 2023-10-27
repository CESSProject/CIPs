# Idle Space Proof Algorithm Proof and Verification Process Optimization
---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 08 | Idle Space Proof Algorithm Proof and Verification Process Optimization | Valor Xu | Review | Standards | Core | 2023/10/26 |

## Abstract
​		The current version of the idle space proof algorithm has identified inefficiencies in the random challenge proof generation and verification processes. To address these issues, we are considering optimizing key computational processes in the generation and verification of random challenge proofs to reduce unnecessary redundant calculations. This optimization aims to improve the speed of both proof generation and verification. Through simulated real-world scenarios, these optimized processes have shown a significant enhancement in the efficiency of proof generation and verification during the random challenge process.

## Motivation
​		In the current version of the idle space proof algorithm, there is a significant amount of redundant computation in both the random challenge proof generation and verification processes. This redundancy has led to an overall lower efficiency, making it challenging to maintain a considerable gap between idle file generation speeds. This inefficiency can pose security risks, and the slow proof generation speed may result in some storage nodes failing to pass random challenges on time. Slow verification speeds can also consume a significant amount of TEE Worker processing time, affecting the overall efficiency of the network's random challenge process.
## Specification
**Optimization of the random challenge proof generation and verification process. **

​		In the previous version, the time-consuming part of the random challenge proof generation and verification process was primarily associated with the generation and verification of the accumulator's Merkle proof. The challenge process involved all the idle data, and, therefore, it required calculating evidence chains for each idle file in the accumulator (evidence chains from the bottom-level accumulator to the top-level accumulator). The evidence chain consists of four layers, and when dealing with a large number of files, generating and verifying these evidence chains involved a significant amount of large integer exponentiation operations, consuming a considerable amount of time.

​		However, for a tree-like structure with multiple levels of accumulators, multiple idle files share a common bottom-level accumulator and a path to the top-level accumulator. This allows for the simplification of the evidence chain, where all nodes with a common branch path can share their upper-level evidence chain. This simplification aims to streamline the proof generation and verification process. As a result, the number of exponentiation operations can be reduced from $4N$ to $N+N^{2/3}+1$, theoretically improving the speed by more than twofold.

![img1](https://github.com/jiuquxzy/CIPs/assets/121914086/aac2a684-d9c4-48ba-bc67-5463c12a5521)


**MHT computational performance optimization**

​		In previous versions, the Merkle Hash Tree (MHT) path proof algorithms involved calculating the complete MHT (Merkle Hash Tree) and then extracting the path proof from a specific leaf node to the root node. When the MHT is large, the MHT calculation process becomes the most time-consuming operation in the entire proof generation process. To address this issue, introducing auxiliary space to divide the entire MHT into several subtrees is considered. This allows for the calculation of only the subtree containing the specific leaf node, which speeds up the path proof calculation. This process can optimize the majority of calculations and is a form of MHT pruning algorithm.

![img2](https://github.com/jiuquxzy/CIPs/assets/121914086/d7dc5c73-7c91-42ea-b384-4ee5e330506a)


​		1024 leaf nodes, and taking the 6th level nodes from the top as auxiliary data (2KB) can save more than 90% of the calculation. Based on experimental results, this optimization can provide at least an 85% performance improvement for the entire proof calculation process. Moreover, for every 16GB of idle data, it only requires an additional 1MB of auxiliary space, making it a very cost-effective solution.

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
