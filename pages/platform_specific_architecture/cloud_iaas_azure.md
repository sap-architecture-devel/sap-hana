# Platform Specific Architecture for Azure (Microsoft Azure)

Description

- [Platform Specific Architecture for Azure (Microsoft Azure)](#platform-specific-architecture-for-azure-microsoft-azure)
  - [Azure: Overall Architecture](#azure-overall-architecture)
  - [Azure: Basic Architecture](#azure-basic-architecture)
    - [Azure: Storage Configurations](#azure-storage-configurations)
  - [Azure: Virtual Hostname/IP](#azure-virtual-hostnameip)
    - [Generic implementation steps](#generic-implementation-steps)
    - [Additional comments](#additional-comments)
  - [Azure: High Availability](#azure-high-availability)
  - [Azure: Disaster Recovery](#azure-disaster-recovery)
  - [Azure: Data Tiering Options](#azure-data-tiering-options)
  - [Azure: XSA](#azure-xsa)

## Azure: Overall Architecture

![Azure: Overall Architecture](../../images/arch-azure-overall.png)

- some general text
  - some basic links to Azure reference architectures and documentation

## Azure: Basic Architecture

Link to generic content: [Module: Basic Architecture](pages/generic_architecture/module_basic_architecture.md#module-basic-architecture)

- supported instance types
- description of single node implementation (storage) + picture
- description of scale-out implementations (storage) + picture
- mention that subnets are stretched across AZs
- links to Azure documentation

### Azure: Storage Configurations

- visualization of storage for Azure

## Azure: Virtual Hostname/IP

Link to generic content: [Module: Virtual Hostname/IP](pages/generic_architecture/module_virtual_hostname.md#module-virtual-hostnameip)

This chapter describes recommended implementation of Virtual Hostname and Virtual IP for an SAP HANA installation on the Azure cloud.

The implementation is based on assigning a _Secondary static private IP address_ to an existing network interface of the Azure Virtual Machine (VM). A _Secondary static private IP address_ can be reassigned to another VM and so it can follow SAP HANA instance relocation. For more details see [Azure: Assign multiple IP addresses](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal). This _Secondary static private IP address_ is associated with the Virtual Hostname which is used during SAP HANA instance installation.

### Generic implementation steps 

- define Virtual IP (in the same subnet as the network interface) and Virtual Hostname for SAP HANA Instance
- assign _Virtual IP_ as _Secondary static private IP address_ to existing network interface (see [Azure: Add IP addresses to a VM](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#add)
- configure OS to use _Virtual IP Address_ 
  - e.g. [Azure: Add IP addresses to a VM operating system](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#os-config)
  - e.g. [SUSE: Administration Guide - Configuring Multiple Addresses](https://documentation.suse.com/sles/12-SP4/single-html/SLES-admin/index.html#sec-basicnet-yast-configure-addresses))
- make Virtual Hostname resolvable (e.g. updating `/etc/hosts`)
- install SAP HANA instance with the _Virtual Hostname_ (see [SAP: Administration Guide - Default Host Names and Virtual Host Names](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/aa7e697ccf214852a283a75126c34370.html))

**Note:** to reassign the _Virtual IP_ to another VM you need first to remove it on the existing VM and after that to assign it on the new VM (see [Azure: Remove IP addresses](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface-addresses#remove-ip-addresses))

### Additional comments

**SAP HANA inbound network communication**

A network communication initiated from a remote system to the Virtual IP of SAP HANA instance is established and takes place between remote system IP and the Virtual IP (_Secondary static private IP address_). 
The _Primary private IP address_ on the same interface is not involved in this communication.

**SAP HANA outbound network communication**

In contrast to an inbound connection, when SAP HANA instance initiates a network connection to a remote system the _Primary private IP address_ is used as source IP instead of Virtual IP (_Secondary static private IP address_).  
If there is requirement to use Virtual IP as the source IP, it could be achieved by means of linux routing. The core of the idea is to add route specifying source address like `ip route add <network/mask> via <gateway> src <virtual_ip>` (see [Routing for multiple uplinks/providers](https://www.tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.rpdb.multiple-links.html#AEN258)).

## Azure: High Availability

Link to generic content: [Module: High Availability](pages/generic_architecture/module_high_availability.md#module-high-availability)

- link to list of Availability Zones in Azure
- comment that it is important to measure AZ latency via niping (I will add this as new section in general part)
- fencing mechanism (options, recommendation)
- how to implement cluster IP (as load balancer)
  - explain why we need load balancer (no ARP invalidations)
  - it is managed by cluster (explain how - but opening port on active node)
  - link to [Video](https://youtu.be/axyPUGS7Wu4) and [PDF](https://www.suse.com/media/presentation/TUT1134_Microsoft_Azure_and_SUSE_HAE%20_When_Availability_Matters.pdf)
- links to Azure/SUSE/RHEL documentation
- how to modify cluster to have active/active
- how to modify cluster to have tenant specific cluster IPs
- anything else?

## Azure: Disaster Recovery

Link to generic content: [Module: Disaster Recovery](pages/generic_architecture/module_disaster_recovery.md#module-disaster-recovery)

- anything to consider? bandwidth?

## Azure: Data Tiering Options

Link to generic content: [Module: Data Tiering Options](pages/generic_architecture/module_data_tiering.md#module-data-tiering-options)

- what is supported what is not (matrix)
- links to Azure documentation
- modified pictures of storage setup (if required)

## Azure: XSA

Link to generic content: [Module: SAP XSA](pages/generic_architecture/module_xsa.md#module-sap-xsa)

- I think there is nothing infrastructure specific
