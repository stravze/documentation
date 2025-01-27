---
title: How to access Red Hat update servers (RHUI 2)
description: Provides information about accessing Red Hat updates using Red Hat Update Infrastructure (RHUI 2)
services: shared-services
author: shighmoor
reviewer: pcantle
lastreviewed: 29/09/2020
toc_rootlink: How To
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Access Red Hat update servers (RHUI 2)
toc_fullpath: How To/shared-how-access-redhat-update-servers.md
toc_mdlink: shared-how-access-redhat-update-servers.md
---

# How to access Red Hat update servers (RHUI 2)

> [!IMPORTANT]
> As RHUI 2 is [scheduled for decommission](https://status.ukcloud.com/incidents/z5jz2wbcwf4f), you should consider updating to RHUI 3. For more information, see [*How to install and use Red hat Update Infrastructure (RHUI 3)*](shared-how-install-rhui-vm-3.md).

## Overview

UKCloud has implemented a Red Hat Update Infrastructure (RHUI) to provide automatic updates to our Red Hat customers in the Assured OFFICIAL and Elevated OFFICIAL security domains. The benefits over the existing system are the reliable availability of patch updates and Red Hat approved OS templates.

Before you attempt to establish a connection to RHUI, you need to make sure your virtual machines (VMs) can communicate with the RHUI, which exists outside of your cloud organisation. This may involve editing the NAT and firewall settings within your edge gateway to allow traffic to traverse into your virtual data centre (VDC). For how to do this, see the articles for creating [NAT](../vmware/vmw-how-create-nat-rules.md) and [firewall](../vmware/vmw-how-create-firewall-rules.md) rules.

### Intended audience

This article is intended for customers who have not yet migrated to RHUI v3. If you need information for RHUI 3, see [*How to access Red Hat update servers (RHUI 3)*](shared-how-access-redhat-update-servers-3.md).

## New Red Hat VM templates

Certain Red Hat templates within the public catalog are automatically configured to receive updates from UKCloud's RHUI. The templates are:

- **RHEL6.6** - UKCloud-RHEL6.6-x86_64-RHUI-Template

- **RHEL7.1** - UKCloud-RHEL7.1-x86_64-RHUI-Template

These templates have the following properties:

- You can find the default root password in the VM Properties Description field (you can set the password using Guest Customization)

- Red Hat Update Infrastructure enabled for standard packages

- VMware Tools / Guest Customization enabled

- 32GB disk space / 1 x vCPU / 2GB ram (default, adjust as required)

You can adjust the VM properties as required and you'll be billed as per the standard VM sizes as described in the [UKCloud Pricing Guide](https://ukcloud.com/pricing-guide). You will not be billed on additional storage until you reach the 60GiB of provisioned disk space.

## Old Red Hat VM templates

If you have already deployed a Red Hat VM using the old templates, you can configure it to access the RHUI by following this guide:

[*How to install Red Hat Update Infrastructure (RHUI) on an existing virtual machine*](shared-how-install-rhui-vm.md)

## Additional resources

High Availability (HA) and Extended Update Support (EUS) are premium Red Hat services, you'll need to notify UKCloud for access to these repositories or will need to bring your own licenses to support these additional services. If required, we can provide access to additional repositories such as Scalable File System, Load Balancer or Resilient Storage. To access these services, please raise a request via the [My Calls](https://portal.skyscapecloud.com/support/ivanti) section in the UKCloud Portal and it will be uploaded to your private catalog.

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
