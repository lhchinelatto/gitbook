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

All VMs have at least two disks (OS and t mp). Disks are stored as virtual hard disks (VHD). Temp disk data may be lost during operations such as moving to a new host. Do not store data on temp disks.

Data disks are registered as SCSI drivers and the size of the VM determines how many data disks you can attach.

Considerations on disks:

* You can choose premium storage for higher performance on SSD disks.
* Using multiple disks, a VM can get up to 256 TB of storage.
* Managed disks are stored as page blobs. They're named managed because the concept of storage account, container and blobs is transparent.
* Available disk types are: Ultra SSD, Premium SSD, Standard SSD and HDD.

Connecting to a VM using Azure Bastion:

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Windows connection uses RDP (Remote Desktop Protocol) which provides a graphical user interface. <mark style="background-color:orange;">RDP default port is 3389</mark>.

Linux connection uses SSH (Secure Shell Protocol).

Azure Bastion is a fully platform-managed PaaS service providing secure and seamless RDP/SSH connection to your VMs over SSL in the virtual network it's provisioned. Using Bastion, VMs doesn't need public IPs. With Bastion you don't need to expose RDP and SSH ports on the internet.

## Configure Virtual Machine Availability

Availability Sets ensures that the VMs runs across different physical servers, racks, storage and network switches in the same DC thus protecting your workload from hardware failure.

* All VMs must perform the identical functionalities.
* All VMs must have the same set of software installed.
* A VM can only be added to an availability set at creation time.

VMs part of an availability set are placed in update and fault domains.

Update domain is a group of nodes that upgrade/reboot together. During planned maintenance, only one update domain is reboot at a time. Default is five update domains, up to 20 configured by the user.

Fault domain is a group of nodes in the same physical unit of failure, sharing same hardware.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

An Availability Zone in Azure is a combination of a fault domain and an update domain. If you create three or more VMs across three zone in an Azure region, the VMs are distributed across three fault domains and three update domains.

Each availability zone is made of one or more datacenter within an Azure region, and there is a minimum of three availability zones per region.

Virtual machine scaling can be vertical or horizontal:

* Vertical scaling (scale up/down): Increase/decrease a VM size depending on the workload.
* Horizontal scaling (scale out/in): Increase/decrease the number of VMs depending on the workload.

### Virtual Machine Scale Sets

VM Scale Sets are a resource used to deploy and manage a set of identical VMs, in order to achieve true autoscaling, by automatically (or manually) increasing or decreasing the number of the VMs as demand changes.

* All VMs are created from the same OS image and configuration.
* Supports Azure Load Balances for basic layer-4 traffic distribution and Azure Application Gateway for advanced layer-7 distribution and SSL termination.
* Supports up to 1000 VMs with standard images and 600 with custom images.

Orchestration modes:

* Uniform: Identical instances from a chosen model.
* Flexible: You manually create and add a VM of any configuration to the scale set.

Autoscaling options:

* Minimum VM number.
* Maximum VM number.
* Scale out (increase VM number):
  * CPU threshold (%)
  * Duration in minutes.
  * Number of instances to increase by.
* Scale in (decrease VM number):
  * CPU threshold (%)
  * Number of instances to decrease by.
  * Scale-In policy: Always balanced across availability zones. Options for highest instance id, newest or oldest VM.

You can also use schedule-based rules for autoscaling.

## Configure Virtual Machine Extensions

Azure Virtual Machine Extensions are small applications that provide pos-deployment configuration and automation tasks.

You can create custom scripts which can be downloaded from GitHub, Azure Storage or uploaded to the Azure portal. Custom Script Extensions times out in 90 minutes.

### Desired State Configuration

Desired State Configuration is a management platform in Windows PowerShell. Desired State Configuration enables deploying and managing configuration data for software services and managing the environment in which these services run. The platform also provides a means to maintain and manage existing configurations.

Used when custom script extensions don't satisfy your requirements.

## Configure Azure App Service Plans

An App Service plan defines a set of compute resources for a web application. One or more application can be configured to run in the same App Service plan. You can configure autoscaling for a plan.

An App Service plan defines three settings:

* Region
* Number of VM instances
* Size of VM instances

App Service Plan tiers:

* **Free/Shared tier**:
  * Apps run by receiving CPU minutes on a shared VM instance and can't scale out.
* **Basic, Standard, Premium or Isolated tiers**:
  * Apps run on all VM instances configured.
  * Multiple apps in the same plan share the same VMs and scale out together.

Using a single plan for multiple apps can save money as you pay for the resources allocated on the plan, but capacity must be managed.

* **Free and Shared**: Runs on shared VMs (with other customers). No SLA provided, no auto scale, single instance.
* **Basic**: Up to 3 dedicated instances with integrated load balancer. No auto scale.
* **Standard**: Up to 10 dedicated instances with integrated load balancer with auto scale options.
* **Premium**: Up to 30 faster dedicated VMs with integrated load balancer and auto scale.
* **Isolated**: Runs in a private virtual network, same VM type as Premium, scaling up to 100 VMs.

You can auto scale using metric-based rules (CPU %, average response time, requests) our time-based rules.

## Configure Azure App Service

Azure App Service brings together everything needed to create web apps, mobile backends and web APIs.

