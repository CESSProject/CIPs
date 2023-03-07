| CIP  | Title    | Author | Status | Type      | Category | Created  |
| ---- | -------- | ------ | ------ | --------- | -------- | -------- |
| 3   | Cache Miner and Retrieval Miner | Shaka  | Draft  | Standards | Core     | 23/03/07 |

## Background

CESS as a distributed storage network, file download is the most frequently used feature by users and the download speed is an important performance indicator that affects user experience.The CESS network needs a content delivery network (CDN) to avoid bottlenecks or mishaps that may affect data transmission speed and stability as much as possible, making file downloads faster and more stable.

## Motivatio

To provide a faster download experience for end users, CESS identified the bottlenecks that currently hamper file download speeds and has developed a proposal to solve them.

CESS’s current file download process: users obtaining file metadata from the blockchain through software packages. This metadata has the location of the storage miners where the file is located, allowing users to download the file directly from the storage miner.

Current bottlenecks of this process:

1. When files are uploaded, their data segments are stored on storage miners distributed worldwide. However, during file download, the user may be geographically distant from the storage miner where the data segments are stored, leading to poor network connectivity and resulting in slow download speeds or even download failures.
2. There is currently no motivation for storage miners to invest in high-bandwidth servers, as the rewards they receive are not linked to their ability to offer fast download services.

## Our proposal

CESS’s solution is to add a caching layer between end users and storage miners, resulting in faster download speeds. 

Note that this proposal introduces an additional download method, which incurs certain costs. End users can still choose the traditional method of directly downloading files from storage miners for free.

### Specification

- **Cache miners**

Cache miners are crucial in the file download process as they cache popular files and offer download services. They operate in a free market, where anyone can run a cache miner to earn rewards. Competition among cache miners is based on their ability to provide stable, fast, and cost-effective services that are competitive within the market.

- **Retrieval miners**

Organizations or enterprises that develop applications on CESS can run as retrieval miners themselves. Retrieval miners are responsible for selecting qualified cache miners to generate download orders and pay download fees to them. In addition, they shield the interaction details with CESS from the applications of the organization or enterprise and provide file download links.

- **Applications**

Applications can obtain fast and concise file download capabilities through retrieval miners without worrying about the interaction details with CESS.

### Overall Architecture

<img src="https://user-images.githubusercontent.com/15166250/223315297-5847b6fd-9835-4879-85d8-56f0ad872432.png" width = "50%" />

### File Download Process:

