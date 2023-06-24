# Configure and Manage Virtual Networks for Azure Administrators

## Plan Virtual Networks

An Azure virtual network is a logical isolation of the Azure cloud that's dedicated to your subscription. A VNET can't span between subscriptions and regions.

You control the DNS server settings for virtual networks, and segmentation of the virtual network into subnets.

OSI layers reference:

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### Create Subnets

Subnets provide a way for you to implement logical divisions within your virtual network. Your network can be segmented into subnets to help improve security, increase performance, and make it easier to manage.

* Each subnet contains a range of IP addresses that fall within the virtual network address space.
* The address range for a subnet must be unique within the address space for the virtual network.
* The range for one subnet can't overlap with other subnet IP address ranges in the same virtual network.
* The IP address space for a subnet must be specified by using CIDR notation.
* Smallest subnet possible is /29 which will give 3 usable IP addresses.
* Subnets are regional and span availability zones.

For each subnet, Azure reserves five IP addresses. The first four addresses and the last address are reserved.

Let's examine the reserved addresses in an IP address range of `192.168.1.0/24`.

| Reserved address                  | Reason                                                                |
| --------------------------------- | --------------------------------------------------------------------- |
| `192.168.1.0`                     | This value identifies the virtual network address.                    |
| `192.168.1.1`                     | Azure configures this address as the default gateway.                 |
| `192.168.1.2` _and_ `192.168.1.3` | Azure maps these Azure DNS IP addresses to the virtual network space. |
| `192.168.1.255`                   | This value supplies the virtual network broadcast address.            |

### Create Virtual Networks

Things to know:

* When you create a virtual network, you need to define the IP address space for the network.
* Plan to use an IP address space that's not already in use in your organization.
  * The address space for the network can be either on-premises or in the cloud, but not both.
  * <mark style="background-color:orange;">You can't redefine the IP address space for a network after it's created.</mark> If you plan your address space for cloud-only virtual networks, you might later decide to connect an on-premises site.
* To create a virtual network, you need to define at least one subnet.

### Plan IP Addressing

There are two types of addresses:

**Private IP addresses** enable communication within an Azure virtual network and your on-premises network. You create a private IP address for your resource when you use a VPN gateway or Azure ExpressRoute circuit to extend your network to Azure.

**Public IP addresses** allow your resource to communicate with the internet. You can create a public IP address to connect with Azure public-facing services.

A resource can have both types of IPs

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Things to know:

* IP addresses can be statically assigned or dynamically assigned.
* You can separate dynamically and statically assigned IP resources into different subnets.
* Static IP addresses don't change and are best for certain situations, such as:
  * DNS name resolution, where a change in the IP address requires updating host records.
  * IP address-based security models that require apps or services to have a static IP address.
  * TLS/SSL certificates linked to an IP address.
  * Firewall rules that allow or deny traffic by using IP address ranges.
  * Role-based virtual machines such as Domain Controllers and DNS servers.

A public IP address resource can be associated with virtual machine network interfaces, internet-facing load balancers, VPN gateways, and application gateways. You can associate your resource with both dynamic and static public IP addresses.

| Resource            | Public IP address association | Dynamic IP address | Static IP address |
| ------------------- | ----------------------------- | ------------------ | ----------------- |
| Virtual machine     | NIC                           | Yes                | Yes               |
| Load balancer       | Front-end configuration       | Yes                | Yes               |
| VPN gateway         | VPN gateway IP configuration  | Yes                | Yes **\***        |
| Application gateway | Front-end configuration       | Yes                | Yes **\***        |

**\*** Static IP addresses are available on certain SKUs only.

| Feature       | Basic SKU                                                                                  | Standard SKU                                         |
| ------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------- |
| IP assignment | Static or Dynamic                                                                          | Static                                               |
| Security      | Open by default                                                                            | Secure by default, closed to inbound traffic         |
| Resources     | Network interfaces, VPN gateways, Application gateways, and internet-facing load balancers | Network interfaces or public standard load balancers |
| Redundancy    | Not zone redundant                                                                         | Zone redundant by default                            |

A private IP address resource can be associated with virtual machine network interfaces, internal load balancers, and application gateways. Azure can provide an IP address (dynamic assignment) or you can assign the IP address (static assignment).

