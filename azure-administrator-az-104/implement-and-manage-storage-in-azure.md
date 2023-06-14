---
description: https://learn.microsoft.com/en-us/training/paths/az-104-manage-storage/
---

# Implement and manage storage in Azure

## Configure Storage Accounts

Storage Accounts are also known as Azure Storage.

Azure Storage can store files, messages, tables, file shares, working data, such as websites, mobile apps, desktop apps, and is also used by IaaS VMs and PaaS services.

Azure Storage supports three data categories:

* Virtual Machine Data - Disks (persistent storage) and files (file shares).
  * Azure Managed Disks
* Unstructured Data - Non-relational data. Mixed information stored together that may not be related.
  * Azure Blob Storage - Scalable, REST-based object store.
  * Azure Data Lake Storage - Hadoop Distributed File System (HDFS) as a service.
* Structured Data - Relational data with a shared schema, often contained in tables.
  * Azure Table Storage - Tables are an autoscaling NoSQL store.
  * Azure Cosmos DB - Globally distributed database service.
  * Azure SQL Database - Fully managed database-as-a-service built on SQL.

Storage Account tiers:

* Standard
  * HDD backed.
  * Lowest cost per GB
* Premium
  * SSD backed.
  * Low latency performance

<mark style="background-color:orange;">Storage Accounts can't be converted from one tier to another.</mark>

### Azure Storage Services

#### Azure Blob Storage

Object storage optimized for storing unstructured data such as text and binary data.

Use cases: Serving images and documents to a browser, storing files for distributed access, streaming audio, and video, storing data for backup and restore, disaster recovery and archiving.

Objects can be accessed via HTTP and HTTPS, NFS protocol, URLs, REST API, Azure CLI, PowerShell, or a client library in multiple languages (.NET, Java, NodPHP,,s, Python, PHP, and Ruby).

#### Azure Files

Network file share. Can be accessed using Server Message Block (SMB) protocol and Network File System (NFS) protocol, REST, and storage client libraries.

Multiple VMs can share the same files with both read and write access.

#### Azure Queue Storage

Used to store and receive messages, which can be up to 64KB in size. They are used to storage lists of messages to be processed asynchronously.

#### Azure Table Storage (Azure Cosmos DB)

Azure Table Storage is part of Azure Cosmos DB. Fully managed NoSQL database service. Cost-effective serverless and automatic scaling options. New service called Azure Cosmos DB Table API provides throughput-optimized tables, global distribution and automatically secondary indexes.

### Storage Account Types

Storage Account types:

| Storage account                                                                                                                  | Supported services                                                                        | Recommended usage                                                                                                                                                                                                                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [**Standard** **general-purpose v2**](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-upgrade) **(GPv2)** | Blob Storage (including Data Lake Storage), Queue Storage, Table Storage, and Azure Files | Standard storage account for most scenarios, including blobs, file shares, queues, tables, and disks (page blobs).                                                                                                                                                                   |
| [**Premium** **block blobs**](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-block-blob-premium)             | Blob Storage (including Data Lake Storage)                                                | Premium storage account for block blobs and appends blobs. Recommended for applications with high transaction rates. Use Premium block blobs if you work with smaller objects or require consistently low storage latency. This storage is designed to scale with your applications. |
| [**Premium** **file shares**](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-create-file-share)            | Azure Files                                                                               | Premium storage account for file shares only. Recommended for enterprise or high-performance scale applications. Use Premium file shares if you require support for both Server Message Block (SMB) and NFS file shares.                                                             |
| [**Premium** **page blobs**](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-pageblob-overview)               | Page blobs only                                                                           | Premium high-performance storage account for page blobs only. Page blobs are ideal for storing index-based and sparse data structures, such as operating systems, data disks for virtual machines, and databases.                                                                    |

<mark style="background-color:orange;">All storage account types are encrypted with Storage Service Encryption (SSE) for data at rest.</mark>

### Replication

Storage accounts are always replicated.

#### Locally Redundant Storage (LRS)

Lowest cost replication. Replicates data in the same datacenter. Useful in cases of data loss. Protects against single hardware failure.

Supported storage accounts: GPv1, GPv2, Blob

#### Zone Redundant Storage (ZRS)

Replicates data across three storage clusters (availability zones) in a single region. Each AZ is autonomous from each other. Protects against local AZ issues (networking, disasters). Isn't available in all regions.

Supported storage accounts: GPv2

#### Geo Redundant Storage (GRS)

Replicates data to a secondary region. Protects against a regional outage or disaster. There are two GRS options:

* GRS - Data is available to be read in the secondary region only if Microsoft initiates a failover from the primary to secondary region.
* Read-access geo-redundant storage (RA-GRS) - Data is always available to be read from the secondary region.

Supported storage accounts: GPv1, GPv2, Blob

#### Geo-Zone Redundant Storage (GZRS)

Best replication solution. Mix of GRS and ZRS. Data is available to read/write in three availability zones and available in another region (GRS or RA-GZRS).

Supported storage accounts: GPv2

### Access Storage

Every object storage in Azure Storage has a unique URL address. Default endpoints are `<storage_account_name>.<blob/table/queue/file>.core.windows.net`

You can also configure custom domains to access blob data by mapping a custom domain and subdomain to the blob or web endpoint.

Storage account names must be globally unique.

### Secure Endpoints

You can limit storage account access from the network by restricting access to selected virtual networks, subnets or IP addresses or by using a private link.

## Configure Azure Blob Storage

Blob Storage uses three resources to store and manage data:

Storage Account > Container > Blob

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A blob can only exist within a container, which can storage an unlimited amount of blobs and a storage account can store an unlimited number of containers.

Container access levels:

* Private (default) - No anonymous access.
* Blob - Allows anonymous public read for the blobs.
* Container - Allows anonymous public read (blob) and list (container)

Access tiers:

| Compare                          | Hot tier                                               | Cool tier                                              | Archive tier                                             |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------- |
| **Availability**                 | 99.9%                                                  | 99%                                                    | Offline                                                  |
| **Availability (RA-GRS reads)**  | 99.99%                                                 | 99.9%                                                  | Offline                                                  |
| **Latency (time to first byte)** | milliseconds                                           | milliseconds                                           | hours                                                    |
| **Minimum storage duration**     | N/A                                                    | 30 days                                                | 180 days                                                 |
| **Usage costs**                  | Higher storage costs, Lower access & transaction costs | Lower storage costs, Higher access & transaction costs | Lowest storage costs, Highest access & transaction costs |

