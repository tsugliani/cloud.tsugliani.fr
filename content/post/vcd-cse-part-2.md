---
title: "vCloud Director & Container Service Extension - Part 2: Requirements"
date: 2018-04-18T18:59:34+02:00

categories:
- vCloud Director
- NSX
- CSE

tags:
- vmware
- cloud
- api
- kubernetes
thumbnailImage: /img/ospo-logo.png
thumbnailImagePosition: left
slug: "vcloud director and cse part 2"
draft: false
---

This post will depict the various requirements a Service Provider will have to fulfill to provide CSE for their end users so they can deploy & manage [Kubernetes](https://kubernetes.io/) clusters.

<!--more-->

Most of the requirements are listed on the [container-service-extension](https://vmware.github.io/container-service-extension/) documentation, but isn't really visual, so hopefully the following diagrams should help understand how everything comes together into a working container as a service solution.

So let's add a bit more detail to a previous diagram to showcase which **APIs** are into play here and refer back to the requirements for every component, and then from a vCloud Director standpoint.

**vCloud Director & CSE API Logical Diagram**
<center>
  ![vCD and CSE API Logical Diagram](/img/vcd-cse-api-logical-diagram.svg)
</center>

I wanted to highlight how **AMQP** plays a very important role when extending vCloud Director, and show which component is interacting with each other through the **APIs** and/or **AMQP**.

For anyone unfamiliar with **AMQP**, I would recommend checking out this blog post:

- [Part 1: RabbitMQ for beginners - What is RabbitMQ?](https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html)



Now for the 2 main components requirements

## Container Service Extension (CSE)

- vCloud Director **System Administrator** credentials
- vSphere **Administrator** credentials (at the day of writing this post)
- RabbitMQ information & credentials

- [Python 3](https://www.python.org/):
  - [pyvcloud](https://vmware.github.io/pyvcloud/)
  - [pyvmomi](https://github.com/vmware/pyvmomi)
  - [vsphere-guest-run](https://github.com/vmware/vsphere-guest-run)
  - [vcd-cli](https://vmware.github.io/vcd-cli/)
  - [container-service-extension](https://vmware.github.io/container-service-extension/)

- Network:
  - Access to vCloud Director
  - Access to vCenter Server
  - Access to RabbitMQ

## vCloud Director

- Release 8.10 minimum
- **System Administrator** credentials
- Supported RabbitMQ instance configured
  - Check vCloud Director <-> RabbitMQ supported matrix on vCD Release Notes
  - [vCloud Director 9.1 Release notes](https://docs.vmware.com/en/vCloud-Director/9.1/rn/rel_notes_vcloud_director_91.html)

- Catalog / Admin Organization setup:
  - Org vDC with **sufficient** resources to spin up the template creation process
  - Org vDC Network **with Internet access** (to download kubernetes components and various packages)

To illustrate the vCloud Director part, Here is a diagram that shows how your setup could look like.

**vCloud Director & Admin / Catalog Organization Sample Diagram**
<center>
  ![vCD and CSE API Logical Diagram](/img/vcd-cse-sp-org-diagram.svg)
</center>

High level steps:

1. CSE will upload vanilla templates OVA to vCloud Director in a CSE Catalog
2. CSE will deploy the template as a VM on a Org VDC Network that requires internet access.
3. CSE will guest customize the VM and add/configure all required components.
4. CSE will then validate the VM and add it back to the CSE Catalog as a valid item for deploying container hosts.

Hope this helps,

In the next blog post we will cover the [Installation & Configuration](https://cloud.tsugliani.fr/2018/07/vcloud-director-and-cse-part-3/) part in more detail.