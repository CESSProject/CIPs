# Proof of Data Reduplication and Recovery
## 1.Background
With the emergence of Storj, Filecoin and other projects, we know that it is feasible to build a decentralized storage network with the help of blockchain. Moreover, users are more willing to participate in the construction of a decentralized storage network by contributing their idle storage resources to obtain certain benefits. However, how to effectively prevent users from cheating is one of the most critical issues facing decentralized storage. The most common cheating behavior is storage space fraud, that is, storage miners provide space that does not match the storage space reported by their system to achieve the purpose of defrauding incentives. The second is outsourcing attack. In order to provide data reliability, users usually consider store multiple copies of data on different miners, and the owners of these miners are likely to be owned by the same person. When a miner is challenged, he can get data from other miners to complete the current verification. another attack behavior is time and space attacks, when miners learn the verification rules, they may only go online for storage at the moment when verification is required to save costs.

In response to these attacks, Boneh et al. innovatively proposed mechanisms such as proof of storage, proof of replication and proof of space-time. By programming a large amount of random data, it is hard for miners to cheat, and these methods have been proved to be effective in theory and practice. However, in order to resist mining fraud, FIL needs to spend a lot of computation on the two parts of encoding and decoding. So far, FIL still has not found an efficient decoding algorithm, resulting in extremely low efficiency for users to retrieve their own stored data. In response to this problem, CESS proposed a proof of data reduplication and recovery (PoDR2) based on data possession. It uses the homomorphic signature feature, combined with the challenge of space-time proof, to achieve a storage proof mechanism with the same effect as FIL. Since the decoding stage is no longer required, the coding efficiency of CESS is increased by ten times, and the decoding efficiency is increased by dozens of times, which greatly improves the QoS of data.

## 2.Motivation
### 2.1 What is proof of replication?
Proof of replication is an implementation of proof of storage, in which the prover P can submit a certificate to the verifier V to prove that he indeed has a copy _Mi_ of a certain data _M_ on his storage device. For example, the prover P is entrusted by the network to store n independent copies of the data _M_. When V challenges P, P needs to prove to V that P does store a copy _Mi_ of every _M_.

The core of proof of replication is to ensure that the prover does store an independent copy of a file.

Proof of replication allows a verifier to challenge whether the prover has stored a copy of the data. So how can you prove that the data is reasonably stored for a certain period of time, rather than discarded after the challenge? A straightforward solution is to use a strategy to challenge the prover P at regular intervals, or at random checkpoints. For each challenge, P needs to generate a proof.

### 2.2 What are the attack models for proof of replication?
Before introducing PoDR2, we can first think about what kind of attacks may face for those proof schemes.

**Assumption**: If the prover P can correctly respond to the challenge of the verifier V, it means that the prover P does have a copy _Mi_ of a certain data _M_ at that moment.

Under such assumptions, it is still impossible to prove that P continues to have the copy of the data for a period of time. There are 3 reasons for this:

- Sybil attacks: the attacker may pretend to physically store many copies by creating multiple identities, but only one is actually stored.
- Outsourcing Attacks: Relying on the ability to quickly obtain data from external data sources, attacker may promise to store more data than their actual physical storage capacity.
- Generation attack: When the network initiates a challenge, the attacker temporarily generates data in some way, and the actual amount of data stored is less than the amount of data declared to the network.

### 2.3 What is the bottleneck of FIL?
FIL's security assumptions are as follows:

- Whenever a miner is challenged by the network, he must submit proofs back to the network within a certain period of time. There is a time-bound.
- If the miner has the challenged copies, he will be able to respond instantly.
- Otherwise, if the miner is missing the challenged copies, he needs to seal (encode), which will be much longer than the period of proof submission.
- Therefore, the miner has to keep the copies in order to pass the proof.

Simply put, the core of its security assumption is that the time spent on sealing replica must be much longer than the proof period. However, this trade-off will directly lead to inefficiencies in  decoding data, which greatly affects practicality.

Thus, can we design a mechanism that protects against the types of attacks described above, but also without the timing assumptions?

## 3.Our proposal
### 3.1 Ideas
The security assumptions of our proposed scheme are as follows:

- Whenever a miner is challenged by the network, he can submit proofs back to the network at any time.
- If the miner has the challenged copies, he will be able to respond instantly.
- And if the miner is missing the challenged copies, he needs to regenerate the copy and sign it which is not possible in our scheme.
- Therefore, the miner has to keep the copies in order to pass the proof.

It can be seen that the core part of our security assumption is that miners cannot rely on themselves, or collude with any others, to regenerate copies. Under this premise, miners can only store copies carefully. He has no way to recover it since the copies is lost, even under the assumption of no time limit.

### 3.2 Prototype design
The scheme design can be theoretically divided into modules such as trusted computing, random coding, and proof of data possession.

#### (1) Trusted computing
The consensus nodes in the CESS network are all configured with the Trusted Execution Environment (TEE). It is also unable to detect the data running in the TEE and control the logic running in the TEE for a consensus node. Therefore, we can safely set the encryption and decryption in the TEE of the consensus node.

#### (2) Random coding
Since each piece of data in the CESS system will exist in the form of multiple copies, we need to perform random encoding processing on each copies, so that the encoded results of each copies are different. Random encoding refers to the random number generated by a cryptographically secure random number generator, and the plaintext are input, and the encoded data is output. For the encoded data, if the random number generated above is unaware to miner, the plaintext cannot be detected. To make this process credible, it also needs to be executed in the TEE.

