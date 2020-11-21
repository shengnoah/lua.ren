---
layout: post
title: System Center Virtual Machine Manager 2012 RC&amp;ndash; Evaluation开放下载 
tags: [lua文章]
categories: [topic]
---
System Center Virtual Machine Manager 2012 RC– Evaluation开放下载

  

System Center Virtual Machine Manager 2012 已经推出2012的RC版了~离上是应该不远了，相关新功能如下

  * **Fabric Management**
    * Setup Upgrade 
      * *New in RC -Upgrade- Setup will support the following upgrade paths: 
        * VMM 2008 R2 SP1 --SC VMM 2012 RC -- SC VMM 2012 RTM 
        * SC VMM 2012 RC -- SC VMM 2012 RTM
    * Hyper-V and Cluster Lifecycle Management – Deploy Hyper-V to bare metal server, create Hyper-V clusters, orchestrate patching of a Hyper-V Cluster 
      * *New in RC: 
        * ISO or CD-based OSD for environments with DHCP without WDS 
        * OSD will now convert dynamic to fixed type of VHD destination 
        * All network adapters on host can be configured during provisioning
      * *New in RC: 
        * Ability to bypass cluster validation during cluster creation 
        * Run cluster validation reports on-demand 
        * New Cluster status tab to view an aggregated status and a cluster validation report 
        * Ability to see current CSV owner in the properties of the cluster
    * Third Party Virtualization Platforms - Add and Manage Citrix XenServer and VMware ESX Hosts and Clusters 
    * Network Management – Manage IP Address Pools, MAC Address Pools and Load Balancers 
      * *New in RC: 
        * Simplification of the logical networks in the Fabric workspace 
        * Ability to see IP addresses that are in use from a IP pool 
        * Added support for Microsoft Network Load Balancer 
        * Gateway and DNS are no longer mandatory fields for logical networks 
        * Load balancer can now support affinity to logical networks
    * Storage Management – Classify Storage, Manage Storage Pools and LUNs 
      * *New in RC 
        * Create persistent sessions to iSCSI array and logon initiator to array 
        * Better scalability of storage operations - LUN create, snapshot, clone, masking, and unmasking 
        * Option to create storage groups per cluster (BETA only supported creation of storage group per node in a cluster) 
        * Enablement of MPIO feature when provisioning a new Hyper-V server 
        * Automatic MPIO device claim 
        * Support for arrays that implement OnePortPerView
    * Update Management- Keep your VMM Fabric Servers (VMM roles, hosts, and clusters) up-to-date with patches. 
      * *New in RC: 
        * Share a WSUS root server between System Center Configuration Manager 2007 R2/ System Center Configuration Manager 2012 Beta 
        * Hyper-V Cluster Orchestration- Nodes put into VMM Maintenance Mode can be set to trigger Maintenance Mode in Operations Manager.
    * Resource Optimization 
      * Dynamic Optimization – proactively balance the load of VMs across a cluster 
      * Power Optimization – schedule power savings to use the right number of hosts to run your workloads – power the rest off until they are needed. 
        * *New in RC: 
          * Set Operations Manager Mode for powered down hosts
      * PRO – integrate with System Center Operations Manager to respond to application-level performance monitors. 
        * *New in RC: 
          * Support added for System Center Operations Manager 2012 Beta 
          * VMM will ship two sample PRO Packs: Cluster scale out and Service scale out MPs
  * **Cloud Management**
    * Abstract server, network and storage resources into private clouds 
    * Delegate access to private clouds with control of capacity, capabilities and user quotas 
    * Enable self-service usage for application administrator to author, deploy, manage and decommission applications in the private cloud
  * **Service Lifecycle Management**
    * Define service templates to create sets of connected virtual machines, OS images and application packages 
      * *New in RC: 
        * Service Designer and Specialization UI enhancements 
        * Added ability to use Service Template Patterns
    * Compose operating system images and applications during service deployment 
      * *New in RC: 
        * IP-based provisioning 
        * New application instance view
    * Scale out the number of virtual machines in a service 
    * Service performance and health monitoring integrated with System Center Operations Manager 
    * Decouple OS image and application updates through image-based servicing 
      * *New in RC: 
        * Streamlined ability to enable OS VHD updates to a Service Template 
        * Publish updated Service Templates in order to update Service Instances
    * Leverage powerful application virtualization technologies such as Server App-V

### System Center Virtual Machine Manager 2012 RC– Evaluation

http://www.microsoft.com/download/en/details.aspx?id=27252

### System Center Virtual Machine Manager 2012 Beta – Evaluation

http://www.microsoft.com/download/en/details.aspx?id=609

###

### System Center Virtual Machine Manager 2012：VMM 获得重大升级

http://technet.microsoft.com/zh-tw/magazine/hh300651.aspx

![](https://img.dazhuanlan.com/2019/11/25/5ddba7c7ce7ca.gif)  
Jerry_IT 周伯恒 2010 ~2016 Microsoft® MVP Award  
博客：http://www.dotblogs.com.tw/jerry710822  
MVP ID: 4027163  
My MVP Profile  
https://mvp.support.microsoft.com/profile/Jerry