---
title: Consuming existing customer services on UKCloud from OpenShift
description: Provides information regarding connecting out from OpenShift clusters to a customers existing services on UKCloud
services: openshift
author: Kieran O'Neill
reviewer: 
lastreviewed: 11/03/2021

toc_rootlink: Reference
toc_sub1: OpenShift v4.x
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Consuming existing customer services on UKCloud from OpenShift
toc_fullpath: Reference/oshift-ref-consume-services-v4.md
toc_mdlink: oshift-ref-consume-services-v4.md
---

# Consuming existing customer services on UKCloud from OpenShift

## Overview

This article outlines how to connect into existing services hosted on UKCloud from your OpenShift cluster via a private network.

### Intended audience

This article is aimed at those with OpenShift deployments with connectivity to a private network.

### Pre-requisites

You will need an existing UKCloud for VMWare, UKCloud for Openstack or UKCloud for OpenShift deployment to connect to as well as the necessary networking set up to connect to. 
This can be requested via a service request or by speaking with your account director. It is best to discuss networking requirements prior to the deployment of your OpenShift cluster
however it can be retrofit if necessary.

### Connecting

Connecting outbound to your services will be achieved via a separate gateway on the OpenShift network using SNAT. Routes will be added to the OpenShift cluster to your desired
subnets via this private gateway. You will need to ensure that subnets do not overlap to ensure the routing works. The SNAT IP will be provided to you so you can allow it
through any firewalls you may have configured.

<diagram here?>

### Use cases

You are deploying an application on OpenShift and have chosen to deploy the backend database on UKCloud for OpenStack for greater resiliency. You will deploy your database within
your OpenStack project and provide the subnet this resides on to us for your OpenShift deployment. You must ensure this subnet does not overlap with the OpenShift subnet which can be configured
to your choosing. Routing will be configured inside OpenShift to allow the connectivity and a DNS forwarding zone can be setup for the private network if this is desired.
Traffic between your OpenShift environment and the OpenStack environment will not leave UKClouds infrastructure and will ensure low latency.

## Further information


## Feedback

If you find an issue with this article, click **Improve this Doc** to suggest a change. If you have an idea for how we could improve any of our services, visit [UKCloud Ideas](https://ideas.ukcloud.com). Alternatively, you can contact us at <products@ukcloud.com>.
