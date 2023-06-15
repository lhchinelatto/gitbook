---
description: >-
  https://learn.microsoft.com/en-us/training/paths/az-104-manage-compute-resources/
---

# Deploy and Manage Azure Compute Resources

## Configure Virtual Machines

Virtual machine name is used as the computer name on the OS configuration. 15 char on Windows and 64 char on Linux, not trivial to change later.

Like other services, not all VM configurations are available on each region due to different hardware available.

Azure Managed Disks handle Azure storage account creation and management in the background for you. Azure Managed Disks are actually page blob storage virtualized disks.

Billing is split in compute and storage:

* **Compute**: Priced on a per-hour basis and billed on a per-minute basis. You're not charged if the VM is deallocated. There are two payment options:
  * **Consumption-based**: You pay for compute capacity by the second. Best for short-term and unpredictable workloads that can't be interrupted.
  * **Reserved Virtual Machine Instances (RI)**: You pay up-front for a VM for one or three years and get up to 72% price saving compared to pay-as-you-go pricing. You can easily exchange or returned for a fee. Best if you have workloads that must run continuously or need budget predictability and can commit to using the VM for a long time.
* **Storage**: The state of the virtual machine has not impact on storage costs. Storage costs uses the storage pricing model.

VM Classifications:

* **General purpose**: Balanced CPU-to-memory ratio (test/dev/small to medium db/low to medium traffic web server)
* **Compute optimized**: High CPU-to-memory ratio (medium traffic webserver/network appliances/batch processing/application servers)
* **Memory optimized**: High memory-to-CPU ratio (database servers/medium to large caches/in-memory analytics)
* **Storage optimized**: High disk throughput and I/O (big data/SQL and NoSQL databases/data warehousing/large transactional databases)
* **GPU**: Graphic rendering VMs, single or multiple GPUs (model training/deep learning)
* **High performance computes**: Fastest and most powerful CPU with optional high-throughput network interfaces (RDMA) (fast performance workloads/high traffic network)

You can resize the VM to any size available in the region. Resizing a VM require a restart and can change configuration settings like the IP address.



























