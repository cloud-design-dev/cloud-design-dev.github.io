---
layout: default
title: Site to Site IPsec Tunnel to IBM Cloud VPC VPNaaS
parent: IBM Cloud VPC
nav_order: 1
---

## Overview
This article will walk you through the process of connecting on on-prem IPsec tunnel to the [IBM Cloud VPC VPN]() as a service offering. This will allow you to communicate from your local machine to the private IP addresses assigned to your VPC compute instances. In this guide you will walk through the following steps: 

 - Provisioning an instance of  VPC VPN as a Service 
 - Adding a Peer connection to VPC VPN to connect to your local network
 - Install strongSwan on a local machine/VM
 - Configuring the local IPsec peer
 - Bringing up the local IPsec tunnel and pinging VPC resources

### Provision and configure the VPC VPNaaS
We’ll start by deploying an instance of VPNaaS. From the main [VPC landing page](https://cloud.ibm.com/vpc-ext/overview) click on VPN Gateways on the left hand navigation bar:

![Go to VPN Overview page](https://dsc.cloud/quickshare/vpc-vpn-gateway.png)

Make sure you select the region where your VPC resides and then click Create VPN. 

![Create VPN Step 1](https://dsc.cloud/quickshare/create-vpn-step1.png)

At the top of the screen give the VPN a name, select the VPC where you would like the VPN deployed, and then select the subnet to use with the VPN. **Note**: Only the resources in the same zone as the subnet you choose can connect through this VPN gateway.

![Create VPN Step 2](https://dsc.cloud/quickshare/vpn-gateway-name.png)

With the subnet selected scroll down and make sure the **New VPN Connection for VPC** option is enabled. Give the new connection a name and provide the local and peer subnets along with the pre-shared key. 

**Note**: If you need to generate a pre-shared key, launch Cloud Shell by clicking the terminal icon in the upper right of the IBM Cloud portal. 
![Launch Cloud Shell](https://dsc.cloud/quickshare/Shared-Image-2020-09-23-09-26-23.png)

Once your Cloud Shell session starts run the following command to generate a 32 character pre-shared key 

```shell
tr -dc "[:alpha:][:alnum:]" < /dev/urandom | head -c 32
```

 In this example my VPC utilizes the 10.240.0.0/24 subnet and my local network uses 172.16 IPs. The *Peer gateway address* is the public IP on your local network. If you are unsure of what this is pull up a browser and head to [IP Chicken](https://www.ipchicken.com/). 
![Add connection information](https://dsc.cloud/quickshare/add-vpn-connection.png)

With all the details added click *Create VPN Gateway* in the right hand navigation bar to deploy the VPN.  We'll give the new VPN a few moments to deploy and then copy down the Peer address that we'll need for the local tunnel configuration. 

![Copy down VPN Peer Address](https://dsc.cloud/quickshare/vpc-vpn-peer-address.png)


### Install strongSwan on local machine
In my example I have a local Ubuntu 18 VM that I will be using as the local IPsec peer. The first step is to install [strongSwan](https://www.strongswan.org/). 

```shell 
$ apt-get update && apt-get install strongswan -y 
```

With strongSwan installed we need to add IP forwarding to our kernel parameters:

```shell
$ sudo cat >> /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1 
net.ipv4.conf.all.accept_redirects = 0 
net.ipv4.conf.all.send_redirects = 0
EOF

$ sysctl -p /etc/sysctl.conf
```

### Configure local ipsec peer

Next we'll update the `/etc/ipsec.secrets` file. The syntax for the file is:

```
LOCAL_PEER_IP VPC_VPN_PEER_IP : PSK "Pre-shared key"
```

For example if your local public IP was 192.168.20.2, the VPC VPN Peer address was 192.168.30.5, and the pre-shared key was `XtemrMYFfmmMCpxgdCwSYoRBKdjQ1ndb` the file would look like this:

```
192.168.20.2 192.168.30.5 : PSK "XtemrMYFfmmMCpxgdCwSYoRBKdjQ1ndb"
```

With the secrets file updated we'll now move on to updating the strongSwan configuration file:

```
# ipsec.conf - strongSwan IPsec configuration file
# basic configuration
config setup
    # strictcrlpolicy=yes
    # uniqueids = no
        charondebug="all"
        uniqueids=yes
        strictcrlpolicy=no

# connection to us-east-vpc
conn home-to-vpc
  authby=secret
  left=%defaultroute
  leftid=<Local Server Public IP>
  leftsubnet=<Local Internal Subnet range>
  right=<VPC VPN Endpoint IP>
  rightsubnet=<VPC Subnet range>,166.8.0.0/14,161.26.0.0/16
  ike=aes256-sha2_256-modp1024!
  esp=aes256-sha2_256!
  keyingtries=0
  ikelifetime=1h
  lifetime=8h
  dpddelay=30
  dpdtimeout=120
  dpdaction=restart
  auto=start
```

We add in the ranges `166.8.0.0/14` and `161.26.0.0/16` so that we can communicate with IBM Cloud services over their private IP address space.

With the ipsec configuration updated, add an iptables rule for post-routing. Again for my tunnel the VPC Subnet is 10.240.0.0/24 and my local internal subnet is 172.16.0.0/24 so adjust the following command to meet your needs.

```
$ iptables -t nat -A POSTROUTING -s 10.240.0.0/24 -d 172.16.0.0/24 -J MASQUERADE
```

Now restart the ipsec service and check the status of the tunnel:

```
$ sudo ipsec restart
Stopping strongSwan IPsec...
Starting strongSwan 5.6.2 IPsec [starter]...

$ sudo ipsec status
Security Associations (1 up, 0 connecting):
 home-to-vpc[1]: ESTABLISHED 16 seconds ago, 10.0.0.67[x.x.x.x]...52.y.y.y[52.y.y.y]
 home-to-vpc{1}:  INSTALLED, TUNNEL, reqid 1, ESP in UDP SPIs: c2225f47_i ccc3d826_o
 home-to-vpc{1}:   10.0.0.0/18 === 161.26.0.0/16 166.8.0.0/14 192.168.0.0/18
```

### Test connectivity to VPC instance
In my VPC I have an instance with a private IP of 10.240.0.6:

```
$ ibmcloud is instances --output json | jq -r '.[] | select(.vpc.name=="us-south-vpc-rt") | .network_interfaces[].primary_ipv4_address'
10.240.0.6

$ ping -c2 -q 10.240.0.6
PING 10.240.0.6 (10.240.0.6) 56(84) bytes of data.

--- 10.240.0.6 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 40.836/41.497/42.159/0.692 ms
```

