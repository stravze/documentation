---
title: Cloud GPU Service Scope
description: Outlines important details regarding Cloud GPU
services: gpu
author: Steve Hall
reviewer: Guy Martin
lastreviewed: 28/06/2019
toc_rootlink: Service Scope
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Cloud GPU Service Scope
toc_fullpath: Service Scope/gpu-sco.md
toc_mdlink: gpu-sco.md
---

# Cloud GPU Service Scope

> [!IMPORTANT]
> Cloud GPU has been retired from sale by UKCloud. We will continue to support all existing customers who are using this service, however, we are no longer providing this service for new workloads. This article provides existing Cloud GPU customers with access to support documentation and we will continue to update it as required. For new requests, contact your Account Manager or Service Delivery Manager.

## About this document

This document describes the boundaries of the Cloud GPU service, along with the division of responsibilities between you and UKCloud, to facilitate the provisioning and ongoing use of the service.

## About Cloud GPU

Cloud GPU enables you to supplement your on-platform compute resources with GPU capabilities.

Utilise the UKCloud for VMware platform with Cloud GPU to meet the specialist requirements of some advanced applications, with the benefits of a cloud environment. It supports the following types of workloads:

- Visualisation workloads. Traditional use cases for GPU processing such as simulations, powering desktop applications with graphics content (for example computer aided design), video encoding, rendering or streaming. Visualisation is currently not suitable for virtual reality (VR) workloads.

- Compute workloads. GPUs have become prevalent in a world that needs a lot of data processing, fast. The parallel nature of GPU cores lends themselves perfectly to support compute-intensive initiatives such as deep learning and large-scale mathematical modelling.

## Service architecture

Cloud GPU is an add-on service for UKCloud for VMware. A GPU product must be purchased in conjunction with UKCloud for VMware and be attached to a PRIORITY VM within a GPU-enabled region.

GPU resource can be assigned to a VM in two ways, depending on the type of GPU acceleration needed.

- Visualisation workloads can be carried out using our NVIDIA M60 GPUs. These are presented to the VM as a profile of a certain size. There are multiple profiles of different sizes that can be associated with a VM

- Compute workloads can be carried out using our NVIDIA P100 GPUs. These are currently presented to a VM on a per-card basis, enabling the VM to have access to the full resource of a card. Currently only two cards can be associated with a single VM.

Visualisation GPUs are mapped to a VM through NVIDIA GRID, and the GRID licence is included in the GPU price.

Visualisation GPUs use a GRID Q profile. Compute GPUs use a GRID B profile, which can be changed to a Q profile by a Service Request.

## Service options

A Cloud GPU-enabled VM must reside within a PRIORITY VDC within a UKCloud for VMware region that is GPU enabled.

Workload type | Resource allocation | Automated Rebalancing
--------------|---------------------|----------------------
PRIORITY | Uncontended (CPU/GiB) | Configured to reduce workload movement around the platform, reducing workload disruption