| Resource               | Private IP address association | Dynamic IP address | Static IP address |
| ---------------------- | ------------------------------ | ------------------ | ----------------- |
| Virtual machine        | NIC                            | Yes                | Yes               |
| Internal load balancer | Front-end configuration        | Yes                | Yes               |
| Application gateway    | Front-end configuration        | Yes                | Yes               |

## Configure Network Security Groups

You can limit network traffic to resources in your virtual network by using a network security group (NSG). You can assign a network security group to a subnet or a network interface, and define security rules in the group to control network traffic.

* A network security group contains a list of security rules that allow or deny inbound or outbound network traffic.
* A network security group can be associated to a subnet or a network interface.
* A network security group can be associated multiple times.
* You create a network security group and define security rules in the Azure portal.

You can assign network security groups to a subnet and create a protected screened subnet (also referred to as a demilitarized zone or _DMZ_). A DMZ acts as a buffer between resources within your virtual network and the internet.

* Use the network security group to restrict traffic flow to all machines that reside within the subnet.
* Each subnet can have a maximum of one associated network security group.

You can assign network security groups to a network interface card (NIC).

* Define network security group rules to control all traffic that flows through a NIC.
* Each network interface that exists in a subnet can have zero, or one, associated network security groups.

Things to know about NSG security rules:

* Azure creates several default security rules within each network security group, including inbound traffic and outbound traffic. Examples of default rules include `DenyAllInbound` traffic and `AllowInternetOutbound` traffic.
* Azure creates the default security rules in each network security group that you create.
* You can add more security rules to a network security group by specifying conditions for any of the following settings:
  * **Name**
  * **Priority**
  * **Port**
  * **Protocol** (Any, TCP, UDP)
  * **Source** (Any, IP addresses, Ranges, Application Security Group, Service tag)
  * **Destination** (Any, IP addresses, Ranges, Application Security Group, Virtual network)
  * **Action** (Allow or Deny)
* Each security rule is assigned a Priority value. All security rules for a network security group are processed in priority order. When a rule has a low Priority value, the rule has a higher priority or precedence in terms of order processing.
* You can't remove the default security rules.
* You can override a default security rule by creating another security rule that has a higher Priority setting for your network security group.
* Valid service tags: `VirtualNetwork`, `Internet`, `SQL`, `Storage`, `AzureLoadBalancer`, and `AzureTrafficManager`.

Default inbound security rules:

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Default outbound security rules:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

For VMs, which can have a subnet associated NSG and a NIC associated NSG, NSG rules are applied in order. The first NSG hit (subnet - inbound / NIC - outbound) may block rules that are allowed on the next.

* For inbound traffic, Azure first processes network security group security rules for any associated subnets and then any associated network interfaces.
* For outbound traffic, the process is reversed. Azure first evaluates network security group security rules for any associated network interfaces followed by any associated subnets.
* For both the inbound and outbound evaluation process, Azure also checks how to apply the rules for intra-subnet traffic.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### Application Security Groups

Application security groups work in the same way as network security groups, but they provide an application-centric way of looking at your infrastructure. You join your virtual machines to an application security group. Then you use the application security group as a source or destination in the network security group rules.

When you control network traffic by using application security groups, you don't need to configure inbound and outbound traffic for specific IP addresses.

By organizing your virtual machines into application security groups, you don't need to also distribute your servers across specific subnets. You can arrange your servers by application and purpose to achieve logical groupings.

## Configure Azure Firewall

Azure Firewall is a managed, cloud-based network security service that protects your Azure Virtual Network resources. It's a fully stateful firewall as a service with built-in high availability and unrestricted cloud scalability. You can centrally create, enforce, and log application and network connectivity policies across subscriptions and virtual networks.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

When you deploy a firewall, the recommended approach is to implement a **hub-spoke** network topology.

The **hub** is a virtual network in Azure that acts as a central point of connectivity to your on-premises network.

**Spokes** are virtual networks that peer with the hub, and can be used to isolate workloads.

Traffic flows between an on-premises datacenter and the hub network through an Azure connection, such as Azure ExpressRoute, Azure VPN Gateway, or Azure Bastion.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

