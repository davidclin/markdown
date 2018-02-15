---
title: How to Set Up VPN between __VENDOR_VPN_GATEWAY__ and Google Cloud VPN
description: Learn how to build site-to-site IPSEC VPN between __VENDOR_VPN_GATEWAY__ and Google Cloud VPN.
author: David Lin , ePlus
tags: Compute Engine, Cloud VPN, __VENDOR_TAGS__
date_published: 2018-02-15
---

This guide walks you through the process to configure the VENDOR_VPN_GATEWAY for
integration with the [Google Cloud VPN Services][cloud_vpn]. This information is
provided as an example only. Please note that this guide is not meant to be a
comprehensive overview of IPsec and assumes basic familiarity with the IPsec
protocol.

[cloud_vpn]: https://cloud.google.com/compute/docs/vpn/overview

## Environment overview

The equipment used in the creation of this guide is as follows:

* Vendor: __VENDOR__
* Model: __MODEL__
* Software Release: __RELEASE__

## Topology

The topology outlined by this guide is a basic site-to-site IPsec VPN tunnel
configuration using the referenced device:

![Topology](images/test.jpg)

## Before you begin

### Overview

The configuration samples which follow will include numerous value substitutions
provided for the purpose of example only. Any references to IP addresses, device
IDs, shared secrets or keys account information or project names should be
replaced with the appropriate values for your environment when following this
guide.

This guide is not meant to be a comprehensive setup overview for the device
referenced, but rather is only intended to assist in the creation of IPsec
connectivity to Google Cloud Platform (GCP) VPC networks. The following is a
high level overview of the configuration process which will be covered:

* Configure the base network configurations to establish L3 connectivity
* Set up the Base VPN configuration, including:
  * Configure IKEv2 Proposal and Policy
  * Configure IKEv2 Keyring
  * Configure IKEv2 profile
  * Configure IPsec Security Association (SA)
  * Configure IPsec transform set
  * Configure IPsec profile
  * Configure IPsec Static Virtual Tunnel Interface (SVTI)
  * Configure Static or Dynamic Routing Protocol to route traffic into the IPsec tunnel
* Testing the IPsec connection
* Advanced VPN configurations

### Getting started

The first step in configuring your __VENDOR_VPN_GATEWAY__ for use with the Google Cloud
VPN service is to ensure that the following prerequisite conditions have been
met:

The __VENDOR_VPN_GATEWAY__ IPsec application requires:

* Advanced Enterprise Services(SLASR1-AES) or Advanced IP Services Technology
  Package License (SLASR1-AIS)
* IPsec RTU license (FLASR1-IPsec-RTU)
* Encryption HW module (ASR1002HX-IPsecHW(=) and ASR1001HX-IPsecW(=)) and Tiered
  Crypto throughput license which applies to ASR1002-HX and ASR1001-HX chassis
  only.

### IPsec parameters

For the __VENDOR_VPN_GATEWAY__ IPsec configuration, the following details will be used:

|Parameter | Value|
--------- |  -----
|IPsec Mode | `Tunnel mode` |
|Auth protocol | `Pre-shared-key` |
|Key Exchange | `IKEv2` |
|Start | `Auto` |
|Perfect Forward Secrecy (PFS) | `Group 16` |
|Dead Peer Detection (DPD) | `60 5 periodic` |

The IPsec configuration used in this guide is specified below:

| Cipher Role | Cipher |
| ------------| -------|
| Encryption | `esp-aes 256 esp-sha-hmac` |
| Integrity | `sha256` |
| Diffie-Hellman (DH) | `group 16` |
| Lifetime | `36,000 seconds (10 hours)` |

## Configuration – GCP
            
### IPsec VPN using static routing

This section provides the steps to create [Cloud VPN on GCP][compute_vpn]. There are two
ways to create VPN on GCP, using Google Cloud Platform Console and the `gcloud`
command-line tool. The upcoming section provide details to both in detail below:

[compute_vpn]: https://cloud.google.com/compute/docs/vpn/overview

#### Using the Google Cloud Platform Console

