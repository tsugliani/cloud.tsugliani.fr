---
title: "vCloud Director & Container Service Extension - Part 1: Architecture"
date: 2018-04-03T15:42:15+02:00
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
slug: "vcloud director and cse part 1"
draft: false
---

This post will depict the general architecture of CSE and how it achieves its purpose.

<!--more-->

Before going into the various CSE *Requirements, Installation & Configuration* details, I think it's important to explain the Architecture of all the components that constitute this Solution and what it means for a Service Provider and a Tenant/End User.

The overall goal of CSE is to provide the following use-cases:

# Kubernetes as a Service

{{<
    image classes="fancybox clear"
    src="/img/vcd-cse-overview-flow.png"
    title="vCloud Director & Container Service Extension (CSE)"
>}}

Basically a Service Provider will offer compute resources to tenants secured through a multitenant IaaS vCloud Director instance, and tenants/end users will have the ability to deploy & manage their kubernetes clusters in a self service fashion.

As you can see in the above picture, we listed some very basic & simple operations to manage the lifecycle of a cluster (create, list, update, delete, commonly known as [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations).

Once their cluster(s) is/are available, the developers will go back to their native kubernetes tooling and leverage the cloud provider resources.

so the initial question that comes to mind is:

*What are the components required for building a Kubernetes cluster ?*

From the kubernetes [website](https://kubernetes.io/) we found a basic diagram depicting what a kubernetes cluster looks like.
This will help understanding all the components CSE should provide/manage to simplify the Tenants/End Users kubernetes usage on a cloud platform.

<center>
  ![Kubernetes Overview](/img/kubernetes-architecture.svg)
</center>
This is a very simplistic view, so let's take the opportunity to describe the components in a bit more detail and how most common deployments may look like.

Kubernetes has 3 major components:

- [etcd](https://coreos.com/etcd/) (Distributed key/value store)
- kubernetes **master** (Controller Manager, Scheduler and API Server)
- kubernetes **worker nodes** (kubelet, proxy and runtime)

I'm not going to describe in detail what each of them do, but you can find the additional information on the [Kubernetes components overview](https://kubernetes.io/docs/concepts/overview/components/) page.

So to summarize *most* deployments will look like these 2 generic nodes.

![Kubernetes Overview](/img/kubernetes-conceptual-deployment.svg)

Obviously various deployment models exist depending on the cluster requirements such as, scale, operations, etc.
They can differ, but this seems to be a pretty common model, the other main alternative option would be to keep the etcd cluster decoupled from the master nodes.

If we try to setup a fully resilient kubernetes cluster in a vCloud Director context this would probably look like this.

![Kubernetes Overview](/img/vcd-cse-kubernetes-logical-diagram.svg)

# CSE Conceptual Architecture

So now that we understand what a kubernetes cluster is, we can focus on how CSE interacts with vCloud Director to make this service possible.

<center>
  ![vCloud Director & CSE](/img/vcd-cse-conceptual-diagram.svg)
</center>

We are going to keep things simple here and explain how CSE achieves it's aims.
This was briefly mentioned in the previous post, so let's clarify/detail some aspects.

**First**, it requires a way to extend the vCloud Director capabilities so that anyone can access the new service.

This is done by extending the public vCloud API (Check [Extending VMware vCloud® API with vCloud Extensibility Framework](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/vcat/vmware-vcloud-api-extension-whitepaper.pdf) for additional information)

> Note: Extending the vCloud API requires an AMQP broker.

This will obviously expose new capabilities to any new Tenant/Consumer of the vCloud Director instance.

**Second**, it requires a way to deploy container hosts & install / configure kubernetes

CSE is exposed on top of the vCloud API, which means it has now the ability to specify what kind of tasks/operations the service will need to offer in a programatic fashion.

Those tasks could be as simple as:

- Create a new kubernetes cluster
- List the kubernetes clusters
- Update a kubernetes cluster (Scale Out/In and/or Up/Down for example)
- Delete a kubernetes cluster

We now need to decouple those tasks in smaller usable entities that match those which vCloud Director/vSphere can manage.

Obvious things we need:

- A VM for each node - **vCloud Director** covers this requirement
- A way to customize the VM (Operating System, credentials, networking, etc) - **vCloud Director** covers this requirement
- A way to customize the guest operating system applications, such as adding kubernetes components, docker, etc. - **CSE** covers this requirement
- A way to customize configuration for the kubernetes environment - **CSE** covers this requirement

> Note: I will describe how each of these steps are actually implemented in related blog posts.

**Third**, it requires a simple & easy way to let a tenant/consumer manage the lifecycle of the kubernetes cluster.
API is great for automation, but for Admin/DevOps people, a CLI is a welcome addition.

CSE comes with a [vcd-cli](https://vmware.github.io/vcd-cli/) extension that leverages the public CSE API into basic administrative/management lifecycle commands to control/operate the kubernetes cluster.

So now we have it, Kubernetes as a Service on top of a multitenant VMware IAAS Platform. (vCloud Director)

This is how the architecture looks with the current version of the service at the time of writing this blog post.

![CSE Architecture Overview](/img/vcd-cse-architecture-overview-diagram.svg)

Next blog post we will go into the Service Provider [Requirements](https://cloud.tsugliani.fr/2018/04/vcloud-director-and-cse-part-2/).






