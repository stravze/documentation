---
title: How to configure virtual network peering between two virtual networks using the UKCloud Azure Stack Hub portal
description: Configure virtual network peering between two virtual networks and allowing them to ping eachother
services: azure-stack
author: Kade Green
reviewer: 
lastreviewed: 

toc_rootlink: Users
toc_sub1: How To
toc_sub2: 
toc_sub3:
toc_sub4:
toc_title: Configure virtual network peering between two virtual networks - Portal
toc_fullpath: Users/How To/azs-how-configure-vnet-peering.md
toc_mdlink: azs-how-configure-vnet-peering.md
---

# How to configure virtual network peering between two virtual networks using the UKCloud Azure Stack Hub portal

## Overview

Overview 

### Intended audience

Audience

## Process Overview

Bellow is a shot overview of the proces. To complete the steps in this guide, you must have appropriate access to a subscription in the Azure Stack Hub portal.

1. [Create two virtual networks in Azure Stack Hub](#Create-two-virtual-networks-in-Azure-Stack-Hub)
2. [Create a virtual machine in each Virtual Network](#Create-a-Virtual-Machine-in-each-Virtual-Network)
3. [Configure virtual network peering](#Configure-virtual-network-peering)
4. 

## Create two virtual networks in Azure Stack Hub

First, you'll need to create two virtual networks. These virtual networks will be able to communicate with eachother. 

> [!NOTE]
> This is a not btw :)

1. Log in to the [Azure Stack Hub portal](https://portal.frn00006.azure.ukcloud.com).

2. In the favourites panel, select **Create a resource**.

   ![Create a resource option in favourites panel](images/azsp_newmenu.png)

3. In the **New** blade, select **Networking**.

   ![Networking option in New blade](images/azs-browser-new-networking.png)

4. In the **Featured** section, select **Virtual network**.

   ![Create new networking resource](images/azs-browser-networking-create.png)

5. In the **Create virtual network** blade, enter the following information:

   - **Subscription** - This is your UKCloud for Microsoft Azure subscription.

   - **Resource Group** - Select an existing resource group, or create a new one by typing a name for your new resource group.
   
    - **Name** - The name of the virtual network.

   - **Location** - This will be `frn00006`, which is the location of the Azure Stack Hub.

     ![Create new virtual network](images/azs-portal-vnet-config-1.png)

    - **Address Space** - The virtual network's address range in CIDR notation.

   - **Subnet** - The subnet's address range in CIDR notation (for example, 10.10.0.0). It must be contained by the address space of the virtual network. The address range of a subnet which is in use can't be edited.

   - **Subnet Name** - The name of the first subnet within the virtual network.

     ![Create new virtual network](images/azs-portal-vnet-config-2.png)

> [!NOTE]
> When creating the second virtual network, make sure to use a different address space than the first, otherwise they will not be able to communicate 

6. Click **Create**.

7. After your virtual network has deployed, you can view it by clicking **All services** in the favourites panel, then selecting **Virtual networks** under the networking section, orit appears inside the resource group.

8. Select your virtual network from the list.

9. In the blade for your virtual network, you can view and monitor the virtual network, change its settings and perform diagnostics and troubleshooting.

10. Repeat the steps above for the second virtual network, but use different address spaces.

![Create new virtual network](images/azs-portal-vnet-overview.png)

## Create a Virtual Machine in each Virtual Network

After creating each virtual network, create a virtual machine in each virtual network.

For more detailed instructions on how to create a virtal machine, see the [*How to create a virtual machine using the UKCloud Azure Stack Hub portal*](azs-how-create-vm-portal.md).

## Configure virtual network peering

1. Click on one of the created virtual netowrks and click **Add**

![Configure virtual netork peering](images/azs-portal-peering-1.png)

2. In the Add peering, enter the following information:

    - **Name**
    - **Peer Details**
    - **Subscription**
    - **Virtual Network**
    - **Name** of peering rule on  other virtual
    - **Allow virtual network access**
    - **Allow virtual network access**
    - **Allow forwarded traffic**
    - **Allow forwarded traffic**
    - **Configure gateway traffic**

3. Select the destinaiton vnet and  name that peering something good

4. Click **OK**

### Testing connection

To test the connection you need to allow your pc through the nsg to use rdp and access the vm

1. Go into each nsg, click on imbound security rules and click add

2. Fill out the perameters 

    - **source** ip address
    - **source port ranges** * (any)
    - **destination** set to your public IP
    - **destination port** set to 3389 (rdp)
    - **protocl** any
    - **action** allow
    - **priority** bump it up abit
    - **name** name the rule
    - **descripton** Describe the rule

3. Click **Add**

>[!NOTE]
>repeat for each nsg

4. then go to connect, rdp, then download rdp file and run it

5. log in with creds you created for the VM

6. go into the firewall settings and allow "file and printer sharing (echo request - icmpv4-in)"

7. ping the other network :) 