1.  [Go to the VPN page](https://console.cloud.google.com/networking/vpn/list)
    in the Google Cloud Platform Console.
1.  Click **Create VPN connection**.
1.  Populate the following fields for the gateway:

    * **Name** — The name of the VPN gateway. This name is displayed in the
      console and used by the `gcloud` command-line tool to reference the gateway.
    * **VPC network** — The VPC network containing the instances the VPN gateway
      will serve. In this case it is `vpn-scale-test-cisco`, a
      [custom VPC network](https://cloud.google.com/compute/docs/vpc/using-vpc#create-custom-network).
      Ensure this network does not conflict with your on-premises networks.
    * **Region** — The region where you want to locate the VPN gateway.
      Normally, this is the region that contains the instances you wish to
      reach. Example: `us-east1`
    * **IP address** — Select a pre-existing [static external IP address](https://cloud.google.com/compute/docs/ip-addresses#reservedaddress).
      If you don't have a static external IP address, you can create one by
      clicking **New static IP address** in the pull-down menu. Selected
      `vpn-scale-test0` for this guide.
1.  Populate fields for at least one tunnel:

    * **Peer IP address** — Enter your on-premises public IP address here, with the
      above mentioned topology it is `204.237.220.4`
    * **IKE version** — IKEv2 is preferred, but IKEv1 is supported if that is
      all the peer gateway can manage.
    * **Shared secret** — Used in establishing encryption for that tunnel. You
      must enter the same shared secret into both VPN gateways. If the VPN
      gateway device on the other side of the tunnel doesn't generate one
      automatically, you can make one up.
    * **Remote network IP range** — `10.0.0.0/8`. The range, or ranges, of the
      peer network, which is the network on the other side of the tunnel from
      the Cloud VPN gateway you are currently configuring.
    * **Local subnets** — Specifies which IP ranges will be routed through the
      tunnel. This value cannot be changed after the tunnel is created because
      it is used in the IKE handshake.
    * Select the gateway's entire subnet in the pull-down menu. Or, you can
      leave it blank since the local subnet is the default.
    * Leave **Local IP ranges** blank except for the gateway's subnet.
1.  Click **Create** to create the gateway and initiate all tunnels, though
    tunnels will not connect until you've completed the additional steps below.
    This step automatically creates a network-wide route and necessary
    forwarding rules for the tunnel.
1.  [Configure your firewall rules](https://cloud.google.com/compute/docs/vpn/creating-vpns#configuring_firewall_rules)
    to allow inbound traffic from the peer network subnets, and you must
    configure the peer network firewall to allow inbound traffic from your
    Compute Engine prefixes.

    * Go to the [Firewall rules](https://console.cloud.google.com/networking/firewalls)
      page.
    * Click **Create firewall rule**.
    * Populate the following fields:
      * **Name:** `vpnrule1`
      * **VPC network:** `vpn-scale-test-cisco`
      * **Source filter:** IP ranges.
      * **Source IP ranges:** The peer ranges to accept from the peer VPN
        gateway.
      * **Allowed protocols and ports:** `tcp;udp;icmp`
    * Click **Create**.

#### Using the `gcloud` command-line tool

1.  Create a custom VPC network. You can also use auto VPC network, make sure
    there is no conflict with your local network range.

        gcloud compute networks create vpn-scale-test-cisco --mode custom
        gcloud compute networks subnets create subnet-1 --network vpn-scale-test-cisco \
            --region us-east1 --range 172.16.100.0/24

1.  Create a VPN gateway in the desired region. Normally, this is the region
    that contains the instances you wish to reach. This step creates an
    unconfigured VPN gateway named `vpn-scale-test-cisco-gw-0` in your VPC
    network.

        gcloud compute target-vpn-gateways create vpn-scale-test-cisco-gw-0 \
            --network vpn-scale-test-cisco --region us-east1

1.  Reserve a static IP address in the VPC network and region where you created
    the VPN gateway. Make a note of the created address for use in future steps.

        gcloud compute --project vpn-guide addresses create --region us-east1 vpn-static-ip

1.  Create a forwarding rule that forwards ESP, IKE and NAT-T traffic toward the
    Cloud VPN gateway. Use the static IP address `vpn-static-ip` you reserved
    earlier. This step generates a forwarding rule named `fr-esp`, `fr-udp500`,
    `fr-udp4500` resp.

        gcloud compute --project vpn-guide forwarding-rules create fr-esp --region us-east1 \
            --ip-protocol ESP --address 35.185.3.177 --target-vpn-gateway vpn-scale-test-cisco-gw-0

        gcloud compute forwarding-rules create fr-udp500 --project vpn-guide --region us-east1 \
            --address 104.196.200.68 --target-vpn-gateway vpn-scale-test-cisco-gw-0 --ip-protocol=UDP --ports 500

        gcloud compute forwarding-rules create fr-udp4500 --project vpn-guide --region us-east1 \
            --address 104.196.200.68 --target-vpn-gateway vpn-scale-test-cisco-gw-0 --ip-protocol=UDP --ports 4500

1.  Create a VPN tunnel on the Cloud VPN Gateway that points toward the external
    IP address `[CUST_GW_EXT_IP]` of your peer VPN gateway. You also need to
    supply the shared secret. The default, and preferred, IKE version is 2. If
    you need to set it to 1, use --ike_version 1. The following example sets IKE
    version to 2. After you run this command, resources are allocated for this
    VPN tunnel, but it is not yet passing traffic.

        gcloud compute --project vpn-guide vpn-tunnels create tunnel1 --peer-address 204.237.220.4 \
            --region us-east1 --ike-version 2 --shared-secret MySharedSecret --target-vpn-gateway \
            vpn-scale-test-cisco-gw-0 --local-traffic-selector=172.16.100.0/24

1.  Use a [static route](https://cloud.google.com/sdk/gcloud/reference/compute/routes/create)
    to forward traffic to the destination range of IP addresses
    ([CIDR_DEST_RANGE]) in your local on-premises network. You can repeat this
    command to add multiple ranges to the VPN tunnel. The region must be the
    same as for the tunnel.

        gcloud compute --project vpn-guide routes create route1 --network [NETWORK] --next-hop-vpn-tunnel \
            tunnel1 --next-hop-vpn-tunnel-region us-east1 --destination-range 10.0.0.0/8

1.  Create firewall rules to allow traffic between on-premises network and GCP
    VPC networks.

        gcloud compute --project vpn-guide firewall-rules create vpnrule1 --network vpn-scale-test-cisco \
            --allow tcp,udp,icmp --source-ranges 10.0.0.0/8

## Configuration – __VENDOR_VPN_GATEWAY__

#### Configure IKEv2 proposal and policy

foo

In this block, the following parameters
are set:

* Encryption algorithm - set to `AES-CBC-256`, `AES-CBC-192`, `AES-CBC-128`
* Integrity algorithm - set to SHA256
* Diffie-Hellman group - set to 16


#### Configure IKEv2 keyring

foo


#### Configure IKEv2 profile

foo


#### Configure static routing to route traffic into the IPsec tunnel

Statically route traffic toward the network in the GCP to the Tunnel interface.


### Test result


### Advanced VPN configurations

#### Configure VPN redundancy

![alt_text](https://storage.googleapis.com/gcp-community/tutorials/using-cloud-vpn-with-cisco-asr/GCP-Cisco-ASR-Topology-Redundant.jpg)

Using redundant tunnels ensures continuous availability in the case of a tunnel fails.

If a Cloud VPN tunnel goes down, it restarts automatically. If an entire virtual
device fails, Cloud VPN automatically instantiates a new one with the same
configuration, so you don't need to build two Cloud VPN gateways. The new
gateway and tunnel connect automatically. For hardware appliances such as Cisco
ASR it is recommended that you deploy atleast 2 ASRs and create VPN tunnels to
GCP from each for redundancy purposes.

The VPN redundancy configuration example is built based on the IPsec tunnel and
BGP configuration illustrated above.


###### Static Routing

If you are using static routing then instead of BGP configurations mentioned
above, you can change the metric (higher the metric lower the preference) for
your static route as shown below:

      cisco-asr#ip route 172.16.100.0 255.255.255.0 Tunnel2 10

###### Static Routing (Optional)

When using static routing GCP provides you an option to customize the priority
in case there are multiple routes with the same prefix length. In order to have
symmetric traffic flow make sure that you set the priority of your secondary
tunnel to higher value than the primary tunnel (default priority is 1000). To
define the route priority run the below command.

    gcloud compute --project vpn-guide routes create route2 --network vpn-scale-test-cisco \
        --next-hop-vpn-tunnel tunnel1 --next-hop-vpn-tunnel-region us-east1 --destination-range \
        10.0.0.0/8 --priority=2000


#### Getting higher throughput

As documented in the [GCP Advanced Configurations](https://cloud.google.com/compute/docs/vpn/advanced),
each Cloud VPN tunnel can support up to 3 Gbps when the traffic is traversing a
[direct peering](https://cloud.google.com/interconnect/direct-peering) link, or
1.5 Gbps when traversing the public Internet. To increase the VPN throughput the
recommendation is to add multiple Cloud VPN gateway on the same region to load
balance the traffic across the tunnels. The 2 VPN tunnels configuration example
here is built based on the IPsec tunnel and BGP configuration illustrated above,
can be expanded to more tunnels if required.


## Testing the IPsec connection

The IPsec tunnel can be tested from the router by using ICMP to ping a host on
GCP. Be sure to use the `inside interface` on the ASR 1000.


## References

Please refer to the following documentation for __VENDOR_VPN_GATEWAY__ Platform feature
configuration guide and datasheet:

* [Example Title Goes Here](http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_vpnips/configuration/xe-3s/sec-sec-for-vpns-w-ipsec-xe-3s-book/sec-cfg-vpn-ipsec.html)

Refer to the following documentation for common error messages and debug commands:

* [Example Title Goes Here](http://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/5409-ipsec-debug-00.html)

To learn more about GCP networking, refer to below documents:

* [GCP VPC Networks](https://cloud.google.com/compute/docs/vpc/)
* [GCP Cloud VPN](https://cloud.google.com/compute/docs/vpn/overview)
* [GCP advanced VPN](https://cloud.google.com/compute/docs/vpn/advanced)
* [Troubleshooting VPN on GCP](https://cloud.google.com/compute/docs/vpn/troubleshooting)
