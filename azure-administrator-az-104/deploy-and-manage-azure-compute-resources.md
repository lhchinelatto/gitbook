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

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

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

## Configure Azure Container Instances

The top-level resource in Azure Container Instances is the container group. Containers in a container group shares host machine, lifecycle, resources, local network, and storage volumes. Like a pod in Kubernetes.

A container group can be deployed by ARM templates or YAML files.

## Configure Azure Kubernetes Service

Azure hosts a kubernetes service and performs critical functions like health monitoring and maintenance.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Kubernetes concepts:

* **Pools**: Group of nodes with identical configuration.
* **Nodes**: VM host.
* **Pods**: Single instance of an application. Can contain multiple containers.
* **Container**: Image with code (software) and it's dependencies.
* **Deployment**: A deployment has one or more identical pods managed by kubernetes.
* **Manifest**: YAML file that describes a deployment.

AKS is divided in two major components:

* **Azure-managed nodes**: Hosts that provides core Kubernetes services and orchestration (controller). These nodes are provided as managed Azure resources and are abstracted from the user.
* **Customer-managed nodes**: Hosts running application workloads (worker). Also known as agent nodes.

More stuff about AKS:

* The initial number of nodes is defined at AKS creation time. The default node pool contains the VMs running your agent nodes.
* The kubelet is the Kubernetes agent that processes the orchestration requests from the Azure-managed node, and scheduling of running the requested containers.
* The kube-proxy component handles virtual networking on each node. The proxy routes network traffic and manages IP addressing for services and pods.
* AKS clusters with Kubernetes with version >= 1.19 uses containerd runtime and < 1.19 uses Moby (upstream Docker).
* Pods typically have a 1:1 mapping with a container, although there are advanced scenarios where a pod might contain multiple containers.
* Multi-container pods are scheduled together on the same node, and allow containers to share related resources.
* When you create a pod, you can define resource limits to request a certain amount of CPU or memory resources. The Kubernetes Scheduler attempts to schedule the pods to run on a node with available resources to meet the request.
* You can specify maximum resource limits that prevent a given pod from consuming too much compute resource from the underlying node (best practice to include limits to all pods to help scheduler)
* A pod is a logical resource, but a container is where the application workloads run.

### Kubernetes Service Networking

* Kubernetes nodes are connected to a virtual network, which provides inbound and outbound connectivity for pods.
* The kube-proxy component runs on each node to provide the network features.
* Network policies configure security and filtering of the network traffic for pods.
* Network traffic can be distributed by using a load balancer.
* Complex routing of application traffic can be achieved with Ingress Controllers.

AKS helps simplify kubernetes networking. When creating a kubernetes load balancer, an Azure Load Balancer resource is created and configured. When opening network ports to pods, the corresponding Azure network security group (NSG) are configured. For HTTP application routing, Azure can configure an external DNS as new ingress routes are configured.

There are four service types for configuring network:

| Service type     | Description                                                                                                                                  | Scenario                                                                                                |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Cluster IP**   | Create an internal IP address for use within an Azure Kubernetes Service cluster.                                                            | _Implement internal-only applications that support other workloads within the cluster_                  |
| **NodePort**     | Create a port mapping on the underlying node.                                                                                                | _Allow direct access to the application with the node IP address and port_                              |
| **LoadBalancer** | Create an Azure Load Balancer resource, configure an external IP address, and connect the requested pods to the load balancer back-end pool. | _Allow customer traffic to reach the application by creating load-balancing rules on the desired ports_ |
| **ExternalName** | Create a specific DNS entry.                                                                                                                 | _Support easier application access_                                                                     |

* You can create external and internal load balancers. Internal load balancers are only assigned private IP address and can't be accessed from the internet.
* Both internal and external static IP address can be assigned.

### Configure Azure Kubernetes Service Storage

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* Traditional storage volumes that store and retrieve data are created as Kubernetes resources backed by Azure Storage.
* You can manually create storage volumes to be assigned to pods directly, or have Kubernetes automatically create them.
* Storage volumes can use Azure Disks or Azure Files:
  * Use **Azure Disks** to create a Kubernetes _DataDisk_ resource. Disks can use Azure Premium storage, backed by high-performance SSDs, or Azure Standard storage, backed by regular HDDs. For most production and development workloads, use Premium storage. Azure Disks are mounted with _ReadWriteOnce_ permissions, so they're available to a single node only. For storage volumes that can be accessed by multiple nodes simultaneously, use Azure Files.
  * Use **Azure Files** to mount an SMB 3.0 share backed by an Azure storage account to pods. Azure Files let you share data across multiple nodes and pods. Files can use Azure Standard storage backed by regular HDDs, or Azure Premium storage, backed by high-performance SSDs.

