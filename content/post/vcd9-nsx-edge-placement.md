---
title: "vCloud Director 9.0: NSX Edge Gateway cluster placement"
date: 2017-09-24
categories:
- vCloud Director
- NSX

tags:
- vmware
- cloud
- design
- api
- network
thumbnailImage: /img/vcloud.png
thumbnailImagePosition: left
---

This post will highlight a long awaited feature, which is now available in vCloud Director 9.0.

We will depict how to **enforce** the placement of the **NSX Edge Gateways** in a resource pool, which ultimately leverages a **specific vSphere Cluster**.

It avoids having to spread all **External** VLANs across every **Compute** Cluster and enforce all the traffic connecting to the outside world would go through a specific location. (**Cluster/TOR/Racks**)

In the previous versions of vCloud Director, one thing we couldn't accomplish easily was to ensure all N/S traffic from the NSX VXLAN overlay to the physical underlay/networks, which is usually VLAN Based **AND** at the same time keep the compute workloads VMs seperated as depicted in the [vCAT-SP - Architecting a vCloud Director Solution / NSX Edge Cluster Design options](http://download3.vmware.com/vcat/vmw-vcloud-architecture-toolkit-spv1-webworks/index.html#page/Cloud%2520Automation%2520and%2520Orchestration%2FArchitecting%2520a%2520vCloud%2520Director%2520Solution%2FArchitecting%2520a%2520vCloud%2520Director%2520Solution.1.036.html%23)


This should mitigate all the previous drawbacks & overhead when using this setup in a leaf & spine scenario.
The below diagram shows the updated behavior when leveraging the feature.

>

{{<
    image classes="fancybox clear"
    src="/img/vcd-edge-cluster-problem-statement.png"
    title="Updated NSX Edge Cluster behavior with spine & leaf topology"
>}}


# Background

You might have encountered 2 main following architectures with NSX:

- **Management**, **Edge** & **Compute** Clusters (requires at least **3** clusters)
- **Collapsed Management/Edge** & **Compute** Clusters (requires at least **2** clusters)

I will not go into details comparing the various pro/cons of both, but just to set a bit of background on why this new feature now exists. They each have benefits and disadvantages depicted in the **NSX Design considerations** section of the [NSX Design Guide](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/products/nsx/vmw-nsx-network-virtualization-design-guide.pdf)

Then what is expected with such a design is:

- N/S traffic goes through the Edge Cluster (VXLAN to VLAN traffic)
- E/W traffic goes through the DLR in the compute(s) clusters and also Edge Cluster (***Note:*** Usually only VXLAN traffic)

So the good news, is that with vCloud Director 9.0 we can now achieve the same recommended design, let's see how this can be implemented.

# Setup

Taking a step back and mapping what we just explained to vCloud Director leveraging NSX, this is what it would look like.

>

{{<
    image classes="fancybox clear"
    src="/img/nsx-edge-cluster-placement.png"
    title="NSX Edge Gateway Cluster Placement High Level Topology"
>}}

The setup still requires some preparation but can be pretty flexible as shown in the above diagram. It relies on vCloud Metadata **`key/values`** that will trigger and enable a specific placement algorithm engine once configured.


***Note:*** This will deny any normal compute workloads to run in that specific resource pool

Requirements:

1. Create a resource pool identifying the Provider VDC in the Edge Cluster on vSphere
2. Each Resource Pool created previously will be added in the Provider vDC
  - In my example you can see the following:
     - There is 2 Provider VDCs, **PvDC A** and **PvDC B**, and we created 2 resource pools to match those in the Edge Cluster **RP PvDC A** and
        **RP PvDC B**
     - Added the resource pool **RP PvDC A** to the **PvDC A** in vCloud Director
     - Added the resource pool **RP PvDC B** to the **PvDC B** in vCloud Director
3. in vCloud Director Edit the Provider vDC as a System Administrator and add the
  following metadata as a key/value pair with the MoRef ID of the resource pool matching the PvDC you are editing.

    ```javascript
        placement.resourcepool.edge = <resgroup-id>
    ```

    {{<
        image classes="fancybox clear"
        src="/img/vcd-edge-cluster-metadata.png"
        title="Provider VDC Metadata"
    >}}
    if you don't know how to fetch the `resgroup-id` Management Object Reference Identifier value check the next section

4. Redeploy the Edge Gateways to force the new placement logic.

## How to fetch the resource pool id ?

There are various ways to fetch those IDs, here are the most common ones you might be familiar with.

### From PowerCLI

This is probably the easiest way for most people to gather the information.

Steps:

1. Fetch the cluster and the resource pool objects

    ```powershell
        $cluster = Get-Cluster -Name "CAI Edge Cluster"
        $respool = $cluster | Get-ResourcePool -Name "PvDC-Cairo-EdgeRP"
    ```

2. Print the Management Object Reference

    ```powershell
    (Get-View $respool).MoRef
    Type               Value
    ----               -----
    ResourcePool       resgroup-1527
    ```

3. The result is **resgroup-1527**.

### From the vSphere Managed Object Browser (MOB)

This requires a bit of habit leveraing the MOB, but with a bit of tinkering, through the vSphere Object models you should find your clusters and the required information.

Steps:

1. Open a browser to your vcenter url: **https://vcenter/mob**
2. Login with your environment credentials
3. Navigate around ;-)

>

{{<
    image classes="fancybox clear"
    src="/img/nsx-edge-cluster-placement-mob.png"
    title="vSphere Managed Object Browser"
>}}

### From the vCloud API

Thanks to [Tomas Fojta](https://fojta.wordpress.com/about/) who found the most relevant [API request](http://pubs.vmware.com/vcd-820/index.jsp#com.vmware.vcloud.api.reference.doc_27_0/doc/operations/GET-DiscoverRPForAdoption.html) to fetch both the resource pool name & ID so we can easily identify the appropriate resource pool.

`GET /api/admin/extension/providervdc/<id>/discoverResourcePools`

Sample request leveraging [Postman](https://www.getpostman.com):

>

{{<
    image classes="fancybox clear"
    src="/img/nsx-edge-cluster-placement-vcloud-api.png"
    title="vCloud API Request"
>}}

# Validating the configuration

Once the Provider VDC Metadata has been configured with the added metadata, head back to vCloud Director, and test that this new feature works as expected by redeploying an Edge Gateway.
You should witness something similar to the following animated process.

>

{{<
    image classes="fancybox clear"
    src="/img/nsx-edge-gateway-redeploy.gif"
    title="Redeploying an Edge Gateway"
>}}

You could easily automate the whole process, from setting the required metadata to the edges redeployment, leveraging both the vSphere & vCloud API.

Hope this helps.