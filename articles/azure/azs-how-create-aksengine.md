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

The [AKS engine command-line utility](https://github.com/Azure/aks-engine/tree/master/) allows Azure Stack Hub tenants to deploy and manage a Kubernetes cluster.
AKS engine can be used to create, upgrade, scale and maintain Azure Resource Manager native clusters running on VMs and other infrastructure-as-a-service (IaaS) resources on Azure Stack Hub.

[Kubernetes on Azure Stack Hub in GA](https://azure.microsoft.com/en-gb/updates/kubernetes-on-azure-stack-in-ga/)

## Useful links

This article is based on Microsoft's deploy documentation for Kubernetes on [Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2005).

* [What is the AKS engine on Azure Stack Hub?](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2005)

* [Set up the prerequisites for the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-set-up?view=azs-2005)

* [Install the AKS engine on Linux in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2005)

* [Install the AKS engine on Windows in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2005)

* [Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster?view=azs-2005)

> [!CAUTION]
> Before proceeding, ensure you are aware of the [issues and limitations](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) of Kubernetes on Azure Stack Hub.

### Intended audience

To complete the steps in this article, you must have appropriate access to a subscription in Azure Stack Hub.

## Prerequisites

### Operator prerequisites

Ask your Azure Stack Hub operator if the following prerequisites below have been met:

* Azure Stack Hub version >= **2005**

* Linux custom script extension version >= **2.0.6**

* AKS base Ubuntu image >= **2020.05.13**

    - **Image name:** AKS Base Ubuntu 16.04-LTS Image Distro

    - **Offer:** aks

    - **Publisher:** microsoft-aks

    - **SKU:** aks-ubuntu-1604-202005

### Tenant prerequisites

* Have appropriate access to a subscription in Azure Stack Hub.

* Ability to create a service principal name (SPN) with **contributor** role assigned to it.

    > [!NOTE]
    > This is used by AKS engine to deploy the Kubernetes cluster on Azure Stack Hub.

* PowerShell 5.1

* [Git CLI](https://git-scm.com/)

* OpenSSH

    > [!NOTE]
    > This is usually located at `C:\Windows\System32\OpenSSH`

    Be able to use the following commands from OpenSSH:

    - `ssh-agent`

    - `ssh-keygen`

* [`AzureStack` PowerShell module](https://docs.ukcloud.com/articles/azure/azs-how-configure-powershell-users.html#install-azure-stack-hub-powershell)

* `AzureRM.OperationalInsights` PowerShell module:

    ```powershell
    Install-Module -Name AzureRM.OperationalInsights -RequiredVersion 5.0.6 -Force -Verbose
    ```

* `AzureAD` PowerShell module:

    ```powershell
    Install-Module -Name AzureAD -Force -Verbose
    ```

* `Posh-SSH >= 2.3.0` PowerShell module:

    ```powershell
    Install-Module -Name Posh-SSH -MinimumVersion 2.3.0 -Force -Verbose -AllowPreRelease
    ```

## Deployment

UKCloud has created a ([DeployAshKubernetes.ps1](linkhere)) deployment script and module ([AksEngine.psm1](linkhere)) to automate the deployment process of a Kubernetes cluster on Azure Stack Hub.

The script performs the following high-level steps:

1. Creates a Service principal name (SPN) in your Azure Stack Hub tenancy with a `contributor` role.

    - This is required to deploy the Kubernetes cluster.

2. Check that the prerequisite PowerShell modules are installed on your machine. If they are not present, attempt to install them.

3. Deploy the AKS engine client VM (either Linux or Windows) using the `New-AksEngineClient` function from [AksEngine.psm1](linkhere).

4. If the switch `-EnableMonitoring` is provided, connect to public Azure, obtain details about the Log Analytics workspace and enable the **ContainerInsights** solution using an ARM template.

    > [!NOTE]
    > If the Log Analytics workspace cannot be found, the script will attempt to create one for you.

5. Download the Kubernetes API profile JSON file, populate appropriately and generate an SSH private key (`linuxprofile_rsa`) for the Kubernetes cluster nodes.

6. Upload the Kubernetes API profile JSON file and SSH private key to the AKS engine client VM.

7. Deploy the Kubernetes cluster on Azure Stack Hub using the provided deployment API profile.

8. Configure the kubectl environment variable on the AKS engine client VM.

9. Configure the Network Security Group (NSG) SSH rule on the Kubernetes cluster to allow IPs from the `-AllowedIPAddresses` parameter.

10. If the switch `-EnableDashboard` is provided, use Helm to deploy the [Kubernetes dashboard](https://github.com/kubernetes/dashboard/tree/master/aio/deploy/helm-chart/kubernetes-dashboard).

### AKS engine client VM

The AKS engine client VM can be deployed as either Linux (Ubuntu 18.04 LTS) or Windows (Server 2019 with containers).

The client VM comes with **Kubectl** and **Helm** configured out of the box ready for you to interact with the Kubernetes cluster.

For Linux, **SSH server** is used to interact with the client VM and run automation tasks against it. Whereas, for Windows; **WinRM** is used.
For additional security, the following Network Security Group (NSG) rules are configured:

* Linux client VM: **SSH** access (port 22) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`

* Windows client VM: **WinRM** access (ports 5985 and 5986) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`

    > [!IMPORTANT]
    > For **WinRM** to connect successfully, **TrustedHosts** is configured on the local machine (with the Windows client VM's IP added) and on the Windows client VM (with the value(s) from `$AllowedIpAddresses`).
    >
    > For **WinRM** to be configured successfully, you **must** have appropriate permissions to edit **TrustedHosts** on your local machine. If you are using **Group Policy**; this may not work.
    >
    > For more information, see [WinRM configuration and remote management](https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management).

* Kubernetes cluster: **SSH** access (port 22) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`. Furthermore, the AKS engine client VM's IP is also permitted.

> [!NOTE]
> For the Windows AKS engine client VM deployment, we spent a lot of time troubleshooting issues with OpenSSH server.
> We experienced the following issues:
>
> * We could not achieve SSH agent forwarding from the client VM -> Kubernetes master node ->  Kubernetes worker node
>
> * We managed to achieve the first hop (from the client VM to the Kubernetes master node) however, we couldn't achieve the final hop to the Kubernetes worker node(s).
>
> * We made sure that each ssh-agent configuration used `ForwardAgent yes` and the SSH server configuration with `AllowAgentForwarding yes` set.
>
> * We tested with PowerShell 5 and 7.
>
> * We experienced that with an active RDP session, everything began working as expected.

### Enabling cluster monitoring

You can choose to enable monitoring of your Kubernetes cluster on Azure Stack Hub. There are two methods provided by Microsoft.

> [!NOTE]
> In order to enable monitoring for your Kubernetes cluster on Azure Stack Hub, you will need an active public Azure subscription.

#### Prerequisite step

Both methods require the prerequisite of using an [Azure Resource Manager (ARM) template](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) to enable the **ContainerInsights** solution for a **Log Analytics Workspace**.

#### [Method One](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-azure-monitor?view=azs-2005#method-one)

> [!IMPORTANT]
> This method is **not** used by the [DeployAshKubernetes.ps1](linkhere) deployment script.
> This method should be used if you decide to enable monitoring for an already deployed Kubernetes cluster on Azure Stack Hub which doesn't have monitoring enabled.

1. Use the following PowerShell script to enable the **ContainerInsights** solution for a **Log Analytics Workspace**:

    Enter details below to provide values for the variables in the following script in this article:

    | Variable name   | Variable description                                               | Input            |
    |-----------------|--------------------------------------------------------------------|------------------|
    | \$LogAnalyticsWorkspaceName    | The name of the Log Analytics Workspace in public Azure.                 | <form oninput="result.value=loganalyticsworkspacenamemonitoring.value" id="loganalyticsworkspacenamemonitoring" style="display: inline;"><input type="text" id="loganalyticsworkspacenamemonitoring" name="loganalyticsworkspacenamemonitoring" style="display: inline;" placeholder="kubernetes-cluster-workspace"/></form> |
    | \$LogAnalyticsWorkspaceResourceGroupName    | The name of the resource group containing the Log Analytics Workspace in public Azure.                 | <form oninput="result.value=loganalyticsworkspaceresourcegroupnamemonitoring.value" id="loganalyticsworkspaceresourcegroupnamemonitoring" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupnamemonitoring" name="loganalyticsworkspaceresourcegroupnamemonitoring" style="display: inline;" placeholder="kubernetes-cluster-workspace-rg"/></form> |
    | \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics Workspace.                 | <form oninput="result.value=loganalyticsworkspacelocationmonitoring.value" id="loganalyticsworkspacelocationmonitoring" style="display: inline;"><input type="text" id="loganalyticsworkspacelocationmonitoring" name="loganalyticsworkspacelocationmonitoring" style="display: inline;" placeholder="UK South"/></form> |
    | \$DownloadDirectoryPath    | The local path used to download the ARM template files to                  | <form oninput="result.value=downloaddirectorypath.value" id="downloaddirectorypath" style="display: inline;"><input type="text" id="downloaddirectorypath" name="downloaddirectorypath" style="display: inline;" placeholder="C:\Temp\EnableMonitoring"/></form> |

    <pre><code class="language-PowerShell"># Declare variables
    $LogAnalyticsWorkspaceName = "<output form="loganalyticsworkspacenamemonitoring" name="result" style="display: inline;">kubernetes-cluster-workspace</output>"
    $LogAnalyticsWorkspaceResourceGroupName = "<output form="loganalyticsworkspaceresourcegroupnamemonitoring" name="result" style="display: inline;">kubernetes-cluster-workspace-RG</output>"
    $LogAnalyticsWorkspaceLocation = "<output form="loganalyticsworkspacelocationmonitoring" name="result" style="display: inline;">UK South</output>"
    $DownloadDirectoryPath = "<output form="downloaddirectorypath" name="result" style="display: inline;">C:\Temp\EnableMonitoring</output>"
    $MonitoringARMTemplateUri = "https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature_prod/docs/templates/azuremonitor-containerSolution.json"
    $MonitoringARMTemplateParamsUri = "https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature_prod/docs/templates/azuremonitor-containerSolutionParams.json"

    # Connect to public Azure
    Connect-AzureRmAccount

    # Check if the Log Analytics resource group exists, If it doesn't exist then create it
    try {
        $LogAnalyticsWorkspaceResourceGroup = Get-AzureRmResourceGroup -Name $LogAnalyticsWorkspaceResourceGroupName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
    }
    catch {
        Write-Warning -Message "Log Analytics workspace resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"" doesn't exist. Creating now..."
        try {
            $LogAnalyticsWorkspaceResourceGroup = New-AzureRmResourceGroup -Name $LogAnalyticsWorkspaceResourceGroupName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
        }
        catch {
            throw "Failed to create Log Analytics workspace resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"":`n$($_.Exception.Message)"
        }
    }
    # Check if the Log Analytics workspace exists, If it doesn't exist then create it
    try {
        Write-Verbose "Attempting to retrieve Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" in resource group ""$($LogAnalyticsWorkspaceResourceGroupName)""."
        $LogAnalyticsWorkspace = Get-AzureRmOperationalInsightsWorkspace -ResourceGroupName $LogAnalyticsWorkspaceResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspaceName -ErrorAction Stop
    }
    catch {
        try {
            Write-Warning -Message "Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" doesn't exist. Creating now..."
            $LogAnalyticsWorkspace = New-AzureRmOperationalInsightsWorkspace -ResourceGroupName $LogAnalyticsWorkspaceResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspaceName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
        }
        catch {
            throw "Failed to create Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" in resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"":`n$($_.Exception.Message)"
        }
    }

    # Download required ARM template to enable the ContainerInsights solution
    # Ensure C:\Temp\EnableMonitoring directory exists
    New-Item -ItemType Directory -Path $DownloadDirectoryPath
    # Download the required ARM template files
    Invoke-WebRequest -Uri $MonitoringARMTemplateUri -OutFile "$($DownloadDirectoryPath)\containerSolution.json"
    Invoke-WebRequest -Uri $MonitoringARMTemplateParamsUri -OutFile "$($DownloadDirectoryPath)\containerSolutionParams.json"
    # Obtain current subscription ID from public Azure
    $SubscriptionId = (Get-AzureRmSubscription).SubscriptionId
    # Set the public Azure subscription
    Set-AzureRmContext -SubscriptionId $SubscriptionId
    # Edit the ARM template parameters file
    $MonitoringARMTemplateParamsJson = Get-Content -Path "$($DownloadDirectoryPath)\containerSolutionParams.json" | ConvertFrom-Json
    $MonitoringARMTemplateParamsJson.parameters.workspaceResourceId.value = $LogAnalyticsWorkspace.ResourceId
    $MonitoringARMTemplateParamsJson.parameters.workspaceRegion.value = $LogAnalyticsWorkspace.Location
    # Set the content of the ARM template parameters file
    $MonitoringARMTemplateParamsJson | ConvertTo-Json -Depth 20 | Set-Content -Path "$($DownloadDirectoryPath)\containerSolutionParams.json"
    # Deploy the ARM template to enable ContainerInsights solution
    New-AzureRmResourceGroupDeployment -Name "OnboardCluster" -ResourceGroupName $LogAnalyticsWorkspaceResourceGroupName -TemplateFile "$($DownloadDirectoryPath)\containerSolution.json" -TemplateParameterFile "$($DownloadDirectoryPath)\containerSolutionParams.json"

    # Output the Log Analytics workspace ID and workspace key
    Write-Output -InputObject "Please make a note of these values, they are required later."
    Write-Output -InputObject "Log Analytics workspace ID ""$($LogAnalyticsWorkspace.CustomerId)""."
    $WorkspaceKey = (Get-AzureRmOperationalInsightsWorkspaceSharedKeys -ResourceGroupName $LogAnalyticsWorkspace.ResourceGroupName -Name $LogAnalyticsWorkspace.Name).PrimarySharedKey
    Write-Output -InputObject "Log Analytics workspace key ""$($WorkspaceKey)""."
    </code></pre>

2. SSH or create a WinRM session to the AKS engine client VM.

3. Add the required Helm repository: `helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/`

4. Run the following command, replacing the **workspace_id**, **workspace_key** (These were outputted in the script above) and **cluster_name** variables to install the Helm chart on the Kubernetes cluster:

    ```bash
    helm install --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<workspace_id>,omsagent.secret.key=<workspace_key>,omsagent.env.clusterName=<cluster_name> myrelease-1 incubator/azuremonitor-containers
    ```

    > [!NOTE]
    > This Helm chart deploys the required OMS agent containers into the **kube-system** namespace in the Kubernetes cluster.

    > [!TIP]
    > You can find the **cluster_name** by running the following command on the Kubernetes master node:
    > ```bash
    > azureuser@k8s-master-85734688-0:~$ kubectl cluster-info
    > Kubernetes master is running at https://kubemaster89296451.frn00006.cloudapp.azure.ukcloud.com
    > ```
    >
    > The **cluster_name** in this example is: **kubemaster89296451**

    > [!TIP]
    > You can watch the deployment using the following command: `kubectl get pods -n kube-system -w`

> [!IMPORTANT]
> It can take around five to ten minutes for the OMS agents to begin submitting data to the Log Analytics workspace.
> You can view your Kubernetes cluster monitoring statistics via the [public Azure portal](https://aka.ms/azmon-containers).

#### Method Two

> [!IMPORTANT]
> This method **is used** by the [DeployAshKubernetes.ps1](linkhere) script.
>
> This method performs the [prerequisite step](#prerequisite-step) during the deployment.

This method utilises the [container-monitoring](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/containermonitoringaddon.md) add-on to deploy **Operations Management Suite (OMS) agent** containers to monitor the Kubernetes cluster on Azure Stack Hub.

To enable this functionality, provide the `-EnableMonitoring` switch to the [DeployAshKubernetes.ps1](linkhere) deployment script as well as the following parameters:

* `-LogAnalyticsWorkspaceName`

* `-LogAnalyticsWorkspaceResourceGroupName`

* `-LogAnalyticsWorkspaceLocation`

> [!IMPORTANT]
> Once the cluster is deployed, it can take around five to ten minutes for the OMS agents to begin submitting data to the Log Analytics workspace.
> You can view your Kubernetes cluster monitoring statistics via the [public Azure portal](https://aka.ms/azmon-containers).

#### Enabling the Kubernetes cluster dashboard

Provide the `-EnableDashboard` switch to the [DeployAshKubernetes.ps1](linkhere) deployment script to deploy the [kubernetes-dashboard](https://github.com/kubernetes/dashboard/tree/master/aio/deploy/helm-chart/kubernetes-dashboard) Helm chart.

#### Deploying a specific version of Kubernetes

Provide the `-KubernetesVersion` and `-AksEngineVersion` parameters to the [DeployAshKubernetes.ps1](linkhere) deployment script to deploy a specific version of Kubernetes on Azure Stack Hub.

> [!WARNING]
> Make sure you consult the [Supported AKS Engine versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) on Azure Stack Hub.

## Deploying the AKS engine client VM and Kubernetes cluster

## [Linux](#tab/tabid-1)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$TenantId    | Azure Active Directory domain name or tenant identifier.                                 | <form oninput="result.value=tenantid.value" id="tenantid" style="display: inline;"><input type="text" id="tenantid" name="tenantid" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$Environment    | The name of the environment you want to log in to.                                 | <form oninput="result.value=environment.value" id="environment" style="display: inline;"><input type="text" id="environment" name="environment" style="display: inline;" placeholder="AzureStackUser"/></form> |
| \$ArmEndpoint    | Azure Stack Hub resource manager endpoint.                 | <form oninput="result.value=armendpoint.value" id="armendpoint" style="display: inline;"><input type="text" id="armendpoint" name="armendpoint" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl.value" id="tenantportalurl" style="display: inline;"><input type="text" id="tenantportalurl" name="tenantportalurl" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| \$ClientVmResourceGroupName | The resource group name to provision the AKS engine client VM resources within.   | <form oninput="result.value=clientvmresourcegroupname.value" id="clientvmresourcegroupname" style="display: inline;"><input type="text" id="clientvmresourcegroupname" name="clientvmresourcegroupname" style="display: inline;" placeholder="AksEngineClient-RG"/></form> |
| \$ClientVmStorageAccountName | The storage account name for the AKS engine client VM.   | <form oninput="result.value=clientvmstorageaccountname.value" id="clientvmstorageaccountname" style="display: inline;"><input type="text" id="clientvmstorageaccountname" name="clientvmstorageaccountname" style="display: inline;" placeholder="aksengineclientstor$(Get-Random -Maximum 999)"/></form> |
| \$ClientVmSubnetName | The subnet name for the AKS engine client VM.   | <form oninput="result.value=clientvmsubnetname.value" id="clientvmsubnetname" style="display: inline;"><input type="text" id="clientvmsubnetname" name="clientvmsubnetname" style="display: inline;" placeholder="AksEngineClientSubnet"/></form> |
| \$ClientVmSubnetRange | The subnet range for the AKS engine client VM.   | <form oninput="result.value=clientvmsubnetrange.value" id="clientvmsubnetrange" style="display: inline;"><input type="text" id="clientvmsubnetrange" name="clientvmsubnetrange" style="display: inline;" placeholder="192.168.1.0/24"/></form> |
| \$ClientVmVNetName | The VNet name for the AKS engine client VM.   | <form oninput="result.value=clientvmvnetname.value" id="clientvmvnetname" style="display: inline;"><input type="text" id="clientvmvnetname" name="clientvmvnetname" style="display: inline;" placeholder="AksEngineClientVNet"/></form> |
| \$ClientVmVNetRange | The VNet range for the AKS engine client VM.   | <form oninput="result.value=clientvmvnetrange.value" id="clientvmvnetrange" style="display: inline;"><input type="text" id="clientvmvnetrange" name="clientvmvnetrange" style="display: inline;" placeholder="192.168.0.0/16"/></form> |
| \$ClientVmPublicIpName | The public IP name for the AKS engine client VM.   | <form oninput="result.value=clientvmpublicipname.value" id="clientvmpublicipname" style="display: inline;"><input type="text" id="clientvmpublicipname" name="clientvmpublicipname" style="display: inline;" placeholder="AksEngineClientPublicIp"/></form> |
| \$ClientVmNsgName | The network security group name for the AKS engine client VM.   | <form oninput="result.value=clientvmnsgname.value" id="clientvmnsgname" style="display: inline;"><input type="text" id="clientvmnsgname" name="clientvmnsgname" style="display: inline;" placeholder="AksEngineClientNsg"/></form> |
| \$ClientVmNicName | The network interface card name for the AKS engine client VM.   | <form oninput="result.value=clientvmnicname.value" id="clientvmnicname" style="display: inline;"><input type="text" id="clientvmnicname" name="clientvmnicname" style="display: inline;" placeholder="AksEngineClientNic"/></form> |
| \$ClientVmComputerName | The computer name for the AKS engine client VM.   | <form oninput="result.value=clientvmcomputername.value" id="clientvmcomputername" style="display: inline;"><input type="text" id="clientvmcomputername" name="clientvmcomputername" style="display: inline;" placeholder="AksEngineClient"/></form> |
| \$ClientVmName | The virtual machine name for the AKS engine client VM.   | <form oninput="result.value=clientvmname.value" id="clientvmname" style="display: inline;"><input type="text" id="clientvmname" name="clientvmname" style="display: inline;" placeholder="AksEngineClientVm"/></form> |
| \$ClientVmUsername | The username for the AKS engine client VM.   | <form oninput="result.value=clientvmusername.value" id="clientvmusername" style="display: inline;"><input type="text" id="clientvmusername" name="clientvmusername" style="display: inline;" placeholder="AksEngineClientUser"/></form> |
| \$AllowedIpAddresses   | A string array of IPs allowed to access the Kubernetes master node and the AKS engine client VM.   | <form oninput="result.value=allowedipaddresses.value" id="allowedipaddresses" style="display: inline;"><input type="text" id="allowedipaddresses" name="allowedipaddresses" style="display: inline;" placeholder='"127.0.0.1","192.168.0.1"'/></form> |
| \$ClusterResourceGroupName   | The resource group name on Azure Stack Hub to deploy the Kubernetes cluster into.         | <form oninput="result.value=clusterresourcegroupname.value" id="clusterresourcegroupname" style="display: inline;"><input type="text" id="clusterresourcegroupname" name="clusterresourcegroupname" style="display: inline;" placeholder="Kube-RG"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=masternodecount.value" id="masternodecount" style="display: inline;"><input type="text" id="masternodecount" name="masternodecount" style="display: inline;" placeholder="3"/></form> |
| \$MasterNodeSize  | The size of master nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=masternodesize.value" id="masternodesize" style="display: inline;"><input type="text" id="masternodesize" name="masternodesize" style="display: inline;" placeholder="Standard_D3_v2"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount.value" id="workernodecount" style="display: inline;"><input type="text" id="workernodecount" name="workernodecount" style="display: inline;" placeholder="3"/></form> |
| \$MasterNodeSize  | The size of worker nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=workernodesize.value" id="workernodesize" style="display: inline;"><input type="text" id="workernodesize" name="workernodesize" style="display: inline;" placeholder="Standard_D3_v2"/></form> |
| \$KubernetesVersion       | The version of Kubernetes to deploy using AKS engine.  | <form oninput="result.value=kubernetesversion.value" id="kubernetesversion" style="display: inline;"><input type="text" id="kubernetesversion" name="kubernetesversion" style="display: inline;" placeholder="1.16.9"/></form> |
| \$AksEngineVersion       | The supported version of the AKS engine CLI to install on the AKS engine client VM.  | <form oninput="result.value=aksengineversion.value" id="aksengineversion" style="display: inline;"><input type="text" id="aksengineversion" name="aksengineversion" style="display: inline;" placeholder="0.51.0"/></form> |
| -EnableMonitoring (switch) | A switch to enable Azure Monitor for containers add-on and ContainerInsights solution for a Log Analytics workspace.       | <form onchange="result.value=enablemonitoring.value" id="enablemonitoring" style="display: inline;"><select name="enablemonitoring" id="enablemonitoring" style="display: inline;"><option value=" -EnableMonitoring">Enable</option><option value="">Disable</option></select></form> |
| \$LogAnalyticsWorkspaceName   | The name of the Log Analytics workspace. If the Log Analytics workspace doesn't exist in the public Azure subscription ($TenantId) it will be created.       | <form oninput="result.value=loganalyticsworkspacename.value" id="loganalyticsworkspacename" style="display: inline;"><input type="text" id="loganalyticsworkspacename" name="loganalyticsworkspacename" style="display: inline;" placeholder="kube-workspace"/></form> |
| \$LogAnalyticsWorkspaceResourceGroupName   | The name of the resource group where the Log Analytics workspace resides in. If the resource group doesn't exist in your public Azure subscription ($TenantId) it will be created. | <form oninput="result.value=loganalyticsworkspaceresourcegroupname.value" id="loganalyticsworkspaceresourcegroupname" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupname" name="loganalyticsworkspaceresourcegroupname" style="display: inline;" placeholder="Kube-workspace-RG"/></form> |
| \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics workspace.                 | <form oninput="result.value=loganalyticsworkspacelocation.value" id="loganalyticsworkspacelocation" style="display: inline;"><input type="text" id="loganalyticsworkspacelocation" name="loganalyticsworkspacelocation" style="display: inline;" placeholder="UK South"/></form> |
| -EnableDashboard (switch) | A switch to enable the Kubernetes cluster dashboard.       | <form onchange="result.value=enabledashboard.value" id="enabledashboard" style="display: inline;"><select name="enabledashboard" id="enabledashboard" style="display: inline;"><option value=" -EnableDashboard">Enable</option><option value="">Disable</option></select></form> |
| -EnableLocallyConfiguredKubectl (switch) | A switch to configure kubectl on the local machine.       | <form onchange="result.value=enablelocallyconfiguredkubectl.value" id="enablelocallyconfiguredkubectl" style="display: inline;"><select name="enablelocallyconfiguredkubectl" id="enablelocallyconfiguredkubectl" style="display: inline;"><option value=" -EnableLocallyConfiguredKubectl">Enable</option><option value="">Disable</option></select></form> |

1. Clone the Github repo using the following command:

    ```bash
    git clone x; cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

   <pre><code class="language-PowerShell">
    .\DeployAshKubernetes.ps1 -TenantId "<output form="tenantid" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -Environment "<output form="environment" name="result" style="display: inline;">AzureStackUser</output>" -ArmEndpoint "<output form="armendpoint" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>" -ClientVmResourceGroupName "<output form="clientvmresourcegroupname" name="result" style="display: inline;">AksEngineClient-RG</output>" -ClientVmStorageAccountName "<output form="clientvmstorageaccountname" name="result" style="display: inline;">aksengineclientstor$(Get-Random -Maximum 999)</output>" -ClientVmSubnetName "<output form="clientvmsubnetname" name="result" style="display: inline;">AksEngineClientSubnet</output>" -ClientVmSubnetRange "<output form="clientvmsubnetrange" name="result" style="display: inline;">192.168.1.0/24</output>" -ClientVmVNetName "<output form="clientvmvnetname" name="result" style="display: inline;">AksEngineClientVNet</output>" -ClientVmVNetRange "<output form="clientvmvnetrange" name="result" style="display: inline;">192.168.0.0/16</output>" -ClientVmPublicIpName "<output form="clientvmpublicipname" name="result" style="display: inline;">AksEngineClientPublicIp</output>" -ClientVmNsgName "<output form="clientvmnsgname" name="result" style="display: inline;">AksEngineClientNsg</output>" -ClientVmNicName "<output form="clientvmnicname" name="result" style="display: inline;">AksEngineClientNic</output>" -ClientVmComputerName "<output form="clientvmcomputername" name="result" style="display: inline;">AksEngineClient</output>" -ClientVmName "<output form="clientvmname" name="result" style="display: inline;">AksEngineClientVm</output>" -ClientVmUsername "<output form="clientvmusername" name="result" style="display: inline;">AksEngineClientUser</output>" -AllowedIpAddresses <output form="allowedipaddresses" name="result" style="display: inline;">"127.0.0.1","192.168.0.1"</output> -ClusterResourceGroupName "<output form="clusterresourcegroupname" name="result" style="display: inline;">Kube-RG</output>" -MasterNodeCount <output form="masternodecount" name="result" style="display: inline;">3</output> -MasterNodeSize "<output form="masternodesize" name="result" style="display: inline;">Standard_D3_v2</output>" -WorkerNodeCount <output form="workernodecount" name="result" style="display: inline;">3</output> -WorkerNodeSize "<output form="workernodesize" name="result" style="display: inline;">Standard_D3_v2</output>" -KubernetesVersion "<output form="kubernetesversion" name="result" style="display: inline;">1.16.9</output>" -AksEngineVersion "<output form="aksengineversion" name="result" style="display: inline;">0.51.0</output>"<output form="enablemonitoring" name="result" style="display: inline;"> -EnableMonitoring</output> -LogAnalyticsWorkspaceName "<output form="loganalyticsworkspacename" name="result" style="display: inline;">kube-workspace</output>" -LogAnalyticsWorkspaceResourceGroupName "<output form="loganalyticsworkspaceresourcegroupname" name="result" style="display: inline;">Kube-workspace-RG</output>" -LogAnalyticsWorkspaceLocation "<output form="loganalyticsworkspacelocation" name="result" style="display: inline;">UK South</output>"<output form="enabledashboard" name="result" style="display: inline;"> -EnableDashboard</output><output form="enablelocallyconfiguredkubectl" name="result" style="display: inline;"> -EnableLocallyConfiguredKubectl</output> -Verbose
    </code></pre>

## [Windows](#tab/tabid-2)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$TenantId    | Azure Active Directory domain name or tenant identifier.                                 | <form oninput="result.value=tenantid1.value" id="tenantid1" style="display: inline;"><input type="text" id="tenantid1" name="tenantid1" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$Environment    | The name of the environment you want to log in to.                                 | <form oninput="result.value=environment1.value" id="environment1" style="display: inline;"><input type="text" id="environment1" name="environment1" style="display: inline;" placeholder="AzureStackUser"/></form> |
| \$ArmEndpoint    | Azure Stack Hub resource manager endpoint.                 | <form oninput="result.value=armendpoint1.value" id="armendpoint1" style="display: inline;"><input type="text" id="armendpoint1" name="armendpoint1" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl1.value" id="tenantportalurl1" style="display: inline;"><input type="text" id="tenantportalurl1" name="tenantportalurl1" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| -WindowsClientVm (switch) | A switch to deploy a Windows AKS engine client VM. If not specified, a Linux AKS engine client VM is deployed. | <form onchange="result.value=windowsclientvm1.value" id="windowsclientvm1" style="display: inline;"><select name="windowsclientvm1" id="windowsclientvm1" style="display: inline;"><option value=" -WindowsClientVm">Enable</option><option value="">Disable</option></select></form> |
| \$WindowsClientVmPassword | The password for the Windows AKS engine client VM.   | <form oninput="result.value=windowsclientvmpassword1.value" id="windowsclientvmpassword1" style="display: inline;"><input type="text" id="windowsclientvmpassword1" name="windowsclientvmpassword1" style="display: inline;" placeholder="SuperSecurePassword123!"/></form> |
| \$ClientVmResourceGroupName | The resource group name to provision the AKS engine client VM resources within.   | <form oninput="result.value=clientvmresourcegroupname1.value" id="clientvmresourcegroupname1" style="display: inline;"><input type="text" id="clientvmresourcegroupname1" name="clientvmresourcegroupname1" style="display: inline;" placeholder="AksEngineClient-RG"/></form> |
| \$ClientVmStorageAccountName | The storage account name for the AKS engine client VM.   | <form oninput="result.value=clientvmstorageaccountname1.value" id="clientvmstorageaccountname1" style="display: inline;"><input type="text" id="clientvmstorageaccountname1" name="clientvmstorageaccountname1" style="display: inline;" placeholder="aksengineclientstor$(Get-Random -Maximum 999)"/></form> |
| \$ClientVmSubnetName | The subnet name for the AKS engine client VM.   | <form oninput="result.value=clientvmsubnetname1.value" id="clientvmsubnetname1" style="display: inline;"><input type="text" id="clientvmsubnetname1" name="clientvmsubnetname1" style="display: inline;" placeholder="AksEngineClientSubnet"/></form> |
| \$ClientVmSubnetRange | The subnet range for the AKS engine client VM.   | <form oninput="result.value=clientvmsubnetrange1.value" id="clientvmsubnetrange1" style="display: inline;"><input type="text" id="clientvmsubnetrange1" name="clientvmsubnetrange1" style="display: inline;" placeholder="192.168.1.0/24"/></form> |
| \$ClientVmVNetName | The VNet name for the AKS engine client VM.   | <form oninput="result.value=clientvmvnetname1.value" id="clientvmvnetname1" style="display: inline;"><input type="text" id="clientvmvnetname1" name="clientvmvnetname1" style="display: inline;" placeholder="AksEngineClientVNet"/></form> |
| \$ClientVmVNetRange | The VNet range for the AKS engine client VM.   | <form oninput="result.value=clientvmvnetrange1.value" id="clientvmvnetrange1" style="display: inline;"><input type="text" id="clientvmvnetrange1" name="clientvmvnetrange1" style="display: inline;" placeholder="192.168.0.0/16"/></form> |
| \$ClientVmPublicIpName | The public IP name for the AKS engine client VM.   | <form oninput="result.value=clientvmpublicipname1.value" id="clientvmpublicipname1" style="display: inline;"><input type="text" id="clientvmpublicipname1" name="clientvmpublicipname1" style="display: inline;" placeholder="AksEngineClientPublicIp"/></form> |
| \$ClientVmNsgName | The network security group name for the AKS engine client VM.   | <form oninput="result.value=clientvmnsgname1.value" id="clientvmnsgname1" style="display: inline;"><input type="text" id="clientvmnsgname1" name="clientvmnsgname1" style="display: inline;" placeholder="AksEngineClientNsg"/></form> |
| \$ClientVmNicName | The network interface card name for the AKS engine client VM.   | <form oninput="result.value=clientvmnicname1.value" id="clientvmnicname1" style="display: inline;"><input type="text" id="clientvmnicname1" name="clientvmnicname1" style="display: inline;" placeholder="AksEngineClientNic"/></form> |
| \$ClientVmComputerName | The computer name for the AKS engine client VM.   | <form oninput="result.value=clientvmcomputername1.value" id="clientvmcomputername1" style="display: inline;"><input type="text" id="clientvmcomputername1" name="clientvmcomputername1" style="display: inline;" placeholder="AksEngineClient"/></form> |
| \$ClientVmName | The virtual machine name for the AKS engine client VM.   | <form oninput="result.value=clientvmname1.value" id="clientvmname1" style="display: inline;"><input type="text" id="clientvmname1" name="clientvmname1" style="display: inline;" placeholder="AksEngineClientVm"/></form> |
| \$ClientVmUsername | The username for the AKS engine client VM.   | <form oninput="result.value=clientvmusername1.value" id="clientvmusername1" style="display: inline;"><input type="text" id="clientvmusername1" name="clientvmusername1" style="display: inline;" placeholder="AksEngineClientUser"/></form> |
| \$AllowedIpAddresses   | A string array of IPs allowed to access the Kubernetes master node and the AKS engine client VM.   | <form oninput="result.value=allowedipaddresses1.value" id="allowedipaddresses1" style="display: inline;"><input type="text" id="allowedipaddresses1" name="allowedipaddresses1" style="display: inline;" placeholder='"127.0.0.1","192.168.0.1"'/></form> |
| \$ClusterResourceGroupName   | The resource group name on Azure Stack Hub to deploy the Kubernetes cluster into.         | <form oninput="result.value=clusterresourcegroupname1.value" id="clusterresourcegroupname1" style="display: inline;"><input type="text" id="clusterresourcegroupname1" name="clusterresourcegroupname1" style="display: inline;" placeholder="Kube-RG"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=masternodecount1.value" id="masternodecount1" style="display: inline;"><input type="text" id="masternodecount1" name="masternodecount1" style="display: inline;" placeholder="3"/></form> |
| \$MasterNodeSize  | The size of master nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=masternodesize1.value" id="masternodesize1" style="display: inline;"><input type="text" id="masternodesize1" name="masternodesize1" style="display: inline;" placeholder="Standard_D3_v2"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount1.value" id="workernodecount1" style="display: inline;"><input type="text" id="workernodecount1" name="workernodecount1" style="display: inline;" placeholder="3"/></form> |
| \$MasterNodeSize  | The size of worker nodes to deploy the Kubernetes cluster with. | <form oninput="result.value=workernodesize1.value" id="workernodesize1" style="display: inline;"><input type="text" id="workernodesize1" name="workernodesize1" style="display: inline;" placeholder="Standard_D3_v2"/></form> |
| \$KubernetesVersion       | The version of Kubernetes to deploy using AKS engine.  | <form oninput="result.value=kubernetesversion1.value" id="kubernetesversion1" style="display: inline;"><input type="text" id="kubernetesversion1" name="kubernetesversion1" style="display: inline;" placeholder="1.16.9"/></form> |
| \$AksEngineVersion       | The supported version of the AKS engine CLI to install on the AKS engine client VM.  | <form oninput="result.value=aksengineversion1.value" id="aksengineversion1" style="display: inline;"><input type="text" id="aksengineversion1" name="aksengineversion1" style="display: inline;" placeholder="0.51.0"/></form> |
| -EnableMonitoring (switch) | A switch to enable Azure Monitor for containers add-on and ContainerInsights solution for a Log Analytics workspace.       | <form onchange="result.value=enablemonitoring1.value" id="enablemonitoring1" style="display: inline;"><select name="enablemonitoring1" id="enablemonitoring1" style="display: inline;"><option value=" -EnableMonitoring">Enable</option><option value="">Disable</option></select></form> |
| \$LogAnalyticsWorkspaceName   | The name of the Log Analytics workspace. If the Log Analytics workspace doesn't exist in the public Azure subscription ($TenantId) it will be created.       | <form oninput="result.value=loganalyticsworkspacename1.value" id="loganalyticsworkspacename1" style="display: inline;"><input type="text" id="loganalyticsworkspacename1" name="loganalyticsworkspacename1" style="display: inline;" placeholder="kube-workspace"/></form> |
| \$LogAnalyticsWorkspaceResourceGroupName   | The name of the resource group where the Log Analytics workspace resides in. If the resource group doesn't exist in your public Azure subscription ($TenantId) it will be created. | <form oninput="result.value=loganalyticsworkspaceresourcegroupname1.value" id="loganalyticsworkspaceresourcegroupname1" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupname1" name="loganalyticsworkspaceresourcegroupname1" style="display: inline;" placeholder="Kube-workspace-RG"/></form> |
| \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics workspace.                 | <form oninput="result.value=loganalyticsworkspacelocation1.value" id="loganalyticsworkspacelocation1" style="display: inline;"><input type="text" id="loganalyticsworkspacelocation1" name="loganalyticsworkspacelocation1" style="display: inline;" placeholder="UK South"/></form> |
| -EnableDashboard (switch) | A switch to enable the Kubernetes cluster dashboard.       | <form onchange="result.value=enabledashboard1.value" id="enabledashboard1" style="display: inline;"><select name="enabledashboard1" id="enabledashboard1" style="display: inline;"><option value=" -EnableDashboard">Enable</option><option value="">Disable</option></select></form> |
| -EnableLocallyConfiguredKubectl (switch) | A switch to configure kubectl on the local machine.       | <form onchange="result.value=enablelocallyconfiguredkubectl1.value" id="enablelocallyconfiguredkubectl1" style="display: inline;"><select name="enablelocallyconfiguredkubectl1" id="enablelocallyconfiguredkubectl1" style="display: inline;"><option value=" -EnableLocallyConfiguredKubectl">Enable</option><option value="">Disable</option></select></form> |


1. Clone the Github repo using the following command:

    ```bash
    git clone x; cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

    <pre><code class="language-PowerShell">
    .\DeployAshKubernetes.ps1 -TenantId "<output form="tenantid1" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -Environment "<output form="environment1" name="result" style="display: inline;">AzureStackUser</output>" -ArmEndpoint "<output form="armendpoint1" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl1" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>"<output form="windowsclientvm1" name="result" style="display: inline;"> -WindowsClientVm</output> -WindowsClientVmPassword "<output form="windowsclientvmpassword1" name="result" style="display: inline;">SuperSecurePassword123!</output>" -ClientVmResourceGroupName "<output form="clientvmresourcegroupname1" name="result" style="display: inline;">AksEngineClient-RG</output>" -ClientVmStorageAccountName "<output form="clientvmstorageaccountname1" name="result" style="display: inline;">aksengineclientstor$(Get-Random -Maximum 999)</output>" -ClientVmSubnetName "<output form="clientvmsubnetname1" name="result" style="display: inline;">AksEngineClientSubnet</output>" -ClientVmSubnetRange "<output form="clientvmsubnetrange1" name="result" style="display: inline;">192.168.1.0/24</output>" -ClientVmVNetName "<output form="clientvmvnetname1" name="result" style="display: inline;">AksEngineClientVNet</output>" -ClientVmVNetRange "<output form="clientvmvnetrange1" name="result" style="display: inline;">192.168.0.0/16</output>" -ClientVmPublicIpName "<output form="clientvmpublicipname1" name="result" style="display: inline;">AksEngineClientPublicIp</output>" -ClientVmNsgName "<output form="clientvmnsgname1" name="result" style="display: inline;">AksEngineClientNsg</output>" -ClientVmNicName "<output form="clientvmnicname1" name="result" style="display: inline;">AksEngineClientNic</output>" -ClientVmComputerName "<output form="clientvmcomputername1" name="result" style="display: inline;">AksEngineClient</output>" -ClientVmName "<output form="clientvmname1" name="result" style="display: inline;">AksEngineClientVm</output>" -ClientVmUsername "<output form="clientvmusername1" name="result" style="display: inline;">AksEngineClientUser</output>" -AllowedIpAddresses <output form="allowedipaddresses1" name="result" style="display: inline;">"127.0.0.1","192.168.0.1"</output> -ClusterResourceGroupName "<output form="clusterresourcegroupname1" name="result" style="display: inline;">Kube-RG</output>" -MasterNodeCount <output form="masternodecount1" name="result" style="display: inline;">3</output> -MasterNodeSize "<output form="masternodesize1" name="result" style="display: inline;">Standard_D3_v2</output>" -WorkerNodeCount <output form="workernodecount1" name="result" style="display: inline;">3</output> -WorkerNodeSize "<output form="workernodesize1" name="result" style="display: inline;">Standard_D3_v2</output>" -KubernetesVersion "<output form="kubernetesversion1" name="result" style="display: inline;">1.16.9</output>" -AksEngineVersion "<output form="aksengineversion1" name="result" style="display: inline;">0.51.0</output>"<output form="enablemonitoring1" name="result" style="display: inline;"> -EnableMonitoring</output> -LogAnalyticsWorkspaceName "<output form="loganalyticsworkspacename1" name="result" style="display: inline;">kube-workspace</output>" -LogAnalyticsWorkspaceResourceGroupName "<output form="loganalyticsworkspaceresourcegroupname1" name="result" style="display: inline;">Kube-workspace-RG</output>" -LogAnalyticsWorkspaceLocation "<output form="loganalyticsworkspacelocation1" name="result" style="display: inline;">UK South</output>"<output form="enabledashboard1" name="result" style="display: inline;"> -EnableDashboard</output><output form="enablelocallyconfiguredkubectl1" name="result" style="display: inline;"> -EnableLocallyConfiguredKubectl</output> -Verbose
    </code></pre>

***

## Post Deployment

### Using SSH agent forwarding

A useful feature provided by SSH agent is agent forwarding. This allows you to forward your local SSH keys along the connection. This is useful as it means you only have to store the SSH private key on the client VM. SSH agent forwarding is utilised by using the `-A` parameter.

1. SSH or RDP to the client VM.

2. Add the **linuxprofile_rsa** SSH private key to the SSH agent using the following command: `ssh-add ~/.ssh/linuxprofile_rsa`

3. SSH to the Kubernetes master node using the following command: `ssh -A azureuser@masternodeip`

4. SSH from the Kubernetes master node to a worker node using the following command: `ssh -A azureuser@workernodeip`