Volumes are defined and created as part of the pod lifecycle and exist only until the pod is deleted. Pods often expect their storage to remain if a pod is rescheduled on a different host during a maintenance event, especially in `StatefulSets` configurations. A persistent volume (`PersistentVolume`) is a storage resource that's created and managed by the Kubernetes API that can exist beyond the lifetime of an individual pod.

* You can use Azure Disks or Azure Files to provide a persistent volume. The choice of whether to use Azure Disks or Azure Files is often determined by the need for concurrent access to the data or the performance tier.
* A persistent volume can be statically created by a cluster administrator, or dynamically created by the Kubernetes API server.
* If a pod is scheduled, and requests Storage that's not currently available, Kubernetes can create the underlying Azure Disks or Azure Files storage. Kubernetes also attaches the storage volume to the pod.
* Dynamic provisioning uses a `StorageClass` type to identify what kind of Azure Storage needs to be created.

To define different tiers of storage, such as Premium and Standard, you can configure a `StorageClass` type. The `StorageClass` type also defines the `reclaimPolicy` actions for the storage. The `reclaimPolicy` definition controls the behavior of the underlying Azure Storage resource when the pod is deleted and the persistent volume might no longer be required. The underlying Storage resource can be deleted, or retained for use with a future pod.

In Azure Kubernetes Service, four initial `StorageClasses` types are created for a cluster by using in-tree storage plugins:

| StorageClass type   | Description                                                      | reclaimPolicy action                                                                                                     |
| ------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `default`           | Use Azure StandardSSD storage to create an Azure managed disk.   | Ensures the underlying Azure disk is deleted when the persistent volume that used the disk is deleted.                   |
| `managed-premium`   | Use Azure Premium storage to create an Azure managed disk.       | Ensures the underlying Azure disk is deleted when the persistent volume that used the disk is deleted.                   |
| `azurefile`         | Use Azure Standard storage to create an Azures Files file share. | Ensures the underlying Azure Files file share is deleted when the persistent volume that used the file share is deleted. |
| `azurefile-premium` | Use Azure Premium storage to create an Azures Files file share.  | Ensures the underlying Azure Files file share is deleted when the persistent volume that used the file share is deleted. |

A persistent volume claim (`PersistentVolumeClaim`) requests either Azure Disks or Azure Files storage of a particular `StorageClass`, access mode, and size.

* The Kubernetes API server can dynamically provision the underlying storage resource in Azure, if there's no existing resource to fulfill the claim based on the defined `StorageClass` type.
* The pod definition includes the volume mount after the volume has been connected to the pod.
* A persistent volume is _bound_ to a persistent volume claim after an available Storage resource is assigned to the pod that requests the volume.
* There's a 1:1 mapping of persistent volumes to claims.

### Configure Azure Kubernetes Service Scaling

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

| Scaling technique                | Description                                                                                                                                                                                                                                                                                                                                                                                                                   | Version requirements                                                    |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Manually scale pods or nodes** | Manually scale your replicas (pods) and nodes to test how your application responds to changes in available resources and state. Manually scaling resources lets you define a specific number of resources to use to maintain a fixed cost, such as the number of nodes. To manually scale, you define the replica or node count, and the Kubernetes API schedules creating new pods or draining nodes.                       | All Kubernetes versions                                                 |
| **Automatically scale pods**     | Use the horizontal pod autoscaler (HPA) to monitor resource demand and automatically scale the number of your replicas. By default, the HPA checks the Metrics API every 30 seconds for any required changes in your replica count. When changes are required, the number of replicas is increased or decreased accordingly.                                                                                                  | AKS clusters that deploy the Metrics Server for Kubernetes 1.8 or later |
| **Automatically scale clusters** | Respond to changing pod demands with the cluster autoscaler, which adjusts the number of your nodes based on the requested compute resources in the node pool. By default, the cluster autoscaler checks the API server every 10 seconds for any required changes in the node count. If the cluster autoscale determines a change is required, the number of nodes in your AKS cluster is increased or decreased accordingly. | RBAC-enabled AKS clusters that run Kubernetes 1.10.x or later           |

