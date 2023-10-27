# File upload process upgrade
---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 07 | File upload process upgrade | Elden Yang | Review | Standards | Core | 2023/10/26 |

## Abstract
​		The current file upload process faces a high probability of randomly allocating unavailable storage nodes, leading to slow file upload speeds and frequent failures. To address this issue, a solution is proposed to transfer the responsibility of selecting storage nodes to DeOSS. This solution upgrades the random allocation mechanism of storage nodes during the file upload process to one where storage nodes actively seek data from DeOSS and DeOSS autonomously selects specific storage nodes to send the data based on certain rules. This approach aims to enhance the proactive nature of storage nodes and improve file persistence efficiency.
## Motivation
​		In the current file upload process, the files uploaded by users are redundantly divided into segments, and these segments are randomly assigned to storage nodes by consensus nodes for storage. However, due to the inability of consensus nodes to determine the true status of storage nodes, there is a high probability of the designated storage nodes being unavailable. This leads to slow file upload speeds and frequent failures. To address the above issue, a solution is proposed to transfer the authority for selecting a storage nodes to DeOSS.
## Specification
**The Core Design Changes :** The process of file upload is being upgraded from random allocation of storage nodes by consensus nodes to a system where, after DeOSS issues an order, storage nodes actively seek data from DeOSS, and ultimately, DeOSS selects a specific storage node to send the data to.

**Process changes:**

1. DeOSS slices and duplicates files according to the specifications, and then submits an order on the blockchain. 
2. Storage nodes, upon retrieving the order, proactively request the files from the DeOSS that initiated the upload.
3. DeOSS selects storage nodes according to predefined rules and sends the file data to them. 
4. After storage nodes download and persist the data, they send transactions to the blockchain for reporting, and the subsequent steps remain the same as in the original process. 

**Potential risks:**
		(1) After DeOSS issues orders, there is a potential risk of a large influx of storage nodes requesting data from DeOSS, which could potentially lead to a service crash or overload in the DeOSS system. 
		(2) This situation could potentially lead to the emergence of a secondary market for DeOSS, where DeOSS selectively sends data only to storage nodes on a whitelist. 
		(3) Historical legacy issue: File size needs to be restricted during upload, as otherwise, it may result in excessively long order metadata, which could prevent the go-substrate framework from reading it properly.
**Programs to update:**
		CESS Node: Modifying the order metadata structure, changing the order declaration interface, and altering the storage node reporting completion of storage interface.
		DeOSS: Changing the upload rules, establishing rules for storage node selection and response to requests.
		Bucket: Modifying the file reception rules, order query mechanism, and establishing request rules. 
		go-sdk: Updating the corresponding interfaces and metadata structures.
		
![img3](https://github.com/jiuquxzy/CIPs/assets/121914086/6e1e5f0d-f9cf-4d10-8bb1-ef2712cb9d33)


_Order generation rules:_
		Current file specifications: segment: 64M, fragment: 16M
		Assuming a file size of "file_size," it will first be padded to the nearest multiple of 64MB, and then divided into "file_size / 64" segments.
		After applying a 1.5x redundancy, each segment will have 6 fragments. 
		The order will be divided into 6 data sets, with each data set containing (file_size / 64) fragments from different segments. 
_Storage node selection rules:_
		In sequential order of request, each request is processed, and based on a random seed result (e.g., with a 50% probability), the storage node's request is accepted. After confirming the connectivity of the storage node and checking for missing parts in the order metadata on the blockchain, the corresponding file data is sent to the storage node. Once the order's blockchain reporting process is completed or the number of data transmissions reaches the order's requirements, further requests related to this order are denied, and the request service for that order is closed. 
		Note:  If the number of data transmissions reaches the order's requirements for the storage node, the request reception is temporarily closed. After a certain interval, the storage node reporting status is checked. If the reporting is still incomplete, the service is reactivated to send the remaining data that hasn't been reported. If the number of data sent reaches the order's demand for the storage node, the reception of requests will be temporarily closed. After a period of time, the storage node reporting status will be checked. If the reporting is still not completed, the service will be reopened and the remaining unused data will be sent. Complete the reported data.
		The ideal storage node demand for a single order is 6. 
_Storage node request rules:_
		To prevent storage nodes from making requests too frequently, storage nodes can set up sleep periods based on the file size of the queried orders. During the sleep period, they can randomly choose a duration to sleep, and then resume making requests once the sleep period ends. If a request fails, appropriate actions can be taken based on the failure reason, and the storage node can wait for the next order polling cycle to repeat the above operation. 

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
