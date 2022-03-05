# Proposal for Offering Storage Pallet to Substrate FRAME

## 1. Background

As a versatile blockchain framework, Substrate has a variety of modules (a.k.a. pallets) for developers to reuse. From resource management such as accounts and assets to utilities such as random number generators and schedulers, these existing pallets could meet the need of most developers' application scenarios. However, there is still room for improvement.

Recently we have a requirement to implement a data storage service on Substrate, and after checking through all existing pallets, we did not find one that meets our need. So we would like to develop a custom pallet to fulfill this purpose.

We are not talking about something very niche here. On the contrary, it is a common scenario that an application would continuously consume and generate various data, whether it is system, user, or just temporary levels, during operations. Many dApps have a large number of scenarios that require off-chain data storage services, such as NFTs. The quality of the storage service chosen will directly affect the performance and reliability of the entire application.

So we hope to offer Substrate/Polkadot community with pallets (and toolchains) dedicated for storage services that are compatible with current Substrate APIs. So developers only need to add tiny amount of code change to leverage CESS stable and secure data storage. We believe this will further enhance the development experiences when adopting Substrate and enrich the Polkadot ecosystem.

## 2. Shortcomings of the current scheme

There is only one pallet related to data storage in the existing Substrate FRAME, aka, [Transaction Storage Palllet](https://paritytech.github.io/substrate/latest/pallet_transaction_storage/index.html). It supports running an IPFS node side-by-side with Substrate and allowing data to be retrieved by IPFS after putting it in Substrate storage. However, its application scope is greatly limited due to its inherent characteristics and several defects in the following aspects.

1. Data need to be uploaded to the blockchain network. Although this data is not actually stored on the chain, they still incur additional gas costs and congestion, which is not suitable for large file storage.

2. All validator nodes need to establish IPFS service for themselves, which subject to many restrictions.

3. The development is difficult since the Substrate-based code needs to be greatly modified.

4. This only supports file upload on the Substrate side. Viewers need to retrieve it via IPFS clients.

## 3. Proposals

We design and implement a data storage service based on Substrate. On one hand, there is no need for a validator node to start additional service, and no major modifications to substrate-based code as well. Therefore, developers can easily integrate our storage service, whether it is a newly-built chain or an existing chain. On the other hand, by customizing the storage REST component, users could upload and download data conveniently without installing additional client programs.

### 3.1 High level design

Our proposal architecture is shown in the figure below, which consists of the Data Storage Pallet and custom-built Storage Sidecar (inspired by [Substrate API Sidecar](https://github.com/paritytech/substrate-api-sidecar)).

# ![Figure 1: Proposal architecture](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/substrate-builder-program/07.svg)

*Figure 1: Proposal architecture*

- **Data Storage Pallet**: Realize the recording and management of stored data. This pallet implements functions related to meta-data, e.g. root data management, data owner management, and data classification regarding the stored data.

- **Custom-built Storage Sidecar**: Provide RESTful service to interact with Data Storage Pallet. The difference from Substrate API Sidecar is that, in addition to the basic functions of interacting with the substrate-based chain, Storage Sidecar encapsulates storage-related API, including data storage and data retrieval. The data transmitted by users will eventually be stored in CESS Storage System through this interface.

### 3.2 Typical example

Data storage and retrieval are the two core features for a data storage service. They are illustrated in details below.

# ![Figure 2: Typical example process](https://raw.githubusercontent.com/CESSProject/W3F-illustration/main/substrate-builder-program/08.svg)

*Figure 2: Typical example process*

**Data Storage**

1. A user calls the data storage API of the Custom-built Storage Sidecar to upload the data file;
2. Forward the data to CESS by calling the encapsulated CESS API;
3. Once it is confirmed that the data has been written, Custom-built Storage Sidecar will call Extrinsic to record the relevant information of the data file on-chain;
4. CESS Storage System maintains the integrity and privacy of data throughout its life cycle.

**Data Retrieval**

5. A user calls the storage API of the Custom-built Storage Sidecar to retrieve the target data;
6. Custom-built Storage Sidecar to query on-chain data routing information;
7. Call the CESS data retrieval API with the routing info;
8. Retrieve and return the target data from CESS Storage System;
9. Return the target data to Custom-built Storage Sidecar;
10. Custom-built Storage Sidecar updates on-chain information, if necessary;
11. Return the target data to the user.

## 4. Team

We have a team of professionals in getting this done. The backgrounds of team members include but are not limited to cloud computing, consensus algorithms and distributed storage. Most of them have been working in their respective fields for many years and have rich industry experience and solutions. The team members are distributed in the UK, the US, China and India, ranging from research scholars and cryptography experts to senior technical managers and Substrate development engineers.

So far, one of the team's project [**CESS**](https://github.com/CESSProject/cess) is gradually integrating into the Polkadot ecosystem. Won the 1st Place in Polkadot Hackthon APAC Edition in 2021, pass all W3F Grants Program milestone deliveries on January 25, 2022, and officially join the Substrate Builder Program on February 14, 2022. The team is currently actively communicating and cooperating with other Polkadot teams and projects.

## 5. Summary

The above is the entire content of the proposal for offering storage pallet to Substrate community. We would like to get feedback about this proposal and if it fits within overall Polakdot/Substrate ecosystem development.