* **Consider cost savings**. Implement a hub-spoke network and reduce your costs by centralizing services that can be shared by multiple workloads in a single location. Examples of services that can use shareable workloads include network virtual appliances (NVAs) and DNS servers.
* **Consider subscription limits**. Overcome subscription limits by peering your virtual networks from different subscriptions to the central hub network.
* **Consider role separation**. Use a hub-spoke network configuration to support separation of responsibilities. The hub-spoke firewall topology lets you distribute maintenance tasks between central IT (SecOps, InfraOps) and workloads (DevOps).
* **Consider shared services**. Support workloads in different environments that require shared services, but not connectivity to each other. Examples of these environments include development and testing where each requires DNS. Place shared services in the hub virtual network, and deploy each environment to a spoke to maintain isolation.
* **Consider centralized control**. Apply the hub-spoke network for enterprises that require central control over security aspects. A common example is a firewall located in the hub and workloads placed in each spoke.

By default, Azure Firewall denies all traffic through your virtual network. The purpose of the default behavior is to provide the highest level of protection against malicious or unknown access. To allow traffic for a particular resource or service, you need to define rules to control the specific traffic.

There are three kinds of rules you can configure for Azure Firewall: NAT, network, and application. The rules are defined in the Azure portal.

Azure Firewall processes the packet by evaluating it against your rules in the following order:

1. Network rules
2. Application rules (for the network and applications)

After a packet is allowed, Azure Firewall checks for NAT rules that define how to route the allowed traffic.

You can configure NAT or Azure Firewall destination network address translation (DNAT) rules to translate and filter inbound traffic to your subnets. Each rule in your NAT rule collection is used to translate your firewall public IP and port to a private IP and port. A NAT rule that routes traffic must be accompanied by a matching network rule to allow the traffic.

Scenarios where NAT rules can be helpful are publishing SSH, RDP, or non-HTTP/S applications to the internet.

The configuration settings for a NAT rule include:

* **Name**: Provide a label for the rule.
* **Protocol**: Choose the TCP or UDP protocol.
* **Source Address**: Identify the address as \* (internet), a specific internet address, or a classless inter-domain routing (CIDR) block.
* **Destination Address**: Specify the external address of the firewall for the rule to inspect.
* **Destination Ports**: Provide the TCP or UDP ports that the rule listens to on the external IP address of the firewall.
* **Translated Address**: Specify the IP address of the service (virtual machine, internal load balancer, and so on) that privately hosts or presents the service.
* **Translated Port**: Identify the port that the inbound traffic is routed to by Azure Firewall.

A network rule has the following configuration settings:

* **Name**: Provide a friendly label for the rule.
* **Protocol**: Choose the protocol for the rule, including TCP, UDP, ICMP (ping and traceroute), or Any.
* **Source Address**: Identify the address or CIDR block of the source.
* **Destination Addresses**: Specify the addresses or CIDR blocks of the destination(s).
* **Destination Ports**: Provide the destination port of the traffic.

Here are the configuration settings for an application rule:

* **Name**: Provide a friendly label for the rule.
* **Source Addresses**: Identify the IP address of the source.
* **Protocol:Port**: Specify `HTTP` or `HTTPS` and the port that the web server is listening on.
* **Target FQDNs**: Provide the domain name of the service, such as `www.contoso.com`. Wildcards (\*) can be used. An FQDN tag represents a group of FQDNs associated with well known Microsoft services. Example FQDN tags include `Windows Update`, `App Service Environment`, and `Azure Backup`.

## Configure Azure DNS

Azure DNS enables you to host your DNS domains in Azure and access name resolution for your domains by using Microsoft Azure infrastructure.

When you create an Azure subscription, Azure automatically creates an Azure Active Directory (Azure AD) domain for your subscription. The initial domain name follows the form `<Your Domain Name>` followed by `.onmicrosoft.com`. The initial domain is intended to be used until you configure your custom domain, which needs to be verified by Azure (can take hours).

Domains must be globally unique within Azure.

An Azure **DNS zone** hosts the DNS records for a domain. To begin hosting your domain in Azure DNS, you need to create a DNS zone for your domain name. Each DNS record for your domain is then created inside your DNS zone.

A DNS record set (also known as a _resource record set_) is a collection of records in a DNS zone.

