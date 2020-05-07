---
title: How to export a disk and use it to create a VM in Hyper-V using the UKCloud Azure Stack Hub portal
description: Details the process of exporting a disk in the UKCloud Azure Stack Hub portal and then using it to create a virtual machine in Hyper-V manager
services: azure-stack
author: William Turner
reviewer: William Turner 
lastreviewed: 07/05/2020

toc_rootlink: Users
toc_sub1: How To
toc_sub2: Export a disk to Hyper-V
toc_sub3:
toc_sub4:
toc_title: Export a disk to Hyper-V - Portal
toc_fullpath: Users/How To/Export a disk to Hyper-V/azs-how-export-disk-to-hyperv-portal.md
toc_mdlink: azs-how-export-disk-to-hyperv-portal.md
---

# How to export a disk and use it to create a VM in Hyper-V using the UKCloud Azure Stack Hub portal

## Overview

The following article shows you how to export a disk in the UKCloud Azure Stack Hub portal and use it to create a virtual machine in Hyper-V manager.

## Intended audience

To complete the steps in this article, you must have appropriate access to a subscription in the Azure Stack Hub portal.

## Exporting a disk from an existing virtual machine using the Azure Stack Hub portal

1. Log in to the Azure Stack Hub portal.

    For more detailed instructions, see the [*Getting Started Guide for UKCloud for Microsoft Azure*](azs-gs.md).

2. In the *favourites* panel, select **Virtual machines**.

    ![Virtual machines button](images/azsp_vmsmenu.png)

3. In the *Virtual machines* blade, select the VM that you want to add the disk to.

    ![Virtual machine selection](images/azs-browser-button-vm-disks.png)

4. Under *Settings*, select **Disks**.

    ![Virtual machine disk button](images/azs-browser-button-vm-disks-setting.png)

5. In the *Disks* blade, select the disk that you would like to export.

    ![Disks blade - Select disk](images/wip.png)

6. In the selected disks' blade, under settings, select **Disk Export**.

    > [!IMPORTANT]
    > A disk cannot be exported if it is attached to a running VM. You will need to stop the VM first to be able to export the disk.

7. 

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