To have GPU enabled in your environment, you will need to raise a Service Request via the [My Calls](https://portal.skyscapecloud.com/support/ivanti) section of the UKCloud Portal.

When you raise a service request, you will need to specify the profile that you want to be made available from the following options:

Workload type | Presentation | Memory size
--------------|--------------|------------
Visualisation | Profile<br>(slice of card) | 1GB<br>2GB<br>4GB<br>8GB
Compute | Card | 16GB

## Storage

Storage can be provisioned to the GPU-enabled VM. This is done independently from GPU provisioning and is within the customers control. For more information, see the [*UKCloud for VMware Service Scope*](../vmware/vmw-sco.md).

## Protection

Snapshots for GPU-enabled VMs are currently not available, but you can use Zerto to protect GPU-enabled VMs.

You can find more information about protection options in the [*UKCloud for VMware Service Scope*](../vmware/vmw-sco.md).

## Service availability

The Service Level Agreement for Cloud GPU is 99.90%.

Unavailability applies to the inability to connect to a new GPU resource in the event of a failure of a GPU service within a single zone. Failure condition is following a hardware fault recognised at the Infrastructure layer or below, and within UKCloud-controlled components.

The SLA of 99.90% supersedes any other SLA relating to the VM that the GPU is associated with.

Planned and Emergency Maintenance are excluded from calculations.

You are entitled to claim Service Credits for outages to services where the SLA falls below 99.90%. For more about how we calculate SLAs, see the [*SLA Definition*](../other/other-ref-sla-definition.md).

## Service background

### The following applies to the Cloud GPU service

- GPU-enabled VM templates control the allocation of GPU resources.

- Visualisation GPU profiles are fully provisioned - you will get 100% of GPU profile resource allocation.

- Compute GPUs are fully provisioned - you will get 100% of GPU resource allocation.

- We actively capacity-manage the cloud platform to ensure you have access to the GPU resources you request.

- You can specify Farnborough or Corsham as the site where you would like to have your Cloud GPU service provisioned. We will try to accommodate requests and will advise you if we are unable do so.

- We control the deployed versions of GPU technology on the platform. This covers internal platform-supporting technologies, and any technology versions available to you. This includes, but isn't limited to, the NVIDIA GRID and the NVIDIA hardware version of the GPU service.

- You can make additional configurations inside a GPU-enabled VM, such as implementing third-party software technologies. Customer implementations inside a VM are solely the responsibility of the customer including any patches or updates.

## Service resilience

- You should be aware that the disruptive nature of GPU hardware failures means that in the event of a GPU hardware failure, the GPU-enabled VM will fail.

- In this event, UKCloud will be responsible for connecting the VM to another available GPU inside the zone. If there are no GPU hosts available inside a zone for connection to, an SLA event will be triggered.

- Additionally, you should also be aware that VMs using the Cloud GPU solution will have on-platform resilience tools (such as vMotion and HA on VMware) disabled. This maintains the connection to the GPU, but reduces the automatic recovery of those VMs in the event of a failure.

## Platform monitoring

As well as monitoring the components that underpin the UKCloud for VMware aspect of the Cloud GPU service, UKCloud also monitor the GPU hardware health and will proactively manage the physical aspects of the GPU to ensure service availability.

UKCloud reserve the right to proactively move a GPU-enabled VM to another GPU-enabled host if we believe a critical component failure is imminent. This action is covered by the GPU SLA.

## Operating systems

**Licencing.** We provide the GRID licence which is included within the GPU price.

UKCloud can also provide Operating Systems (OS) licensing, and is mandatory with respect to Windows Server. For more information, see the [UKCloud Pricing Guide](https://ukcloud.com/pricing-guide).

**VM server images.** We provide base GPU-enabled VM images for the OS for which we provide licensing. You can access these from VMware Cloud Director once the GPU-enabled image has been made available to you.

**Update services.** We make update repositories available for all software for which we provide licensing. We don't provide software update facilities for non-UKCloud licensed software.

**Anti-virus.** We do not provide anti-virus software as part of the service.

## Networks

All networks available to the UKCloud for VMware platform are available for GPU-enabled VMs.

A full overview of the connectivity options and network architecture is available in [*Understanding connectivity options in UKCloud for VMware*](../vmware/vmw-ref-connectivity-options.md).

## Protective monitoring

We have implemented GPG 13-aligned Protective Monitoring across the Assured and Elevated platforms at the hypervisor level and below.

We do not provide Protective Monitoring services above the hypervisor (for example, for your VM) - it is your responsibility to monitor at this level.

In line with UKCloud's SISP, we provide notification of customer-impacting security incidents. It is your responsibility to report similar incidents to us.

## Platform management

You can access, manage and view the GPU-connected VMs in the same way that you would manage the rest of your UKCloud for VMware service. Access only those features allowed by their role, in any of the following ways:

- **vCloud API.** Enables the programmatic creation and management of GPU-enabled VMs inside the platform.

- **VMware Cloud Director tenant portal.** Provides a graphical interface to access the VMware Cloud Director environment (depending on assigned permissions) and manage GPU-enabled VMs.

- **UKCloud Portal.** Enables the creation of compute services and subsequently, PRIORITY VDCs and the requesting of GPU resources for environments. The Portal also includes an overview of actual and estimated spend, along with service configuration information. Access to incident and request management is also possible through the UKCloud Portal.

You cannot access the underlying infrastructure. This includes (but isn't limited to) the GPU hardware and the GPU management plane.

## Service reporting

**Visibility.** Maintenance notifications and Service Status reports are delivered through the Portal.

**Billing.** We provide you with monthly invoices and detailed CSV reports covering your previous monthly spend. Your GPU-enabled machines will be within the reports, and a separate column will provide you with information regarding your GPU usage.

## Customer service

**Cloud Architect.** UKCloud Cloud Architects support you during the design of solutions for the cloud platform. They are ideally placed to help reconcile your requirements with the UKCloud platform. We recommend engagement with a Cloud Architect when implementing complex solutions, such as those using HybridConnect or a Walled Garden.

**Service Delivery Manager (SDM).** An assigned point of contact who will provide any assistance you require during your use of the service, including onboarding, service reviews and incident reporting and escalation.

**Support Desk.** After the initial on-boarding and design phase, you can utilise the standard UKCloud support entitlement, which is outlined in the [*Customer Engagement Factsheet*](https://ukcloud.com/wp-content/uploads/2018/08/ukcloud-factsheet-customer-care.pdf).

## Customer responsibilities

Alongside the responsibilities outlined in the [*UKCloud for VMware Service Scope*](../vmware/vmw-sco.md), you are responsible for assessing whether the UKCloud GPU service can support the various requirements of your application.

## Service provisioning

Outside of the creation of your UKCloud for VMware resources needed to support your GPU service, customers can request the addition of GPU capabilities in GPU-enabled zones via the service request option, and the service will be available within 10 working days.

If you request access to Cloud GPU resources for existing VMs that are not in a GPU-enabled zone, our SDMs will work with you to configure new UKCloud for VMware, High Performance Compute or UKCloud for OpenStack services in the correct zone.

UKCloud has created several videos, helpful guides, manuals and FAQs to help train and instruct users so that they are up and running quickly and easily. These are available within the [Knowledge Centre](https://docs.ukcloud.com/).

As previously mentioned, your Service Delivery Manager (SDM) will provide any assistance during the provisioning of the service.

UKCloud also has a large ecosystem of partners who can deliver additional services, such as support and professional services. UKCloud would be pleased to introduce you to the right partner to suit your needs.

## Service constraints

For information about Planned and Emergency Maintenance, see [*Understanding UKCloud service maintenance windows*](../other/other-ref-maintenance-windows.md).

You should be aware of some of the limitations of parts of the SLA. The disruptive nature of GPU hardware failures means that in the event of a GPU hardware failure, the VM will also fail.

In this event, UKCloud will be responsible for connecting the VM to another available GPU inside the zone. If there are no GPU hosts available inside a zone for connection to, an SLA event will be triggered.

You should be aware that by adding a Cloud GPU to a VM, the SLA for the core compute service is replaced for that VM.

## Supporting documents and resources

The following documents contain more information about Cloud GPU:

- [*Cloud GPU FAQ*](gpu-faq.md)

- [*Cloud GPU Factsheet*](https://ukcloud.com/wp-content/uploads/2018/08/cloud-gpu-factsheet.pdf)

- [*How to set up Cloud GPU Compute for UKCloud for VMware*](../vmware/vmw-how-setup-gpu-compute.md)

- [*How to set up Cloud GPU Visualisation for UKCloud for VMware*](../vmware/vmw-how-setup-gpu-visualisation.md)

- [*UKCloud Terms & Conditions for G-Cloud 11*](../other/other-ref-terms-and-conditions.md)

- [*UKCloud SLA Definition*](../other/other-ref-sla-definition.md)

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