*   All records in a DNS record set must have the same name and the same record type.

    Consider the following example where we have two records in a record set. All records have the same name, `www.contoso.com.`. All records have the same record type, `A`. Each record in the set has a different value. In this case, each record provides a different IP address.

    ```console
    www.contoso.com.        3600    IN    A     134.170.185.46
    www.contoso.com.        3600    IN    A     134.170.188.221
    ```
* A DNS record set can't contain two identical records.
*   A record set of type `CNAME` can contain only one record.

    A `CNAME` record (or _Canonical name record_) provides an alias of one domain name to another. This record is used to provide another name for your domain. The DNS `lookup` operation tries to find your domain by retrying the `lookup` with the other name specified in the `CNAME` record.

Azure Private DNS provides name resolution within your virtual networks and not provide name resolution on the internet.

## Configure Azure Virtual Network Peering

Azure Virtual Network peering lets you connect virtual networks in the same or different regions, so resources in both networks can communicate with each other. After the two VNets are peered, they operate as a single network (for connectivity purposes).

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

There are two types of network peering, **Global** and **Regional**.

| Benefit                         | Description                                                                                                                                                                                                                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Private network connections** | When you implement Azure Virtual Network peering, network traffic between peered virtual networks is private. Traffic between the virtual networks is kept on the Microsoft Azure backbone network. No public internet, gateways, or encryption is required in the communication between the virtual networks. |
| **Strong performance**          | Because Azure Virtual Network peering utilizes the Azure infrastructure, you gain a low-latency, high-bandwidth connection between resources in different virtual networks.                                                                                                                                    |
| **Simplified communication**    | Azure Virtual Network peering lets resources in one virtual network communicate with resources in a different virtual network, after the virtual networks are peered.                                                                                                                                          |
| **Seamless data transfer**      | You can create an Azure Virtual Network peering configuration to transfer data across Azure subscriptions, deployment models, and across Azure regions.                                                                                                                                                        |
| **No resource disruptions**     | Azure Virtual Network peering doesn't require downtime for resources in either virtual network when creating the peering, or after the peering is created.                                                                                                                                                     |

To implement virtual network peering, your Azure account must be assigned to the `Network Contributor` or `Classic Network Contributor` role (or a custom role with needed permissions).

When virtual networks are peered, you can configure Azure VPN Gateway in the peered virtual network as a _transit point_.

\


<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Things to know about how VPN Gateway is implemented with Azure Network peering:

* A virtual network can have only one VPN gateway.
* Gateway transit is supported for both regional and global virtual network peering.
* When you allow VPN gateway transit, the virtual network can communicate to resources outside the peering. In our sample illustration, the gateway subnet gateway within the hub virtual network can complete tasks such as:
  * Use a site-to-site VPN to connect to an on-premises network.
  * Use a vnet-to-vnet connection to another virtual network.
  * Use a point-to-site VPN to connect to a client.
* Gateway transit allows peered virtual networks to share the gateway and get access to resources. With this implementation, you don't need to deploy a VPN gateway in the peer virtual network.
* You can apply network security groups in a virtual network to block or allow access to other virtual networks or subnets. When you configure virtual network peering, you can choose to open or close the network security group rules between the virtual networks.

### Extend Peering with User-Defined Routes and Service Chaining

If you peer VNet A with B, and B with C, VNet A doesn't have connectivity with C.

There are ways to extend peering:

* Hub and spoke networks
* User-defined routes
* Service chaining

You can implement these mechanisms and create a multi-level hub and spoke architecture.



<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

| Mechanism                    | Description                                                                                                                                                                                                                                                                                                           |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hub and spoke network**    | When you deploy a hub-and-spoke network, the hub virtual network can host infrastructure components like a network virtual appliance (NVA) or Azure VPN gateway. All the spoke virtual networks can then peer with the hub virtual network. Traffic can flow through NVAs or VPN gateways in the hub virtual network. |
| **User-defined route (UDR)** | Virtual network peering enables the next hop in a user-defined route to be the IP address of a virtual machine in the peered virtual network, or a VPN gateway.                                                                                                                                                       |
| **Service chaining**         | Service chaining lets you define UDRs. These routes direct traffic from one virtual network to an NVA or VPN gateway.                                                                                                                                                                                                 |

## Configure Azure VPN Gateway