* **Multiple languages and frameworks supported**: ASP.NET (Windows only), Java, Ruby (Linux only), Node.js, PHP and Python. Windows and Linux custom containers (you can push your code or a docker container).
* **OS**: Your app can run on Windows or Linux (some runtime stacks only support one type of OS).
* **DevOps optimization**: Azure DevOps, GitHub, BitBucket, DockerHub and Azure Container Registry.
* **Connections to SaaS platforms and op-prem**: More than 50 connectors for enterprise systems (such as SAP), SaaS services, internet services (such as Facebook).
* **Security and compliance**: App Service is ISO, SOC and PCI compliant. You can authenticate users with AAD or social services (Google, Facebook...).
* **Visual Studio integration**: Dedicated tools in Visual Studio.
* **API and mobile features**: Provides turn-key CORS support for RESTful API scenarios and simplifies mobile app scenarios by enabling authentication, offline data sync, push notifications and more.
* **Serverless code**: Lets you run serverless code snippet or scrip on-demand.

The App Service must be associated with an App Service Plan on creation time.

Post-creation extra settings:

* **Always On**: You can keep your app loaded even when there's no traffic. This setting is required for continuous WebJobs or for WebJobs that are triggered by using a CRON expression.
* **ARR affinity**: In a multi-instance deployment, you can ensure your app client is routed to the same instance for the life of the session.
* **Connection strings**: Connection strings for your app are encrypted at rest and transmitted over an encrypted channel.

When you create a web app with App Service you can choose between manual or automated (continuous integration).

Continuous integration (automated deployment) deploys new features when you push code into production branch of your code repository (GitHub, Azure DevOps or Gitbucket). This feature uses auto-sync.

You can also manually deploy by adding a git URL to push code, use Azure CLI command `webapp up`, Visual Studio deployment wizard or FTP/FTPS.

When you deploy to Azure App Service, you can use a separate deployment slot instead of the default production slot. Slots are live apps that have their own hostname, only available in Standard, Premium and Isolated tiers (each tier offers a different number of slots).

You can deploy new code to a staging slot before swapping it with production. When swapping, your staging will go to production and production to staging, meaning that if anything goes wrong you can swap again to restore previous version. You can auto swap Windows based web apps using Azure DevOps (it'll swap staging a prod after new code is ready to run on staging).

Some of the slot settings are swappable and other are slot-specific, meaning they'll stay the same after a swap.

| Swapped settings                                                                                                                                                                                                                                                                                                                                                              | Slot-specific settings                                                                                                                                                                                                                                                                                                         |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p>General settings, such as framework version, 32/64-bit, web sockets<br>App settings <strong>*</strong><br>Connection strings <strong>*</strong><br>Handler mappings<br>Public certificates<br>WebJobs content<br>Hybrid connections <strong>**</strong><br>Service endpoints <strong>**</strong><br>Azure Content Delivery Network <strong>**</strong><br>Path mapping</p> | <p>Custom domain names<br>Non-public certificates and TLS/SSL settings<br>Scale settings<br>Always On<br>IP restrictions<br>WebJobs schedulers<br>Diagnostic settings<br>Cross-origin resource sharing (CORS)<br>Virtual network integration<br>Managed identities<br>Settings that end with the suffix _EXTENSION_VERSION</p> |

Azure App Service provides built-in authentication and authorization support, meaning you can sign in users without or with minimum code writing.

The security module runs in the same environment of your application code, but separately. Is configured by using app settings and when enabled, every incoming HTTP request passes through the module before it's handled to the app itself.

The module is able to:

* Authenticate users with the specified provier.
* Validate, store and refresh tokens.
* Manage the authenticated session.
* Inject identity information into request headers.

Module options:

* **Allow anonymous requests (no action)**: Defer authorization of unauthenticated traffic to your application code. This provides more flexibility for handling anonymous requests.
* **Allow only authenticated requests**: Redirects all anonymous request to authorization page. With this feature you don't need to change any code, but also restricts anonymous requests to any page.
* **Logging and tracing**: View authentication and authorization traces directly in your log files.

You can configure custom domains in paid tiers of App Service plan if you don't want to use the default `<app_name>.azurewebsites.net` domain. For that you need to:

* Reserve your domain (you can buy a domain name directly from Azure portal).
* Create DNS records to map the domain to your web app. You can create an A (address) record, which maps an IP to a domain name, or a CNAME (canonical name) record, which maps a domain name to another domain name.

You can configure and schedule backups for your app within Azure App Service options with a Standard or Premium tier plan. For that you need an Azure storage account and container in the same subscription as the app.

Azure App Service can backup to the storage account your app configuration settings, file content and any database connected to your app. The backup consists of a zip file and a XML file with a manifest of the zip file. Partial backups and restores are supported. They can hold up to 10 GB of app and database content.

If the storage account is enabled with a firewall, you can't use the storage account as the backup destination.

You can get insights of your App Service using Application Insights, a feature of Azure Monitor. It can be configured within your App Service (not supported for all frameworks).

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

With Applications Insights you can track:

* Request rates.
* Dependency rates.
* Failure rates.
* Exceptions.
* Page views.
* Load performance.
* User and session counts.
* Performance counters.
* Host diagnostics.
* Trace logs.
* Custom events and metrics.