Things to consider when using horizontal autoscaling (scaling pods):

* **Consider number of pods (replicas)**. When you configure the HPA for a given deployment, you define the minimum and maximum number of pods (replicas) that can run.
* **Consider scaling metrics**. To use the HPA, define the metric to monitor and to use as the basis for scaling decisions, such as CPU usage.
*   **Consider cooldown for scaling events**. As the HPA checks the Metrics API every 30 seconds, previous scale events might not complete before subsequent checks occur. The HPA might change the number of replicas before the previous scale event receives the application workload and resource demands to adjust accordingly.

    To minimize race events, set cooldown or delay values to define how long the HPA must wait after a scale event before another scale event is triggered. This behavior allows the new replica count to take effect and the Metrics API to reflect the distributed workload. By default, the delay on scale up events is 3 minutes, and the delay on scale down events is 5 minutes.
* **Consider tuning cooldown values**. You might need to tune cooldown values. Default cooldown values might give the impression that the HPA isn't scaling the replica count quickly enough. To more quickly increase the number of replicas in use, reduce the `--horizontal-pod-autoscaler-upscale-delay` value when you create your HPA definitions by using the Azure CLI `kubectl` tool.

Things to consider when using cluster autoscaling (scaling nodes):

* **Consider combining with HPA**. Cluster autoscaler is typically used alongside the horizontal pod autoscaler. When the two scaling techniques are combined, the HPA increases or decreases the number of pods based on application demand. The cluster autoscaler adjusts the number of nodes as needed to run the extra pods accordingly.
*   **Consider scale-out events**. If a node doesn't have sufficient compute resources to run a requested pod, that pod can't progress through the scheduling process. The pod can't start unless other compute resources are available within the node pool.

    When the cluster autoscaler notices pods that can't be scheduled due to node pool resource constraints, the number of nodes within the node pool is increased to provide the extra compute resources. When the extra nodes are successfully deployed and available for use within the node pool, the pods are then scheduled to run on them.
* **Consider burst scaling to Azure Container Instances**. If your application needs to scale rapidly, some pods might remain in a state waiting to be scheduled until the new nodes deployed by the cluster autoscaler can accept the scheduled pods. For applications that have high burst demands, you can scale with virtual nodes and Azure Container Instances. We take a closer look at rapid burst scaling in the next section.
*   **Consider scale-in events**. The cluster autoscaler monitors the pod scheduling status for nodes that haven't recently received new scheduling requests. This scenario indicates that the node pool has more compute resources than required, so the number of nodes can be decreased.

    A node that passes a threshold for not being needed for 10 minutes is scheduled for deletion by default. When this situation occurs, pods are scheduled to run on other nodes within the node pool, and the cluster autoscaler decreases the number of nodes.
* **Consider avoiding single pods**. Your applications might experience some disruption as pods are scheduled on different nodes when the cluster autoscaler decreases the number of nodes. To minimize disruption, avoid applications that use a single pod instance.

### Configure AKS Burst Scaling to Azure Container Instances

Kubernetes has built-in components to scale your replicas (pods) and nodes. If your application needs to scale rapidly, the horizontal pod autoscaler might schedule more pods than can be provided by the existing compute resources in the node pool. As a result, the cluster autoscaler is triggered to deploy more nodes in the node pool. This scenario can take a few minutes for the nodes to successfully provision.

To resolve this situation, you can rapidly scale your Azure Kubernetes Service cluster by integrating with Azure Container Instances.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* Azure Container Instances lets you quickly deploy your container instance without extra infrastructure overhead. When you connect with AKS, your container instance becomes a secured, logical extension of your AKS cluster.
* The Virtual Kubelet component is installed in your AKS cluster. The component presents your container instance as a virtual Kubernetes node.
* Kubernetes schedules pods to run as container instances through virtual nodes, rather than pods on virtual machine nodes directly in your AKS cluster.
* Your application requires no modification to use virtual nodes.
* Deployments can scale across AKS and Container Instances. There's no delay when the cluster autoscaler deploys new nodes in your AKS cluster.
* Virtual nodes are deployed to another subnet in the same virtual network as your AKS cluster. This virtual network configuration allows the traffic between Container Instances and AKS to be secured. Like an AKS cluster, a container instance is a secure, logical compute resource that's isolated from other users.

