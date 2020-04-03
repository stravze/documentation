---
title: How to deploy a Kubernetes cluster on Azure Stack Hub using AKS engine
description: Deploy a Kubernetes cluster using AKS engine
services: azure-stack
author: Daniel Brennand
reviewer: Daniel Brennand
lastreviewed: 

toc_rootlink: Users
toc_sub1: How To
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Deploy a Kubernetes cluster using AKS engine
toc_fullpath: Users/How To/azs-how-create-aksengine.md
toc_mdlink: azs-how-create-aksengine.md
---

# How to deploy a Kubernetes cluster on Azure Stack Hub using AKS engine

## Overview

The AKS engine command-line utility allows Azure Stack Hub tenants to deploy and manage a Kubernetes cluster.
AKS engine can be used to create, upgrade, scale and maintain Azure Resource Manager native clusters running on VMs and other infrastructure-as-a-service (IaaS) resources on Azure Stack Hub.

[Kubernetes on Azure Stack Hub in GA](https://azure.microsoft.com/en-gb/updates/kubernetes-on-azure-stack-in-ga/)

## Useful links

This article is based on Microsoft's deploy documentation for Kubernetes on [Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1910).

* [What is the AKS engine on Azure Stack Hub?](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1910)

* [Set up the prerequisites for the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-set-up?view=azs-1910)

* [Install the AKS engine on Linux in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-1910)

* [Install the AKS engine on Windows in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-1910)

* [Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster?view=azs-1910)

### Intended audience

To complete the steps in this guide, you must have appropriate access to a subscription in Azure Stack Hub.

## Prerequisites

### Operator prerequisites

Ask your Azure Stack Hub operator if the following prerequisites below have been met:

* Azure Stack Hub version >= **1910**

* Linux custom script extension version >= **2.0.6**

* AKS base Ubuntu image >= **2019.10.24**

    - **Image name:** AKS Base Ubuntu 16.04-LTS Image

    - **Offer:** aks

    - **Publisher:** microsoft-aks

    - **SKU:** aks-ubuntu-1604-201910

### Tenant prerequisites

* A Service principal name (SPN) with **contributor** role assigned to it.

    > [!NOTE]
    > This is used by AKS engine to deploy the Kubernetes cluster on Azure Stack Hub.

* Have appropriate access to a subscription in Azure Stack Hub.

* Ability to generate public and private SSH key pairs using:

    - OpenSSH

    - `ssh-keygen`

* Ability to create a Secure Shell (SSH) to a remote machine using:

    - An SSH client

## Deploying the Kubernetes cluster using AKS engine

UKCloud has created a script ([DeployAzSHKubernetes.ps1](linkhere)) and module ([AKSEngine.psm1](linkhere)) to automate the deployment process of the Kubernetes cluster on Azure Stack Hub.

## [Linux](#tab/tabid-1)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$ArmEndpoint    | The Azure Resource Manager endpoint for Azure Stack Hub.                 | <form oninput="result.value=armendpoint.value" id="armendpoint" style="display: inline;"><input type="text" id="armendpoint" name="armendpoint" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$AzureStackUsername        | The username used to login to the tenant domain to create a new SPN.                           | <form oninput="result.value=azurestackusername.value" id="azurestackusername" style="display: inline;"><input type="text" id="azurestackusername" name="azurestackusername" style="display: inline;" placeholder="admin@contoso.onmicrosoft.com"/></form> |
| \$AzureStackUserPassword        | The password used to login to the tenant domain to create a new SPN.                          | <form oninput="result.value=azurestackuserpassword.value" id="azurestackuserpassword" style="display: inline;"><input type="text" id="azurestackuserpassword" name="azurestackuserpassword" style="display: inline;" placeholder="Password123!"/></form> |
| \$TenantDomain    | The tenant domain to login to.                                 | <form oninput="result.value=tenantdomain.value" id="tenantdomain" style="display: inline;"><input type="text" id="tenantdomain" name="tenantdomain" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl.value" id="tenantportalurl" style="display: inline;"><input type="text" id="tenantportalurl" name="tenantportalurl" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with.                                | <form oninput="result.value=masternodecount.value" id="masternodecount" style="display: inline;"><input type="text" id="masternodecount" name="masternodecount" style="display: inline;" placeholder="4"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount.value" id="workernodecount" style="display: inline;"><input type="text" id="workernodecount" name="workernodecount" style="display: inline;" placeholder="4"/></form> |

1. Clone the Github repo using the following command:

    ```bash
    git clone x && cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

    <pre><code class="language-PowerShell">
    .\DeployAzSHKubernetes.ps1 -AzureStackUsername "<output form="azurestackusername" name="result" style="display: inline;">admin@contoso.onmicrosoft.com</output>" -AzureStackUserPassword "<output form="azurestackuserpassword" name="result" style="display: inline;">Password123!</output>" -TenantDomain "<output form="tenantdomain" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -ArmEndpoint "<output form="armendpoint" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>" -MasterNodeCount <output form="masternodecount" name="result" style="display: inline;">4</output> -WorkerNodeCount <output form="workernodecount" name="result" style="display: inline;">4</output> -Verbose
    </code></pre>

## [Windows](#tab/tabid-2)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$ArmEndpoint    | The Azure Resource Manager endpoint for Azure Stack Hub.                 | <form oninput="result.value=armendpoint1.value" id="armendpoint1" style="display: inline;"><input type="text" id="armendpoint1" name="armendpoint1" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$AzureStackUsername        | The username used to login to the tenant domain to create a new SPN.                           | <form oninput="result.value=azurestackusername1.value" id="azurestackusername1" style="display: inline;"><input type="text" id="azurestackusername1" name="azurestackusername1" style="display: inline;" placeholder="admin@contoso.onmicrosoft.com"/></form> |
| \$AzureStackUserPassword        | The password used to login to the tenant domain to create a new SPN.                          | <form oninput="result.value=azurestackuserpassword1.value" id="azurestackuserpassword1" style="display: inline;"><input type="text" id="azurestackuserpassword1" name="azurestackuserpassword1" style="display: inline;" placeholder="Password123!"/></form> |
| \$TenantDomain    | The tenant domain to login to.                                 | <form oninput="result.value=tenantdomain1.value" id="tenantdomain1" style="display: inline;"><input type="text" id="tenantdomain1" name="tenantdomain1" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl1.value" id="tenantportalurl1" style="display: inline;"><input type="text" id="tenantportalurl1" name="tenantportalurl1" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| \$WindowsVMPassword     | The password used when deploying the Windows client VM. | <form oninput="result.value=windowsvmpassword.value" id="windowsvmpassword" style="display: inline;"><input type="text" id="windowsvmpassword" name="windowsvmpassword" style="display: inline;" placeholder="Password123!"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with.                                | <form oninput="result.value=masternodecount1.value" id="masternodecount1" style="display: inline;"><input type="text" id="masternodecount1" name="masternodecount1" style="display: inline;" placeholder="4"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount1.value" id="workernodecount1" style="display: inline;"><input type="text" id="workernodecount1" name="workernodecount1" style="display: inline;" placeholder="4"/></form> |

1. Clone the Github repo using the following command:

    ```bash
    git clone x && cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

    <pre><code class="language-PowerShell">
    .\DeployAzSHKubernetes.ps1 -AzureStackUsername "<output form="azurestackusername1" name="result" style="display: inline;">admin@contoso.onmicrosoft.com</output>" -AzureStackUserPassword "<output form="azurestackuserpassword1" name="result" style="display: inline;">Password123!</output>" -TenantDomain "<output form="tenantdomain1" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -ArmEndpoint "<output form="armendpoint1" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl1" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>" -Windows -WindowsVMPassword "<output form="windowsvmpassword" name="result" style="display: inline;">Password123!</output>" -MasterNodeCount <output form="masternodecount1" name="result" style="display: inline;">4</output> -WorkerNodeCount <output form="workernodecount1" name="result" style="display: inline;">4</output> -Verbose
    </code></pre>

***
