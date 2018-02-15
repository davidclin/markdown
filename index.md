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

![Topology](./images/test.jpg = 10x)

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

### IPsec VPN using dynamic routing

For dynamic routing you use [Cloud Router](https://cloud.google.com/router/docs/concepts/overview)
to establish BGP sessions between the 2 peers.

#### Using the Cloud Platform Console

1.  [Go to the VPN page](https://console.cloud.google.com/networking/vpn/list)
    in the Google Cloud Platform Console.
1.  Click **Create VPN connection**.
1.  Populate the following fields for the gateway:

    * **Name** — The name of the VPN gateway. This name is displayed in the
      console and used in by the `gcloud` command-line tool to reference the gateway.
    * **VPC network** — The VPC network containing the instances the VPN gateway
      will serve. In this case it is `vpn-scale-test-cisco`, a
      [custom VPC network](https://cloud.google.com/compute/docs/vpc/using-vpc#create-custom-network).
    * **Region** — The region where you want to locate the VPN gateway.
      Normally, this is the region that contains the instances you wish to
      reach. Example: `us-east1`
    * **IP address** — Select a pre-existing [static external IP address](https://cloud.google.com/compute/docs/ip-addresses#reservedaddress).
      If you don't have a static external IP address, you can create one by
      clicking **New static IP address** in the pull-down menu. Selected
      `vpn-scale-test0` for this guide.

1.  Populate fields for at least one tunnel:

    * **Peer IP address** — `204.237.220.4` Public IP address of the peer
      gateway.
    * **IKE version** — IKEv2 is preferred, but IKEv1 is supported if that is
      all the peer gateway can manage.
    * **Shared Secret** — Character string used in establishing encryption for
      that tunnel. You must enter the same shared secret into both VPN gateways.
      If the VPN gateway device on the peer side of the tunnel doesn't generate
      one automatically, you can make one up.
    * **Routing options** — Select **Dynamic (BGP)**.
    * **Cloud router** — Select **Create cloud router**, then populate the
      following fields. When you are done, click **Save and continue**.
    * **Name** — The name of the Cloud Router. This name is displayed in the
      console and used by the `gcloud` command-line tool to reference the
      router. Example: `vpn-scale-test-cisco-rtr`
    * **Google ASN** — The [private ASN](https://tools.ietf.org/html/rfc6996)
      (64512 - 65534, 4200000000 - 4294967294) for the router you are
      configuring. It can be any private ASN you are not already using.
      Example: `65002`
    * **BGP session** — Click the pencil icon, then populate the following
      fields. When you are done, click **Save and continue**.
    * **Name** — `bgp-peer1`
    * **Peer ASN** — The [private ASN](https://tools.ietf.org/html/rfc6996)
      (64512 - 65534, 4200000000 - 4294967294) for the router you are
      configuring. It can be any private ASN you are not already using.
      Example: `65001`
    * **Google BGP IP address** — The two BGP interface IP addresses must be
      *link-local* IP addresses belonging to the same /30 subnet in
      `169.254.0.0/16`. Example: `169.254.1.1`
    * **Peer BGP IP address** — See explanation for **Google BGP IP address**.
      Example: `169.254.1.2`

1.  Click **Create** to create the gateway, Cloud Router, and all tunnels,
    though tunnels will not connect until you've configured the peer router as
    well.

    This step automatically creates the necessary forwarding rules for the
    gateway and tunnels.

1.  [Configure your firewall rules](https://cloud.google.com/compute/docs/vpn/creating-vpns#configuring_firewall_rules)
    to allow inbound traffic from the peer network subnets, and you must
    configure the peer network firewall to allow inbound traffic from your
    Compute Engine prefixes.

    * Go to the [Firewall rules](https://console.cloud.google.com/networking/firewalls)
      page.
    * Click **Create firewall rule**.
    * Populate the following fields:
      * **Name:** `vpnrule1`
      * **VPC network:** `my-network`
      * **Source filter:** IP ranges.
      * **Source IP ranges:** The peer ranges to accept from the peer VPN gateway.
      * **Allowed protocols and ports:** tcp;udp;icmp
    * Click **Create**.

#### Using the `gcloud` command-line tool

1.  Create a custom VPC network. You can also use auto VPC network, make sure
    there is no conflict with your local network range.

        gcloud compute networks create vpn-scale-test-cisco --mode custom

        gcloud compute networks subnets create subnet-1 --network vpn-scale-test-cisco \
            --region us-east1 --range 172.16.100.0/24

1.  Create a VPN gateway in the desired region. Normally, this is the region
    that contains the instances you want to reach. This step creates an
    unconfigured VPN gateway named `vpn-scale-test-cisco-gw-0` in your VPC
    network.

        gcloud compute target-vpn-gateways create vpn-scale-test-cisco-gw-0 --network \
            vpn-scale-test-cisco --region us-east1

1.  Reserve a static IP address in the VPC network and region where you created
    the VPN gateway. Make a note of the created address for use in future steps.

        gcloud compute --project vpn-guide addresses create --region us-east1 vpn-static-ip

1.  Create a forwarding rule that forwards ESP, IKE and NAT-T traffic toward the
    Cloud VPN gateway. Use the static IP address `vpn-static-ip` you reserved
    earlier. This step generates a forwarding rule named `fr-esp`, `fr-udp500`,
    `fr-udp4500` resp.

        gcloud compute --project vpn-guide forwarding-rules create fr-esp  --region us-east1 \
            --ip-protocol ESP --address 35.185.3.177 --target-vpn-gateway vpn-scale-test-cisco-gw-0

        gcloud compute --project vpn-guide forwarding-rules create fr-udp500 --region us-east1 \
            --ip-protocol UDP --ports 500 --address 35.185.3.177 --target-vpn-gateway vpn-scale-test-cisco-gw-0

        gcloud compute --project vpn-guide forwarding-rules create fr-udp4500 --region us-east1 \
            --ip-protocol UDP --ports 4500 --address 35.185.3.177 --target-vpn-gateway vpn-scale-test-cisco-gw-0

1.  Create [Cloud Router](https://cloud.google.com/compute/docs/cloudrouter) as
    shown below:

        gcloud compute --project vpn-guide routers create vpn-scale-test-cisco-rtr --region us-east1 \
            --network vpn-scale-test-cisco --asn 65002

1.  Create a VPN tunnel on the Cloud VPN Gateway that points toward the external
    IP address `[CUST_GW_EXT_IP]` of your peer VPN gateway. You also need to
    supply the shared secret. The default, and preferred, IKE version is 2. If
    you need to set it to 1, use --ike_version 1. The following example sets IKE
    version to 2. After you run this command, resources are allocated for this
    VPN tunnel, but it is not yet passing traffic.

        gcloud compute --project vpn-guide vpn-tunnels create tunnel1 --peer-address 204.237.220.4 \
            --region us-east1 --ike-version 2 --shared-secret MySharedSecret --target-vpn-gateway \
            vpn-scale-test-cisco-gw-0 --router vpn-scale-test-cisco-rtr

1.  Update the Cloud Router config to add a virtual interface (--interface-name)
    for the BGP peer. The BGP interface IP address must be a *link-local* IP
    address belonging to the IP address range `169.254.0.0/16` and it must
    belong to same subnet as the interface address of the peer router. The
    netmask length is recommended to be 30. Make sure each tunnel has a unique
    pair of IPs. Alternatively, you can leave `--ip-address` and `--mask-length`
    blank, and leave `--peer-ip-address` blank in the next step, and IP
    addresses will be automatically generated for you.

        gcloud compute --project vpn-guide routers add-interface vpn-scale-test-cisco-rtr \
            --interface-name if-1 --ip-address 169.254.1.1 --mask-length 30 --vpn-tunnel tunnel1 --region us-east1

1.  Update the Cloud Router config to add the BGP peer to the interface. This
    example uses ASN 65001 for the peer ASN. You can use your public ASN or
    [private ASN](https://tools.ietf.org/html/rfc6996)
    (64512 - 65534, 4200000000 - 4294967294) that you are not already using in
    the peer network. The BGP peer interface IP address must be a *link-local*
    IP address belonging to the IP address range `169.254.0.0/16`. It must
    belong to same subnet as the GCP-side interface. Make sure each tunnel has a
    unique pair of IPs.

        gcloud compute --project vpn-guide routers add-bgp-peer vpn-scale-test-cisco-rtr --peer-name \
            bgp-peer1 --interface if-1 --peer-ip-address 169.254.1.2 --peer-asn 65001 --region us-east1

1.  View details of the Cloud Router and confirm your settings.

        gcloud compute --project vpn-guide routers describe vpn-scale-test-cisco-rtr --region us-east1

1.  Create firewall rules to allow traffic between on-prem network and GCP VPC networks.

        gcloud  compute --project vpn-guide firewall-rules create vpnrule1 --network vpn-scale-test-cisco \
            --allow tcp,udp,icmp --source-ranges 10.0.0.0/8
            
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

## Configuration – Cisco ASR 1000

### Base network configurations (to establish L3 connectivity)

This section provides the base network configuration of Cisco ASR 1000 to
establish network connectivity. At least one internal facing interface is
required to connect to your own network, and one external facing interface is
required to connect to GCP. A sample interface configuration is provided below
for reference:

    ! Internal interface configuration
    interface TenGigabitEthernet0/0/2
    description internal facing interface
    ip address 10.0.200.1 255.0.0.0
    !
    !External interface configuration
    interface TenGigabitEthernet0/0/0
    description external facing interface
    ip address 204.237.220.4 255.255.255.224

### Base VPN configurations

#### Configure IKEv2 proposal and policy

Create an Internet Key Exchange (IKE) version 2 proposal object. IKEv2 proposal
objects contain the parameters required for creating IKEv2 proposals when
defining remote access and site-to-site VPN policies. IKE is used to
authenticate IPsec peers, negotiate and distribute IPsec encryption keys, and
automatically establish IPsec security associations (SAs). The default proposal
associated with the default policy is used for negotiation. An IKEv2 policy with
no proposal is considered incomplete. In this block, the following parameters
are set:

* Encryption algorithm - set to `AES-CBC-256`, `AES-CBC-192`, `AES-CBC-128`
* Integrity algorithm - set to SHA256
* Diffie-Hellman group - set to 16

      crypto ikev2 proposal VPN_SCALE_TEST_IKEV2_PROPOSAL
      encryption aes-cbc-256 aes-cbc-192 aes-cbc-128
      integrity sha256
      group 16
      !         
      crypto ikev2 policy VPN_SCALE_TEST_IKEV2_POLICY
      proposal VPN_SCALE_TEST_IKEV2_PROPOSAL

#### Configure IKEv2 keyring

The IKEv2 keyring is associated with an IKEv2 profile and hence, caters to a set
of peers that match the IKEv2 profile.

    crypto ikev2 keyring VPN_SCALE_TEST_KEY
     peer GCP1
      address 104.196.200.68
      pre-shared-key MySharedSecret
    !

#### Configure IKEv2 profile

An IKEv2 profile must be configured and must be attached to an IPsec profile on
both the IKEv2 initiator and responder. In this block, the following parameters
are set:

* IKEv2 Lifetime - set the lifetime of the security associations (after which a
  reconnection will occur). Set to 36,000 seconds as recommended configuration
  on ASR 1000 router.
* DPD – set the dead peer detection interval and retry interval, if there are no
  response from the peer, the SA created for that peer is deleted. Set to 60
  seconds keepalive interval and 5 seconds retry interval as recommended
  configuration on ASR 1000 router.

      crypto ikev2 profile VPN_SCALE_TEST_IKEV2_PROFILE
       match address local interface TenGigabitEthernet0/0/0
       match identity remote any
       authentication local pre-share
       authentication remote pre-share
       keyring local VPN_SCALE_TEST_KEY
       lifetime 36000
       dpd 60 5 periodic
      !

#### Configure IPsec security association

Create IPsec security-association (SA) rules. A security association is a
relationship between two or more entities that describes how the entities will
use security services to communicate securely. During tunnel establishment, the
two peers negotiate security associations that govern authentication,
encryption, encapsulation, and key management. These negotiations involve two
phases: first, to establish the tunnel (the IKE SA) and second, to govern
traffic within the tunnel (the IPsec SA). The following commands set the SA
lifetime and timing parameters.

* `IPsec SA lifetime` – 1 hour is the recommended value on ASR 1000 router.

* `IPsec SA replay window-size` – 1024 is the recommended value on ASR 1000 router.

      crypto ipsec security-association lifetime seconds 3600
      crypto ipsec security-association replay window-size 1024

#### Configure IPsec transform set

A transform set represents a certain combination of security protocols and
algorithms. During the IPsec SA negotiation, the peers agree to use a particular
transform set for protecting a particular data flow.

    crypto ipsec transform-set VPN_SCALE_TEST_TS esp-aes 256 esp-sha-hmac mode tunnel

#### Configure IPsec profile

Defines the IPsec parameters that are to be used for IPsec encryption between
two IPsec routers in IPsec profile configuration. In this block, the following
parameters are set

* Perfect Forward Secrecy (PFS) - PFS ensures that the same key will not be
  generated again, so forces a new diffie-hellman key exchange. Set to group16
  as recommended configuration on ASR 1000 router.
* SA Lifetime - set the lifetime of the security associations (after which a
  reconnection will occur). Set to `3600 seconds` as recommended configuration
  on ASR 1000 router.

      crypto ipsec profile VPN_SCALE_TEST_VTI
       set security-association lifetime seconds 3600
       set transform-set VPN_SCALE_TEST_TS
       set pfs group16
       set ikev2-profile VPN_SCALE_TEST_IKEV2_PROFILE

#### Configure IPsec static virtual tunnel interface (SVTI)

A tunnel interface is configured to be the logical interface associated with the
tunnel. All traffic routed to the tunnel interface will be encrypted and
transmitted to the GCP. Similarly, traffic from the GCP will be logically
received on this interface.

Association with the IPsec security association is done through the
`tunnel protection` command.

Adjust the maximum segment size (MSS) value of TCP packets going through a
router. The recommended value is 1360 when the number of IP MTU bytes is set to
1400. With these recommended settings, TCP sessions quickly scale back to
1400-byte IP packets so the packets will "fit" in the tunnel.

    !
    interface Tunnel1
     ip address 169.254.0.58 255.255.255.252
     ip mtu 1400
     ip tcp adjust-mss 1360
     tunnel source TenGigabitEthernet0/0/0
     tunnel mode ipsec ipv4
     tunnel destination 104.196.200.68
     tunnel protection ipsec profile VPN_SCALE_TEST_VTI
    !

#### Configure static or dynamic routing protocol to route traffic into the IPsec tunnel

BGP is used within the tunnel to exchange prefixes between the GCP and the ASR
1000 router. The GCP will announce the prefix corresponding to your Cloud.

BGP timers are adjusted to provide more rapid detection of outages.

To advertise additional prefixes to GCP, copy the "network" statement and
identify the prefix you wish to advertise. Make sure the prefix is present in
the routing table of the ASR 1000 with a valid next-hop.

    router bgp 65001
     bgp log-neighbor-changes
     neighbor 169.254.0.1 description BGP session over Tunnel1
     neighbor 169.254.0.1 remote-as 65002
     neighbor 169.254.0.1 timers 20 60 60
    !
    address-family ipv4
     network 10.0.0.0
     neighbor 169.254.0.1 activate
    exit-address-family

or

Statically route traffic toward the network in the GCP to the Tunnel interface.

    ip route 172.16.100.0 255.255.255.0 Tunnel 1

Check [Best practices](https://cloud.google.com/router/docs/resources/best-practices) 
for further recommendations on peer configurations.

### Saving the configuration

To save the running configuration and set it as the default startup, run the
following command on Cisco IOS terminal:

    copy run start

### Test result

    cisco-asr#ping 172.16.100.2 source 10.0.200.1
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.100.2, timeout is 2 seconds:
    Packet sent with a source address of 10.0.200.1
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 18/19/20 ms

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

    cisco-asr#ping 172.16.100.2 source 10.0.200.1
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.100.2, timeout is 2 seconds:
    Packet sent with a source address of 10.0.200.1
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 18/19/20 ms


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
