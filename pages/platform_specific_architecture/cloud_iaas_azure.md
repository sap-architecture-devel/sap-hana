# Platform Specific Architecture for Azure (Microsoft Azure)

- [Platform Specific Architecture for Azure (Microsoft Azure)](#platform-specific-architecture-for-azure-microsoft-azure)
  - [Azure: Overall Architecture](#azure-overall-architecture)
  - [Azure: Basic Architecture](#azure-basic-architecture)
    - [Azure: Supported Instance Types for SAP HANA](#azure-supported-instance-types-for-sap-hana)
    - [Azure: Storage Setup for SAP HANA Implementation](#azure-storage-setup-for-sap-hana-implementation)
      - [Azure: Storage Setup for SAP HANA Implementation - Azure Premium SSD](#azure-storage-setup-for-sap-hana-implementation---azure-premium-ssd)
      - [Azure: Storage Setup for SAP HANA Implementation - Azure Ultra disk](#azure-storage-setup-for-sap-hana-implementation---azure-ultra-disk)
      - [Azure: Storage Setup for SAP HANA Implementation - Azure NetApp Files](#azure-storage-setup-for-sap-hana-implementation---azure-netapp-files)
    - [Azure: Networking specifics for Azure Availability Zones](#azure-networking-specifics-for-azure-availability-zones)
  - [Azure: Virtual Hostname/IP](#azure-virtual-hostnameip)
    - [Azure: Generic implementation steps](#azure-generic-implementation-steps)
    - [Azure: Additional comments](#azure-additional-comments)
  - [Azure: High Availability](#azure-high-availability)
  - [Azure: Disaster Recovery](#azure-disaster-recovery)
  - [Azure: Data Tiering Options](#azure-data-tiering-options)
    - [Azure: Persistent Memory (NVRAM)](#azure-persistent-memory-nvram)
    - [Azure: SAP HANA Native Storage Extensions (NSE)](#azure-sap-hana-native-storage-extensions-nse)
    - [Azure: SAP HANA Extension Nodes](#azure-sap-hana-extension-nodes)
    - [Azure: SAP HANA Dynamic Tiering (DT)](#azure-sap-hana-dynamic-tiering-dt)
  - [Azure: XSA](#azure-xsa)

## Azure: Overall Architecture

Following diagram is illustrating complete version of Reference Architecture for SAP HANA tailored for Azure (Microsoft Azure).

For detailed explanation of individual modules please see individual sections in [Generic SAP HANA Architecture](../../README.md#table-of-content).

![Azure: Overall Architecture](../../images/arch-azure-overall.png)

You can also review official Azure Reference Architecture and other documentation:

- [Azure: SAP HANA infrastructure configurations and operations](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations)
- [Azure: SAP HANA (Large Instances) architecture](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-architecture)

## Azure: Basic Architecture

Link to generic content: [Module: Basic Architecture](../generic_architecture/module_basic_architecture.md#module-basic-architecture)

### Azure: Supported Instance Types for SAP HANA

Not every instance type is supported for productive SAP HANA usage.

Official list of SAP certified instance types is available at [SAP: The SAP HANA Hardware Directory](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). This should always be used to decide whether the particular instance type is supported for SAP HANA or not.

Azure specific list of certified instances with additional details can be found in following references:

- [Azure: SAP HANA certifications](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/sap-certifications#sap-hana-certifications)
- [Azure: Available SKUs for HANA Large Instances](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-available-skus)

### Azure: Storage Setup for SAP HANA Implementation

SAP HANA Storage Configuration for Virtual Machines is coming in following flavors:

- [Azure Premium SSD](cloud_iaas_azure.md#azure-storage-setup-for-sap-hana-implementation---azure-premium-ssd) - cheaper storage that meets SAP KPI requirements for most of the SAP HANA workloads
- [Azure Ultra disk](cloud_iaas_azure.md#azure-storage-setup-for-sap-hana-implementation---azure-ultra-disk) - high-performance storage intended for most demanding SAP HANA workloads
- [Azure NetApp Files](cloud_iaas_azure.md#azure-storage-setup-for-sap-hana-implementation---azure-netapp-files) - special option for [SAP HANA Host Auto-Failover (in single Availability Zone)](../generic_architecture/alternative_implementations.md#sap-hana-host-auto-failover-in-single-availability-zone)

Recommended disk setup for each option is described is following sections.

SAP HANA Storage Configuration for HANA Large Instances is always based on NFS storage as supported deployment options are covered in [Azure: Supported scenarios for HANA Large Instances](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-supported-scenario).

#### Azure: Storage Setup for SAP HANA Implementation - Azure Premium SSD

![Azure: Storage Architecture - Azure Premium SSD](../../images/arch-azure-storage1.png)

| Filesystem    | Name             | Device type        | Comment
|:--------------|:-----------------|:-------------------|:----------------
| /             | Root volume      | P6 or P10          |
| /hana/data    | SAP HANA data    | P20, P30 or P40    |
| /hana/log     | SAP HANA logs    | P20                |
| /hana/shared  | SAP HANA shared  | P20 or P30         | For Single-Node deployments
| /hana/shared  | SAP HANA shared  | Azure NetApp Files | For Scale-Out deployments
| /usr/sap      | SAP binaries     | P6                 |
| /backups      | SAP HANA backup  | P20, P30, P40, P50 | For Single-Node deployments
| /backups      | SAP HANA backup  | Azure NetApp Files | For Scale-Out deployments

Instance specific sizing recommendations are available at [Azure: Azure Premium SSD](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations-storage#solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines).

Note: Cost conscious storage configuration is available for use, however, it is not officially supported by SAP.

#### Azure: Storage Setup for SAP HANA Implementation - Azure Ultra disk

![Azure: Storage Architecture - Azure Ultra disk](../../images/arch-azure-storage2.png)

| Filesystem    | Name             | Device type        | Comment
|:--------------|:-----------------|:-------------------|:----------------
| /             | Root volume      | P6 or P10          |
| /hana/data    | SAP HANA data    | Ultra disk         | Alternatively can be on one volume with logs
| /hana/log     | SAP HANA logs    | Ultra disk         | Alternatively can be on one volume with data
| /hana/shared  | SAP HANA shared  | P20 or P30         | For Single-Node deployments
| /hana/shared  | SAP HANA shared  | Azure NetApp Files | For Scale-Out deployments
| /usr/sap      | SAP binaries     | P6                 |
| /backups      | SAP HANA backup  | P20, P30, P40, P50 | For Single-Node deployments
| /backups      | SAP HANA backup  | Azure NetApp Files | For Scale-Out deployments

Instance specific sizing recommendations are available at [Azure: Azure Ultra disk](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations-storage#azure-ultra-disk-storage-configuration-for-sap-hana).

Note: Please be aware about Ultra disk limitations as described in [Azure: Ultra disk - GA scope and limitations](https://docs.microsoft.com/en-gb/azure/virtual-machines/windows/disks-types#ga-scope-and-limitations).

#### Azure: Storage Setup for SAP HANA Implementation - Azure NetApp Files

![Azure: Storage Architecture - Azure NetApp Files](../../images/arch-azure-storage3.png)

| Filesystem    | Name             | Device type        | Comment
|:--------------|:-----------------|:-------------------|:----------------
| /             | Root volume      | P6 or P10          |
| /hana/data    | SAP HANA data    | Azure NetApp Files | For bigger systems one volume per mount point
| /hana/log     | SAP HANA logs    | Azure NetApp Files | For bigger systems one volume per mount point
| /hana/shared  | SAP HANA shared  | Azure NetApp Files |
| /usr/sap      | SAP binaries     | P6                 |
| /backups      | SAP HANA backup  | Azure NetApp Files |

Instance specific sizing recommendations are available at [Azure: Azure NetApp Files](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations-storage#nfs-v41-volumes-on-azure-netapp-files).

Note: Please be aware about Azure NetApp Files limitations as described in [Azure: Azure NetApp Files - Important considerations](https://docs.microsoft.com/en-gb/azure/virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse#important-considerations).

Key limitation is fact that Azure NetApp Files are not Availability Zone aware and can cause performance degradation when not deployed in close proximity (for example following High Availability takeover).

### Azure: Networking specifics for Azure Availability Zones

As visualized on the Overall Architecture diagram - in Azure both Availability Zones are using one subnet stretched across multiple Availability Zones. This is achieved by Network Virtualization, however, side effect of this approach is that classical [ARP cache](https://en.wikipedia.org/wiki/ARP_cache) invalidations are not working - this is having big impact on implementation of Cluster IP in Azure.

Impact on implementation of Cluster IP in Azure is described in [Azure: High Availability](#azure-high-availability).

## Azure: Virtual Hostname/IP

Link to generic content: [Module: Virtual Hostname/IP](../generic_architecture/module_virtual_hostname.md#module-virtual-hostnameip)

This chapter describes recommended implementation of Virtual Hostname and Virtual IP for an SAP HANA installation on the Azure cloud.

The implementation is based on assigning a _Secondary static private IP address_ to an existing network interface of the Azure Virtual Machine (VM). A _Secondary static private IP address_ can be reassigned to another VM and so it can follow SAP HANA instance relocation. For more details see [Azure: Assign multiple IP addresses](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal). This _Secondary static private IP address_ is associated with the Virtual Hostname which is used during SAP HANA instance installation.

### Azure: Generic implementation steps 

- define Virtual IP (in the same subnet as the network interface) and Virtual Hostname for SAP HANA Instance
- assign _Virtual IP_ as _Secondary static private IP address_ to existing network interface (see [Azure: Add IP addresses to a VM](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#add)
- configure OS to use _Virtual IP Address_ 
  - e.g. [Azure: Add IP addresses to a VM operating system](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-multiple-ip-addresses-portal#os-config)
  - e.g. [SUSE: Administration Guide - Configuring Multiple Addresses](https://documentation.suse.com/sles/12-SP4/single-html/SLES-admin/index.html#sec-basicnet-yast-configure-addresses))
- make Virtual Hostname resolvable (e.g. updating `/etc/hosts`)
- install SAP HANA instance with the _Virtual Hostname_ (see [SAP: Administration Guide - Default Host Names and Virtual Host Names](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/aa7e697ccf214852a283a75126c34370.html))

**Note:** to reassign the _Virtual IP_ to another VM you need first to remove it on the existing VM and after that to assign it on the new VM (see [Azure: Remove IP addresses](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface-addresses#remove-ip-addresses))

### Azure: Additional comments

**SAP HANA inbound network communication**

A network communication initiated from a remote system to the Virtual IP of SAP HANA instance is established and takes place between remote system IP and the Virtual IP (_Secondary static private IP address_). 
The _Primary private IP address_ on the same interface is not involved in this communication.

**SAP HANA outbound network communication**

In contrast to an inbound connection, when SAP HANA instance initiates a network connection to a remote system the _Primary private IP address_ is used as source IP instead of Virtual IP (_Secondary static private IP address_).  
If there is requirement to use Virtual IP as the source IP, it could be achieved by means of linux routing. The core of the idea is to add route specifying source address like `ip route add <network/mask> via <gateway> src <virtual_ip>` (see [Routing for multiple uplinks/providers](https://www.tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.rpdb.multiple-links.html#AEN258)).

## Azure: High Availability

Link to generic content: [Module: High Availability](../generic_architecture/module_high_availability.md#module-high-availability)

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

Link to generic content: [Module: Disaster Recovery](../generic_architecture/module_disaster_recovery.md#module-disaster-recovery)

Disaster Recovery for SAP HANA via SAP HANA System Replication is not infrastructure dependent.

## Azure: Data Tiering Options

Link to generic content: [Module: Data Tiering Options](../generic_architecture/module_data_tiering.md#module-data-tiering-options)

Following data tiering options are supposed on Azure for traditional Virtual Machines (VMs) and HANA Large Instances (HLI):

| Data Tiering Option                                                                       | Supported (VMs) | Supported (HLI)
|:------------------------------------------------------------------------------------------|:----------------|:----------------
| [Persistent Memory (NVRAM)](#azure-persistent-memory-nvram)                               | No              | No
| [SAP HANA Native Storage Extensions (NSE)](#azure-sap-hana-native-storage-extensions-nse) | Yes             | Yes
| [SAP HANA Extension Nodes](#azure-sap-hana-extension-nodes)                               | Yes             | Yes
| [SAP HANA Dynamic Tiering (DT)](#azure-sap-hana-dynamic-tiering-dt)                       | Yes             | No

### Azure: Persistent Memory (NVRAM)

Microsoft Azure platform does not offer any instance types having NVRAM that are supported for productive SAP HANA usage.

### Azure: SAP HANA Native Storage Extensions (NSE)

SAP HANA Native Storage Extensions only need additional disk space compared to regular SAP HANA deployments. Microsoft Azure platform does allow to provision additional disks to both SAP HANA VMs and HANA Large Instances to provide additional capacity into existing filesystems. There is no change to the design of the storage layout.

### Azure: SAP HANA Extension Nodes

SAP HANA Extension Node implementation is based on provisioning additional SAP HANA node (with increased storage capacity) to existing SAP HANA system. Result is SAP HANA Scale-Out system where one of the nodes is designated as SAP HANA Extension Node. Storage layout is same as for regular SAP HANA nodes and it is visualized above in section [Azure: Storage Setup for SAP HANA Implementation](#azure-storage-setup-for-sap-hana-implementation).

Additional Information:

- [Azure: HANA Large Instances: Use SAP HANA data tiering and extension nodes](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-data-tiering-extension-nodes)

### Azure: SAP HANA Dynamic Tiering (DT)

SAP HANA Dynamic Tiering (DT) implementation in Microsoft Azure platform is based on provisioning additional VM for Dynamic Tiering component and connecting it to VM hosting SAP HANA instance, thus forming two-node distributed setup. Storage layout is identical to SAP HANA Scale-out setup as illustrated above in section [Azure: Storage Setup for SAP HANA Implementation](#azure-storage-setup-for-sap-hana-implementation).

Additional Information:

- [Azure: SAP HANA Dynamic Tiering 2.0 for Azure virtual machines](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations#sap-hana-dynamic-tiering-20-for-azure-virtual-machines)

## Azure: XSA

Link to generic content: [Module: SAP XSA](../generic_architecture/module_xsa.md#module-sap-xsa)

SAP HANA extended application services, advanced model (XSA) component is not infrastructure dependent.
