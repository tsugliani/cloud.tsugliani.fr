---
title: "vCloud Director & Container Service Extension"
date: 2018-03-29T15:08:08+01:00
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
slug: "vcloud director and cse"
draft: false
---

This is the first post of a series that will highlight a new extension for vCloud Director that offers the ability for tenants to manage [Kubernetes](https://kubernetes.io/) clusters.

<!--more-->

{{<
    image classes="fancybox clear"
    src="/img/vcd-cse-overview.png"
    title="vCloud Director & CSE"
>}}

[Container Service Extension](https://vmware.github.io/container-service-extension/) 1.0.0 was just released with vCloud Director 9.1.0, and is officially supported by VMware.

I have been personally involved and giving a lot of feedback on the project since the early beta days, and very pleased to see how far it has come, and continues to improve from all the constructive feedback it received through the last months.

My colleague [Steve Dockar](https://twitter.com/SteveDockar) & myself have been working on various content around it (Whitepaper, Training materials, POC, etc) and obviously fetching all the relevant ideas, thoughts from early adopters over the last 2 or 3 months and bringing those back to [Paco Gomez](https://twitter.com/pacogomez) who leads this project.

One important thing to mention, is that Container Service Extension (**CSE**) is an Open Source project hosted here:

- GitHub: [https://github.com/vmware/container-service-extension](https://github.com/vmware/container-service-extension)

The Goal of CSE is to provide a very simple way of deploying & managing kubernetes clusters on top of vCloud Director, and allow any developer to focus on the application layer using his known tools offering a *frictionless* #devops experience.

To achieve this, CSE does various things:

- It **extends the vCloud API** to offer the ability to manage the lifecycle of Kubernetes clusters (Swagger available)
- It **provides a way to build automatically Host Container VM Templates** (Those are the templates an end user will be deploying & preconfiguring through this service)
- [vcd-cli](https://github.com/vmware/vcd-cli) is the official vCloud Director command line interface that has been recently updated to leverage the CSE API through an extension to facilitate the interaction/management.

CSE is getting more and more traction (thanks to the popular kubernetes trend), we will try to depict the most common topics in various different blog posts which should cover the following sections:

From a **Service Provider** perspective:

- [Architecture](https://cloud.tsugliani.fr/2018/04/vcloud-director-and-cse-part-1/)
- [Requirements](https://cloud.tsugliani.fr/2018/04/vcloud-director-and-cse-part-2/)
- [Installation & Configuration](https://cloud.tsugliani.fr/2018/07/vcloud-director-and-cse-part-3/)
- Troubleshooting / Tips & Tricks
- Extending
  - Templates
  - Sample customization

From a **Tenant** (End User) perspective:

- Requirements / Installation & Configuration
- Usage (Common use cases)
  - Routed network (Connected directly)
  - SSL VPN (Nomad Developer)
  - IPSec (Developer Branch Office example)
- Troubleshooting / Tips & Tricks
- Extending
  - Ingress Controller (Traefik/NSX LB)

And more *generally*:

- Current Pros/Cons
- Roadmap

Hope this will bring answers to most of your questions about this project. <br />
If something is still unclear or bothering you, let me know.