## Manage Virtual Machines with the Azure CLI

{% code fullWidth="false" %}
```bash
az vm create \
  --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
  --location westus \
  --name SampleVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --verbose
az vm image list --output table
az vm image list --publisher Microsoft --output table --all
az vm image list --location eastus --output table
az vm list-sizes --location eastus --output table
az vm create \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM2 \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --verbose \
    --size "Standard_DS2_v2"
az vm list-vm-resize-options \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM \
    --output table
az vm resize \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM \
    --size Standard_D2s_v3
az vm list --output table
az vm list-ip-addresses -n SampleVM -o table
az vm show --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb --name SampleVM
az vm show \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM \
    --query "osProfile.adminUsername"
az vm show \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM \
    --query "networkProfile.networkInterfaces[].id"
az vm stop \
    --name SampleVM \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb
az vm get-instance-view \
    --name SampleVM \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" -o tsv
az vm start \
    --name SampleVM \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb
az vm open-port \
    --port 80 \
    --resource-group learn-0655786f-bf0e-412f-9c79-8f6eb0fd6efb \
    --name SampleVM

```
{% endcode %}

## Create a Windows Virtual Machine in Azure

Exercise module

## Host a Web Application with Azure App Service

Exercise module

## Protect your Virtual Machine Settings with Azure Automation State Configuration

Azure Automation State Configuration is an Azure service built on PowerShell. It enables you to consistently deploy, reliably monitor, and automatically update the desired state of all your resources.

Azure Automation State Configuration uses Desired State Configuration (DSC) artifacts to manage Linux and Windows VMs, in the cloud or on-premises.&#x20;

It has a built-in pull server. You can target nodes to automatically receive configurations from this pull server, conform to the desired state, and report back on their compliance.

You can use Azure Monitor logs to review the compliance of your nodes by configuring Azure Automation State Configuration to send this data.

A declarative programming language separates intent (what you want to do) from execution (how you want to do it).

Instead of writing scripts to manage configuration state, you use declarative DSC PowerShell language.

```powershell
Configuration Create_Share
{
   Import-DscResource -Module xSmbShare
   # A node describes the VM to be configured

   Node $NodeName
   {
      # A node definition contains one or more resource blocks
      # A resource block describes the resource to be configured on the node
      xSmbShare MySMBShare
      {
          Ensure      = "Present"
          Name        = "MyFileShare"
          Path        = "C:\Shared"
          ReadAccess  = "User1"
          FullAccess  = "User2"
          Description = "This is an updated description for this share"
      }
   }
}
```

The local configuration manager (LCM) is the component of Windows Management Framework (WMF) responsible for updating the state of a node to match desired state.

The LCM can operate in two modes:

* **Push mode**: The admin manually pushes configuration to nodes, and the LCM will make sure each node matches the desired state.
* **Pull mode**: A pull server holds the configuration information, and the LCM on each node polls the pull server at intervals (default 15min) retrieving latest configuration details. Pull mode can help you achieve a state of continuous compliance for security and regulatory standards.

Azure Automation DSC supports a series of Windows OS versions and Linux distributions using the DSC Linux extension.

If your nodes are located in a private network, DSC needs the following ports and URLs to communicate with Azure Automation:

* **Port**: Only TCP 443 is required for outbound internet access.
* **Global URL**: \*.azure-automation.net
* **Global URL of US Gov Virginia**: \*.azure-automation.us
* **Agent service**: https://`<workspaceId>`.agentsvc.azure-automation.net

To list available PowerShell DSC resources, use cmdlet:

```powershell
Get-DscResource | select Name,Module,Properties
```

Some of the built-in DSC resources:

| Resource               | Description                                           |
| ---------------------- | ----------------------------------------------------- |
| File                   | Manages files and folders on a node                   |
| Archive                | Decompresses an archive in the .zip format            |
| Environment            | Manages system environment variables                  |
| Log                    | Writes a message in the DSC event log                 |
| Package                | Installs or removes a package                         |
| Registry               | Manages a node's registry key (except HKEY Users)     |
| Script                 | Executes PowerShell commands on a node                |
| Service                | Manages Windows services                              |
| User                   | Manages local users on a node                         |
| WindowsFeature         | Adds or removes a role or feature on a node           |
| WindowsOptionalFeature | Adds or removes an optional role or feature on a node |
| WindowsProcess         | Manages a Windows process                             |

Anatomy of a DSC code block:

```powershell
Configuration MyDscConfiguration {              ##1
param
(
    [string] $ComputerName='localhost'
)
    Node $ComputerName {                          ##2
        WindowsFeature MyFeatureInstance {      ##3
            Ensure = 'Present'
            Name = 'Web-Server'
        }
    }
}
MyDscConfiguration -OutputPath C:\temp\         ##4
```

1. **Configuration**: The configuration block is the outermost script block. It starts with the `Configuration` keyword, and you provide a name. Here, the name of the configuration is `MyDscConfiguration`.\
   The configuration block describes the desired configuration. Think of a configuration block like a function, except that it contains a description of the resources to install rather than the code to install them.\
   Like a PowerShell function, a configuration block can take parameters. For example, you could parameterize the node name.
2.  **Node**: You can have one or more node blocks. The node block determines the names of .mof files that are generated when you compile the configuration. For example, the node name `localhost` generates only one _localhost.mof_ file. But you can send that .mof file to any server. You generate multiple .mof files when you use multiple node names.

    Use the array notation in the node block to target multiple hosts. For example:

    ```powershell
    Node @('WEBSERVER1', 'WEBSERVER2', 'WEBSERVER3')
    ```
3. **Resource**: One or more resource blocks can specify the resources to configure. In this case, a single resource block references the `WindowsFeature` resource. The `WindowsFeature` resource here ensures that the Windows feature `Web-Server` is installed.
4.  **MyDscConfiguration**: This call invokes the `MyDscConfiguration` block. It's like running a function. When you run a configuration block, it's compiled into a Managed Object Format (MOF) document. MOF is a compiled language created by Desktop Management Task Force, and it's based on interface definition language.

    For every node listed in the DSC script, a .mof file is created in the folder you specified with the `-OutputPath` parameter.

In a configuration data block, you can provide data that the configuration process might need. You apply this data to named nodes, or you apply it globally across all nodes.

A configuration data block is a named block that contains an array of nodes. The array must be named `AllNodes`. Inside the `AllNodes` array, you specify the data for a node by using the `NodeName` variable.

Using the previous scenario, let's say that on the web server that's installed on each node, you want to set the `SiteName` property to different values. You could define a configuration data block like this:

```powershell
$datablock =
@{
    AllNodes =
    @(
        @{
            NodeName = "WEBSERVER1"
            SiteName = "WEBSERVER1-Site"
        },
        @{
            NodeName = "WEBSERVER2"
            SiteName = "WEBSERVER2-Site"
        },
        @{
            NodeName = "WEBSERVER3"
            SiteName = "WEBSERVER3-Site"
        }
    );
}
```

If you want to set a property to the same value in each node, in the `AllNodes` array, specify `NodeName = "*"`.

A DSC script might require credential information for the configuration process. Avoid putting a credential in plaintext in your source code management tool. Instead, DSC configurations in Azure Automation can reference credentials stored in a `PSCredential` object. You define a parameter for the DSC script by using the `PSCredential` type. Before running the script, get the credentials for the user, use the credentials to create a new `PSCredential` object, and pass this object into the script as a parameter.

Credentials aren't encrypted in .mof files by default. They're exposed as plaintext. To encrypt credentials, use a certificate in your configuration data. The certificate's private key needs to be on the node on which you want to apply the configuration. Certificates are configured through the node's LCM.

Starting in PowerShell 5.1, .mof files on the node are encrypted at rest. In transit, all credentials are encrypted through WinRM.

After you create a compiled .mof file for a configuration, you can push it to a node by running the `Start-DscConfiguration` cmdlet. If you add the path to the directory, it applies any .mof file it finds in that directory to the node:

```powershell
Start-DscConfiguration -path D:\
```

Pull mode:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
