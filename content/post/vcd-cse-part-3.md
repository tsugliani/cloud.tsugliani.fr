---
title: "vCloud Director & Container Service Extension - Part 3: Installation & Configuration"
date: 2018-07-17T13:52:34+02:00

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
slug: "vcloud director and cse part 3"
draft: false
---

This post will depict the installation & configuration process for the Service Provider components.

<!--more-->

# Installation

The CSE service is designed to be installed by the vCloud Director **System Administrator** on a virtual machine (the CSE appliance) with network connectivity to the vCloud Director infrastructure where the following components access is required:<br/>
<br />

* vCloud Director instance (Public Load Balancer VIP for multiple cells)
* vCenter Server(s)
* AMQP Server

For the simplicity of this blog post, I'll leverage a [Photon OS](https://vmware.github.io/photon/) 2.0 VM based on the official OVA available [here](https://github.com/vmware/photon/wiki/Downloading-Photon-OS#photon-os-20-ga-binaries).

Once deployed, make sure you configured networking correctly from our requirement bulleted list above, leverage the following [Administrator Guide](https://github.com/vmware/photon/blob/master/docs/photon-admin-guide.md#using-the-network-configuration-manager)
 related section.

Next step is to install all the required components by executing the following commands to install the required system packages + python3 related modules. (as It's my nested lab, I installed everything with the `root` user, but you can obviously dedicate a `cse` user)

```bash
root@cse [ ~ ]# export LANG=en_US.UTF-8
root@cse [ ~ ]# tdnf remove toybox -y
root@cse [ ~ ]# tdnf install -y build-essential python3-setuptools python3-tools python3-pip python3-devel
root@cse [ ~ ]# pip3 install --upgrade pip
root@cse [ ~ ]# pip3 install --upgrade container-service-extension
root@cse [ ~ ]# cse version
```

I made a small [Asciinema](https://asciinema.org) recording on how the whole installation process would look like. (leveraging a docker photon os container to replicate the installation part quickly)

<script src="https://asciinema.org/a/192456.js" id="asciicast-192456" async></script>

Now that Container Service Extension is installed, the next logical step is to setup the configuration and run the service.

# Configuration

CSE offers a way to generate a `sample` config file by executing the following command.

```bash
root@cse [ ~ ]# cse sample > $HOME/config.yaml
```

Edit the file and identify the following sections:<br />
<br />

- **amqp**<br />
  This section covers the RabbitMQ Message Bus that vCloud Director leverages for all blocking tasks/notifications and extensibility. <br />
  *-> Make sure to modify this section to match your environment settings* <br /><br />
  Sample: <br />

    ```yaml
    amqp:
          exchange: vcdext
          host: rabbitmq.fqdn.com
          username: guest
          password: guest
          port: 5672
          prefix: vcd
          routing_key: cse
          ssl: false
          ssl_accept_all: false
          vhost: /
    ```

- **vcd**<br />
  This section covers all your vCloud Director environment settings. <br />
  *-> Make sure to modify this section to match your environment settings* <br /><br />
  Sample: <br />

    ```yaml
    vcd:
          api_version: '29.0'
          host: vcd.fqdn.com
          port: 443
          username: administrator
          password: my_secret_password
          verify: false
          log: true
    ```

- **vcs**<br />
  This section covers the vCenter(s) environment settings.<br />
  Since CSE 1.0 we now support multiple vCenter Servers within the same vCloud Director environment. <br />
  *-> Make sure to modify this section to match your environment settings* <br /><br />
  Sample: <br />

    ```yaml
    vcs:
        - name: vcenter-compute-01.fqdn.com
          username: administrator@corp.local
          password: mysecretpassword
          verify: false
        - name: vcenter-compute-02.fqdn.com
          username: administrator@vsphere.local
          password: mysecretpassword
          verify: false

    ```

- **broker**<br />
  This section covers all the internal vCloud Director constructs + templates that CSE will be leveraging to enable this service for every tenant. The sample configuration provides 2 templates for the container hosts (1 photon OS & 1 ubuntu) <br />
  *-> Make sure to modify this section to match your environment settings* <br /><br />
  Sample: <br />

      ```yaml
      broker:
        catalog: cse
        cse_msg_dir: /tmp/cse
        default_template: photon-v2
        ip_allocation_mode: pool
        network: myOrgRoutedNetwork
        org: MyAdminCatalogOrg
        storage_profile: '*'
        templates:
        - admin_password: myTemplateRootPassword
          catalog_item: photon-custom-hw11-2.0-304b817-k8s
          cleanup: true
          cpu: 2
          description: 'PhotonOS v2 / Docker 17.06.0-4 / Kubernetes 1.9.1 / weave 2.3.0'
          mem: 2048
          name: photon-v2
          sha1_ova: b8c183785bbf582bcd1be7cde7c22e5758fb3f16
          source_ova: http://dl.bintray.com/vmware/photon/2.0/GA/ova/photon-custom-hw11-2.0-304b817.ova
          source_ova_name: photon-custom-hw11-2.0-304b817.ova
          temp_vapp: photon2-temp
        - admin_password: myTemplateRootPassword
          catalog_item: ubuntu-16.04-server-cloudimg-amd64-k8s
          cleanup: true
          cpu: 2
          description: 'Ubuntu 16.04 / Docker 18.03.0~ce / Kubernetes 1.10.1 / weave 2.3.0'
          mem: 2048
          name: ubuntu-16.04
          sha1_ova: 12014032ec640c9dd98e1839adbf5c40aca86344
          source_ova: https://cloud-images.ubuntu.com/releases/xenial/release-20180418/ubuntu-16.04-server-cloudimg-amd64.ova
          source_ova_name: ubuntu-16.04-server-cloudimg-amd64.ova
          temp_vapp: ubuntu1604-temp
        type: default
        vdc: myAdminCatalogOrgVDC
      ```

Once you completed every section with the appropriate parameters & your infrastructure specifics, execute the following command:

```bash
root@cse [ ~ ]# chmod 600 $HOME/config.yaml
root@cse [ ~ ]# cse install --config $HOME/config.yaml
```

This step will take a while as it will go through various steps, from checking the rabbitmq connection, to the vCloud Director access + vCenter.

Then it will download both the Photon OS + Ubuntu OVA Templates that will be uploaded + customized to host docker/kubernetes components, and finally added to the CSE catalog as the base templates for the tenants to use when creating a new kubernetes cluster.


Here is the long & boring video showing that process, you can probably skip or watch it to pinpoint some key steps.

<script src="https://asciinema.org/a/KowCJ56CdIiaXvv17EQqfbC8n.js" id="asciicast-KowCJ56CdIiaXvv17EQqfbC8n" async></script>

# Creating a service and starting it up

Finally we are going to create a small script for a systemd service for CSE to ensure it will run automatically at boot time.

```bash
root@cse [ ~ ]# mkdir -vp $HOME/cse
root@cse [ ~ ]# cp $HOME/config.yaml $HOME/cse

root@cse [ ~ ]# cat << EOF > $HOME/cse/cse.sh
#!/usr/bin/env bash

cse run --config /root/cse/config.yaml
EOF
```

In my case RabbitMQ isn't installed on the CSE VM as I use it for other extensions/extensibility purposes, so here is an example of my systemd service

```bash
root@cse [ ~/cse ]# cat << EOF > /etc/systemd/system/cse.service
[Unit]
Description=Container Service Extension for VMware vCloud Director
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/root/cse/cse.sh
Type=simple
User=root
WorkingDirectory=/root/cse
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

Enable and Start the `cse.service`.

```bash
root@cse [ ~/cse ]# systemctl enable cse.service
Created symlink /etc/systemd/system/multi-user.target.wants/cse.service → /etc/systemd/system/cse.service.
root@cse [ ~/cse ]# systemctl start cse.service
root@cse [ ~/cse ]# systemctl status cse.service
● cse.service - Container Service Extension for VMware vCloud Director
   Loaded: loaded (/etc/systemd/system/cse.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2018-08-01 08:25:41 UTC; 52min ago
 Main PID: 1079 (bash)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/cse.service
           ├─1079 bash /root/cse/cse.sh
           └─1080 /usr/bin/python3.6 /usr/bin/cse run --config /root/cse/config.yaml

Aug 01 08:25:42 cse cse.sh[1079]: Python version >= 3.6 (installed: 3.6.5): success
Aug 01 08:25:42 cse cse.sh[1079]: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised.
Aug 01 08:25:42 cse cse.sh[1079]: Connected to AMQP server (rabbitmq.corp.local:5672): success
Aug 01 08:25:42 cse cse.sh[1079]: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised.
Aug 01 08:25:42 cse cse.sh[1079]: Connected to vCloud Director as system administrator (vcd-01a.corp.local:443): success
Aug 01 08:25:42 cse cse.sh[1079]: Connected to vCenter Server vcsa-01a as administrator@corp.local (vcsa-01a.corp.local:443): success
Aug 01 08:25:42 cse cse.sh[1079]: Container Service Extension for vCloud Director running
Aug 01 08:25:42 cse cse.sh[1079]: config file: /root/cse/config.yaml
Aug 01 08:25:42 cse cse.sh[1079]: see file log file for details: cse.log
Aug 01 08:25:42 cse cse.sh[1079]: waiting for requests, press Ctrl+C to finish

```

Hope this helps,

In the next blog post we will cover the **Troubleshooting / Tips & Tricks** part in more detail in case you are facing some issues setting up this part as I've been contacted by many to help setup the environment.