A VPN gateway is a specific type of virtual network gateway that's used to send encrypted traffic between your Azure virtual network and an on-premises location over the public internet. A VPN gateway can also be used to send encrypted traffic between your Azure virtual networks over the Microsoft network.

Things to know about VPN Gateways:

* When you implement a VPN gateway, the VPN service intercepts your data and applies encryption before it reaches the internet.
* The VPN service uses a secure pathway (called a _VPN tunnel_) for movement of your data between your device and the internet. The VPN tunnel is what enables your secure connection to the internet.
* A virtual network can have only one VPN gateway.
* Multiple connections can be created to the same VPN gateway.
* When you create multiple connections to the same VPN gateway, all VPN tunnels share the available gateway bandwidth.
* A VPN gateway can be deployed in Azure availability zones to gain resiliency, scalability, and higher availability to virtual network gateways. Availability zones physically and logically separate gateways within a region, while protecting your on-premises network connectivity to Azure from zone-level failures.

| Configuration                                                                | Scenarios                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>Site-to-site</strong><br>(S2S)</p>                                | <p>- Connect your on-premises datacenters to your Azure virtual networks through an IPsec/IKE (IKEv1 or IKEv2) VPN tunnel<br>- Support cross-premises and hybrid configurations<br>- Configure S2S VPN and Azure ExpressRoute for the same virtual network<br>- Configure S2S VPN as a secure failover path for ExpressRoute<br>- Use S2S VPNs to connect to sites outside your network that are connected through ExpressRoute</p>                                                                            |
| <p><strong>Point-to-site</strong><br>(P2S or User VPN)</p>                   | <p>- Connect individual devices to your Azure virtual networks<br>- Create a secure connection to your virtual network from an individual client computer<br>- Useful for remote or traveling workers who want to connect to Azure virtual networks from their current location<br>- Support a few clients that need to connect to a virtual network</p>                                                                                                                                                       |
| <p><strong>Virtual network to virtual network</strong><br>(VNet-to-VNet)</p> | <p>- Connect one virtual network to another virtual network through an IPsec/IKE VPN tunnel<br>- Build a network that integrates cross-premises connectivity with inter-virtual network connectivity by combining VNet-to-VNet communication with multi-site connection configurations<br>- Connect virtual networks in the same or different regions<br>- Connect virtual networks in the same or different subscriptions<br>- Connect virtual networks that have the same or different deployment models</p> |
| **Highly available**                                                         | <p>- Support high availability for cross-premises and VNet-to-VNet connections<br>- Implement high availability for multiple on-premises VPN devices<br>- Implement high availability for an active-active Azure VPN gateway<br>- Implement high availability for a combination of multiple on-premises VPN devices and an active-active Azure VPN gateway</p>                                                                                                                                                 |

### Create Site-to-Site Connections

Steps to create a S2S connection:

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

#### Create Gateway Subnet (step 3):

* You deploy a gateway in your virtual network by adding a gateway subnet.
* Your gateway subnet must be named _GatewaySubnet_.
* The gateway subnet contains the IP addresses that are used by your virtual network gateway resources and services.
* When you create your gateway subnet, gateway virtual machines are deployed to the gateway subnet and configured with the required VPN gateway settings.
* **Consider gateway subnet size**. Some configurations require a larger gateway subnet than others. For the recommended sizes, refer to the documentation for the configuration that you're planning to create. If possible, it's best to create a gateway subnet by using a classless inter-domain routing (CIDR) block of /28 or /27. This approach should provide enough IP addresses to accommodate future configuration requirements.
* **Consider no extra resources**. Identify your required virtual network gateway resources, including virtual machines. When you create your gateway subnet, gateway virtual machines are deployed to the gateway subnet. Never deploy other resources to the gateway subnet, such as extra virtual machines.
* **Consider network security groups**. Network security groups (NSGs) can't be used to create the gateway subnet. If you try to create your gateway subnet with NSGs, the configuration will be blocked.

#### Create VPN Gateway (step 4):

Important settings:

* **Gateway type**: Select the type of gateway to create, **VPN** (Azure VPN gateway) or **ExpressRoute** (Azure ExpressRoute).
* **VPN type**: Select the type of VPN to create, **Route-based** or **Policy-based**. The type of VPN you choose depends on the make and model of your VPN device, and the kind of VPN connection you intend to create. In the next unit, we examine the options for this parameter setting in detail.
  * Route-based VPN gateways are the most common. Typical scenarios include point-to-site, inter-virtual network, or multiple site-to-site connections. Select route-based when your virtual network coexists with an Azure ExpressRoute gateway, or if you need to use the IKEv2 protocol.
  * Policy-based VPN gateways support only the IKEv1 protocol.
*   **SKU**: Use the drop-down menu to select a gateway SKU. Review SKU options in the [Determine gateway SKU and generation](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku) unit.

    Your choice affects the number of VPN tunnels you can have and the Aggregate Throughput Benchmark. The benchmark value is based on measurements of multiple tunnels aggregated through a single gateway. The throughput isn't guaranteed due to internet traffic conditions and your application behavior.
*   **Generation**: Use the drop-down menu to select the gateway generation, **Generation1** or **Generation2**. Generation2 offers improved performance and SLA for the same price as Generation1.

    * Generation1 supports Basic and VpnGw1 SKUs, along with most other SKUs supported in Generation2.
    * Generation2 supports most SKUs available in Generation1, along with VpnGw4 and VpnGw5.

    &#x20;<mark style="background-color:orange;">You can't change generations or SKUs across generations.</mark>
* **Virtual network**: Use the drop-down menu to select an existing virtual network for the VPN gateway or select **Create virtual network** to configure a new virtual network. Keep in mind that a virtual network can't be associated with more than one gateway. The virtual network sends and receives traffic through the VPN gateway.

**VPN types**:

* **Route-based VPNs** use _routes_ in the IP forwarding or routing table to direct packets into their corresponding tunnel interfaces. The tunnel interfaces then encrypt or decrypt the packets in and out of the VPN tunnels. The policy (or traffic selector) for route-based VPNs are configured as any-to-any (or wild cards).
  * Most VPN gateway configurations require a route-based VPN.
  * Use a route-based VPN when your virtual network coexists with an Azure ExpressRoute gateway, or if you need to use the IKEv2 protocol.
*   **Policy-based VPNs** encrypt and direct packets through IPsec tunnels based on the IPsec policies. The policies are configured with the combinations of address prefixes between your on-premises network and the Azure virtual network. The policy (or traffic selector) is defined as an access list in the VPN device configuration.

    Keep in mind the following limitations about policy-based VPNs:

    * A policy-based VPN can be used on the Basic gateway SKU only. The policy-based VPN type isn't compatible with other gateway SKUs.
    * When you use a policy-based VPN, you can have only one VPN tunnel.
    * You can only use policy-based VPNs for S2S connections, and only for certain configurations.

<mark style="background-color:orange;">After a virtual network gateway is created, you can't change the VPN type.</mark>

**SKU and Generation**:

* **Tunnels**: The maximum number of site-to-site (S2S) and Net-to-VNet tunnels that can be created for the SKU.
* **Connections**: The maximum number of point-to-site (P2S) IKEv2 connections that can be created for the SKU.
* **Aggregate Throughput Benchmark**: The benchmark is based on measurements of multiple VPN tunnels aggregated through a single gateway. The Aggregate Throughput Benchmark for a VPN gateway is S2S + P2S combined. The Aggregate Throughput Benchmark isn't a guaranteed throughput due to internet traffic conditions and your application behavior.

Note: Basic SKU is considered a legacy SKU

Generation1

| SKU       | Tunnels | Connections | Benchmark |
| --------- | ------- | ----------- | --------- |
| VpnGw1/Az | Max. 30 | Max. 250    | 650 Mbps  |
| VpnGw2/Az | Max. 30 | Max. 500    | 1.0 Gbps  |
| VPNGw3/Az | Max. 30 | Max. 1000   | 1.25 Gbps |

Generation2

| SKU       | Tunnels  | Connections | Benchmark |
| --------- | -------- | ----------- | --------- |
| VpnGw2/Az | Max. 30  | Max. 500    | 1.25 Gbps |
| VPNGw3/Az | Max. 30  | Max. 1000   | 2.5 Gbps  |
| VPNGw4/Az | Max. 100 | Max. 5000   | 5.0 Gbps  |
| VPNGw5/Az | Max. 100 | Max. 10000  | 10.0 Gbps |

#### Create Local Network Gateway&#x20;