![file download](https://user-images.githubusercontent.com/15166250/223300199-04d8c589-e5c3-41e4-9d4d-9475d749b75b.svg)

1. Users query file metadata on the blockchain to obtain information on file data segments.
2. Users send requests to retrieval miners to obtain download links of data segments.
3. Retrieval miners send payment transactions to cache miners to pay for the download of data segments collectively and obtain the block height of the transaction.
4. Retrieval miners send requests to cache miners to obtain file data segment download links. Cache miners verify payment transactions based on the received block height and return a data segment download token.
5. Retrieval miners concatenate the cache miner's IP address and token to form the data segment download links and return them to the user.
6. Users download data segments from cache miners using the download links.

### Retrieval Miner 

#### Functionality:

##### 1. Recording of Cache Miner Information:

**(1) Function overview**

Retrieval miners regularly fetch cache miner information from the chain network, calculate node distances, and update local records.

**(2) Basic process**

1. Start a timer to execute the task periodically with a 10-second interval.
2. Get all cache miner from the chain.
3. Iterate the cache miner list and extract each cache miner's IP address + port to form an endpoint list.
4. Iterate the endpoint list and check if the cache miner exists locally based on the endpoint. If it exists, determine whether the information needs to be updated. If it does, update the record. If not, continue to iterate to the next endpoint. If it does not exist, determine the latitude and longitude based on the IP address, calculate the distance between nodes, and store it as a new record.

**(3) Flowchart**

<img src="https://user-images.githubusercontent.com/15166250/223336122-4bfd5fa6-c35a-4dc3-a10f-53cc4282fc32.svg" width = "30%" />

##### 2. Optimal miner calculation:

**(1) Function overview**

Based on the pricing of cached miners across the network and their proximity, select the optimal cached miner for downloading.

**(2) Basic process**

1. Iterate the received list of cached miners.
2. Filter out the minimum distance, maximum distance, minimum price, and maximum price from the miner list.
3. Calculate the miner's score using the algorithm below:
$Distance score = 10000 * [1 - |(miner distance - minimum distance) / maximum distance|]$
$Price score = 10000 * [1 - |(miner price - minimum price) / maximum price|]$
4. Select the cached miner with the highest score, which is the optimal miner.

##### 3. Generate order:

**(1) Function overview**

Provide users with order transaction services, automatically select the optimal miner based on the file blocks the user needs to download, send an order, and pay the fee.

**(2) Basic process**

1. After receiving the http request, first, determine whether the parameters comply with the rules.
2. Determine whether to generate orders based on block hash or block number.
3. Obtain file metadata based on the file hash, and determine whether the file block hash matches the file.
4. First, search for the cached miner who holds this file from the local cache. If the query result is zero, return the entire list of miners.
5. From the returned list of cached miners, select the most suitable cached miner based on the optimal miner algorithm.
6. Add a cache order for this miner.
7. Wait for the scheduled task, or when the local order list accumulates to the limit, submit this batch of orders together and send the transaction.

**(3) Flowchart**

<img src="https://user-images.githubusercontent.com/15166250/223335289-ad99f47b-79fb-4899-9806-ba7fdebe6730.svg" width = "50%" />

##### 4. Get file download link:

**(1) Function overview**

15 seconds after the user sends the order for caching, the user gets the file download link based on the HTTP request sent by cached miners and order ID. The link will be valid for a certain period, during which the user can download the file repeatedly.

**(2) Basic process**

1. Parse whether the order ID and cached miner in the parameters comply with the rules.
2. Based on the order ID, query the local order list to obtain the transaction hash.
3. Read the configuration file to obtain the account private key seed to generate a signature private key.
4. Use the private key to sign the transaction hash and order ID.
5. Pack the transaction hash, signature, order ID, and other metadata, and send an http request to the specified cached miner to obtain a token.
6. After successfully obtaining the token, concatenate it to form an http request link and return it to the user.

##### 5. Query cached miner information:

**(1) Function overview**

Users can obtain all cached miner information stored locally by retrieval miner by sending a http request.

**(2) Basic process**

1. Retrieve all local cached miners.
2. Iterate each cache miner, pack and return it to the user.

#### HTTP interface

Query the cache miner where the file is located, selecting one cache miner based on the unit price and network transfer rate from multiple dimensions.

| URL      | /file/{$hash} |
| -------- | ------------- |
| Method   | GET           |
| Request  | None          |
| Response | { <br> "result": true, <br> "data": { <br> "cacheMinerIp": "110.14.57.121" <br> } <br> }|

#### Database

- Information list of local cache miners (including on-chain data and local statistics on network transmission rates of cache miners)
- Relationship between files and the cache miners where they are stored

### Pallet-cacher

#### Storage

- **StorageMap<AccountId, CacherInfo>** <br> stores information of all cache miners

#### Transactions

- **register(CacherInfo)** <br> CacherInfo - payment receiving account, IP address, and price per byte to download, register cache miner
- **update(CacherInfo)** <br> updates cache miner information
- **logout()** <br> logs out the cache miner
- **pay(Vec(Bill))** <br> Bill - billing ID, payment receiving account, total amount, file hash, data segment hash, retrieval miners pay cache miners in batches separately for downloading file data segments during the validity period

### Cache Miner 

#### Functionality:

##### 1. Registration:

**(1) Function overview**

Send a transaction to the chain, register a specified account as a cache miner and broadcast node information to the entire network.

**(2) Basic process**

1. Initialize the chain client.
2. Read the configuration file for parameters such as IP, port, and price.
3. Organize metadata and send a registration transaction to the chain.

##### 2. Update Price:

**(1) Function overview**

Send a transaction to the chain to update the information status of the cache miner on the chain.

**(2) Basic process**

1. Initialize the chain client.
2. Read the configuration file for parameters such as IP, port, and price.
3. Organize metadata and send a registration transaction to the chain.

##### 3. Exit:

**(1) Function overview**

Send a transaction to the chain to delete the cache miner's information status on the chain.

**(2) Basic process**

1. Initialize the chain client.
2. Send an exit transaction to the chain.

##### 4. File Download:

**(1) Function overview**

Returns file data based on the user's download request link. If the file does not exist, register the caching task for that file.

**(2) Basic process**

1. Parse the request parameters, obtain the ticket, and check if the ticket is expired or used.
2. Check if the local cache is hit. First, check the cached file list. If it exists, it is considered a hit. If it does not exist in the list, check if the cache file exists in the specified directory. If the file matches, it is considered a hit.
3. If it is a hit, return the file directly. If it not, handle it differently depending on the situation: 
4. If the file is deleted on-chain, delete the ticket record and return an error message.
    - If it is in the list of failed caching files, delete the ticket record and return an error message.
    - If the file is lost, return the caching progress, also considered a success.

**(3) Flowchart**

<img src="https://user-images.githubusercontent.com/15166250/223345188-ac23ec18-f82c-409d-b8ad-f934e59ba29a.svg" />

##### 5. Cache Files:

**(1) Function overview**

Periodically execute cache tasks to download the file blocks in the current cache task from the storage node.

**(2) Basic process**

1. Iterate the cache task list.
2. Parse the file hash from the block ID. If there is no directory named after the file hash, create a directory with the file hash as its name.
3. Start the download task if the file does not exist, and download it from the storage nodes.
4. Check whether the file is downloaded successfully. If it fails, add the file block to the cache failure list.

##### 6. Archive the Hash List of Cached Files Periodically:

**(1) Function overview**

Periodically execute the archive task to archive the locally cached file hashes to the metadata.json file.

**(2) Basic process**

1. Iterate the list of cached files and organize the information format.
2. Rewrite the metadata.json file based on the organized information list.

##### 7. Clearing Strategy:

**(1) Function overview**

Set the maximum cache capacity, when the cached capacity reaches 95% of the maximum capacity, automatically cache files, release disk space until it reaches 80% of the maximum capacity.

**(2) Basic process**

1. Check if the current usage space is 95% of the maximum capacity. If it exceeds, start executing the task.
2. Randomly select files to delete, probabilities dynamically adjusted by the clearing space of this scheduled task and the current usage space. and generate a list.
3. Iterate the random list, add the files to the deletion queue one by one until the space requirements are met.

#### HTTP interface

- Query cache miner information

| URL      | /stats |
| -------- | ------ |
| Method   | GET    |
| Request  | None   |
| Response | { <br> "result": true, <br> "data": { <br> "geoLocation": "Asia", <br> "bytePrice": 10000, <br> "speed": 10485760, <br> "status": "active" <br> } <br> } |

- Query the cached file hash list

| URL      | /cached |
| -------- | ------- |
| Method   | GET     |
| Request  | None    |
| Response | { <br> "result": true, <br> "data": [ <br> "0xffee", <br> "0xffff" <br> ] <br> } |

- Query file

| URL      | /file/{$hash} |
| -------- | ------------- |
| Method   | GET           |
| Request  | None          |
| Response | { <br> "result": true, <br> "data": { <br> "price": 1048576000000, <br> "size": 104857600, <br> "shardCount": 10 <br> } <br> } |

- Download File

| URL     | /download |
| ------- | --------- |
| Method  | POST      |
| Request | { <br> "billId": "0xbbbb", <br> "hash": "0xffff" <br> } |
| Response | Stream |

**Users pay by for downloads directly**

- What motivates cache miners to join the CESS network?

The simplest solution to this problem is to have retrieval nodes pay cache miners for every download. As long as the price is right, this will bring cache miners into the network.

- This direct payment method has some advantages over the third-party subsidy download method.

First, each download process is a local exchange between two entities: a retrieval node and a cache miner node. This means that at the end of this exchange process, both parties get what they need, so there is no need for further arbitration or accounting processes.

Secondly, as the retrieval node needs to pay the cache miner node for the transaction, it can also prevent the retrieval node from launching a witch attack or distributed denial of service attack on the cache miner nodes.

## Copyright

Copyright and related rights waived via [CC0](https://github.com/CESSProject/CIPs/blob/main/CIP-0.md).

