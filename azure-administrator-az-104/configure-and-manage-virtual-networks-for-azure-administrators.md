# Configure and Manage Virtual Networks for Azure Administrators

## Plan Virtual Networks

An Azure virtual network is a logical isolation of the Azure cloud that's dedicated to your subscription. A VNET can't span between subscriptions and regions.

You control the DNS server settings for virtual networks, and segmentation of the virtual network into subnets.

OSI layers reference:

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Default outbound security rules:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

For VMs, which can have a subnet associated NSG and a NIC associated NSG, NSG rules are applied in order. The first NSG hit (subnet - inbound / NIC - outbound) may block rules that are allowed on the next.

* For inbound traffic, Azure first processes network security group security rules for any associated subnets and then any associated network interfaces.
* For outbound traffic, the process is reversed. Azure first evaluates network security group security rules for any associated network interfaces followed by any associated subnets.
* For both the inbound and outbound evaluation process, Azure also checks how to apply the rules for intra-subnet traffic.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### Application Security Groups

Application security groups work in the same way as network security groups, but they provide an application-centric way of looking at your infrastructure. You join your virtual machines to an application security group. Then you use the application security group as a source or destination in the network security group rules.

When you control network traffic by using application security groups, you don't need to configure inbound and outbound traffic for specific IP addresses.

By organizing your virtual machines into application security groups, you don't need to also distribute your servers across specific subnets. You can arrange your servers by application and purpose to achieve logical groupings.

## Configure Azure Firewall

Azure Firewall is a managed, cloud-based network security service that protects your Azure Virtual Network resources. It's a fully stateful firewall as a service with built-in high availability and unrestricted cloud scalability. You can centrally create, enforce, and log application and network connectivity policies across subscriptions and virtual networks.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

When you deploy a firewall, the recommended approach is to implement a **hub-spoke** network topology.

The **hub** is a virtual network in Azure that acts as a central point of connectivity to your on-premises network.

**Spokes** are virtual networks that peer with the hub, and can be used to isolate workloads.

Traffic flows between an on-premises datacenter and the hub network through an Azure connection, such as Azure ExpressRoute, Azure VPN Gateway, or Azure Bastion.

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

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