#### (3) Proof of data possession
The proof of data possession process can be briefly described as the following steps: 1) The user uploads the file to the trusted entities; 2) The trusted entities divide the plaintext of the file and sign the original data segment with the user's private key; 3) The trusted party sends the the signed data package (including public key) is distributed to the storage node; 4) After that, at any point in time, the trusted entities can initiate an audit on the storage node to verify the integrity of the data.

The overall flow of the scheme is shown below.

# ![Figure 1: Schematic flow](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/CIPs/PODR2/podr2-1.svg)
_Figure 1: Schematic flow_

## 4.Implementation
### 4.1 System architecture
The system architecture is shown in the figure below, and the relevant entities are introduced as follows:

# ![Figure 2: Proposal architecture](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/CIPs/PODR2/podr2-2.svg)
_Figure 2: Proposal architecture_

- **User**: An entity that has a need to store data, which can be an individual, application, or organization.
- **CESS Portal**: The entrance for external interaction with CESS, which can be client program, web application or SDK. Provide supporting functions such as data upload and download around data storage services.
- **CESS Scheduler**: Data will first be forwarded to the entity via CESS Portal. Mainly responsible for three types of operations: 1) pre-processing data, including data replication, random coding, slicing and other operations; 2) Sign the processed user data in TEE; 3) At the same time, as the verifier, It is responsible for verifying the data and other operations.
- **CESS Bucket**: Be responsible for receiving and storing user data from CESS Scheduler. The data in each bucket will be organized in a standardized architecture and respond to random queries of the blockchain in each subsequent cycle.
- **CESS Network**: The blockchain network implemented by CESS protocol which is responsible for maintaining the consistency of system operation and data.

### 4.2 Algorithm description
The algorithms included in PoDR2 can be roughly divided into data pre-treated, setup, integrity verification, bucket editing, operation of idle space, location and recovery, etc. the specific description is as follows.

#### 4.2.1 Pre-treated
- **duplicate**: It is executed by the CESS Scheduler and is responsible for copying the data into a number of copies. The default is 3 copies.
- **rand-encode**: It is executed by CESS Scheduler (TEE required), and each copies is randomly coded to generate different data contents.
- **slice**: Executed by CESS Scheduler, each copies is sliced according to fixed specifications to obtain a set of data segments.
- **distribute**: It is executed by the CESS Scheduler to randomly allocate data segments to CESS Buckets of different miners.

#### 4.2.2 Setup
- **key-gen**: It is executed by the CESS Scheduler (TEE required) to generate key pairs (_sk_, _pk_).
- **sign-gen**: It is executed by the CESS Scheduler (TEE required). With the private key sk and data segment as the input, the signature information and the relevant Merkle Hash Tree are output.

#### 4.2.3 Integrity verification
- **gen-proof**: It is executed by the CESS Bucket. The proof is generated according to the random challenge on the blockchain in each cycle.
- **verify-proof**: It is executed by CESS Scheduler to verify the correctness of the proofs.

#### 4.2.4 Bucket editing
- **append**: The CESS Scheduler cooperates with the CESS Bucket to ensure that the new data segment is successfully added to the CESS Bucket.
- **delete**: The CESS Scheduler cooperates with the CESS Bucket to remove the targeted data segment in the CESS Bucket.

#### 4.2.5 Idle space operations
- **rand-pad**: Executed by CESS Scheduler (TEE required), create a data segment and fill it with random data.
- **replace**: The CESS Scheduler cooperates with the CESS Bucket to realize the data segment replacement function in the CESS Bucket.

#### 4.2.5 Location and recovery
- **locate**: The CESS Scheduler is responsible for locating the fault data segment when the verification fails.
- **recover**: It is executed by the CESS Scheduler to extract and generate new parallel data segments from the replica data segments and replace the fault data segments.

## 5. Details of process
In the system design of CESS, the minimum unit of data storage is data segment. According to its properties, it can be divided into idle segment and service segment. CESS Bucket can display the capacity space it provides for the system by stacking idle segments. CESS Network statistics and management of these storage spaces. When the user data comes in, the CESS Scheduler processes the user data into service segments and randomly assigns them to the CESS Bucket. The storage of each service segment will replace the existing idle segment.

### 5.1 Filling of idle segments
The filling of idle segments is a process in which the CESS Bucket provides storage space to the CESS Network. At the beginning, each CESS Scheduler in the system will negotiate a periodic key. Then, the CESS Scheduler will consider the declared capacity and current capacity of the CESS Bucket of the storage miner. If the former is greater than the latter, it means that the bucket can continue to be filled. The steps of sending and filling can be divided into pre-processing and filling. The pre-processing includes _rand-pad_ and _sign-gen_ algorithms, and the proof algorithms includes _gen-proof_ and _verify-proof_.

# ![Figure 3: Process of padding idle storage](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/CIPs/PODR2/podr2-3.svg)
_Figure 3: Process of padding idle storage_

### 5.2 Replacing of service segments
Replacing of service segments is the process by which CESS stores user’s data. After the data comes in, the CESS Scheduler will pre-process the data, including copying, random encoding, slicing, and signing. After the pre-processing is completed, the service segment is sent to the CESS Bucket for replacement. Once the replacement is complete, Bucket needs to periodically prove the data segment’s integrity.

# ![Figure 4: Process of uploading user file](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/CIPs/PODR2/podr2-4.svg)
_Figure 4: Process of uploading user file_

