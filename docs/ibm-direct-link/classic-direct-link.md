---
layout: default
title: IBM Cloud Classic Direct Link Overview
parent: IBM Cloud Direct Link
nav_order: 1
---

# Classic Direct Link Overview
IBM Cloud Direct Link on Classic offerings provide connectivity from an external source into a customer's IBM Cloud private network. Direct Link on Classic can be viewed as an alternative to a traditional site-to-site VPN solution, which is designed for customers that need more consistent, higher-throughput connectivity between a remote network and their IBM Cloud environments.

![DL Decision Tree](https://dsc.cloud/quickshare/dl-decision-tree.png)

## Exchange 
Direct Link Exchange on Classic allows customers to use an Exchange provider to deliver connectivity to their IBM Cloud. An Exchange provider is a colocation or data center provider that is already connected to the IBM Cloud network, by using multi-tenant, high capacity links (also known as a network-to-network interface, or NNI). Customers typically can purchase a virtual circuit at this provider, bringing connectivity at a reduced cost, because the physical connectivity from IBM Cloud to the Exchange provider is in place already, shared among other customers.

## Connect
Direct Link Connect on Classic allows customers to use a connection through our Carrier partners who own and operate a facility-based network. A Connect provider is a network service provider (NSP) that is already connected to the IBM Cloud network, by using multi-tenant, high capacity links (also known as a network-to-network interface, or NNI). Customers typically can purchase a virtual circuit at this provider, bringing connectivity at a reduced cost, because the physical connectivity from IBM Cloud to the Connect provider is in place already, shared among other customers.

## Dedicated
Direct Link Dedicated on Classic allows customers to terminate a single-tenant, fiber-based cross-connect into the IBM Cloud network. This offering can be used by customers with colocation premises that are next to IBM Cloud PoPs and data centers; as well as network service providers that deliver circuits to customer premises or other data centers.

## Dedicatated Hosting
Direct Link Dedicated Hosting on Classic provides connectivity similar to Direct Link Dedicated, but the connection point is next to an IBM Cloud data center, which improves latency for higher-performance use cases. IBM Cloud offers various customizable colocation services with this solution.

# Find Locations and Providers via CLI
From the customer portal, launch [Cloud Shell](https://cloud.ibm.com/shell) and run the following commands to pull provider and location information:

*Get All Available Locations*  
```shell
$ ibmcloud sl call-api Network_DirectLink_Location getAllObjects
```