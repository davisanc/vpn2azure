## Site-to-Site VPN between pfSense and Azure with BGP to allow dynamic discovery of your networks

This post explains how to set up a VPN connection from an open-source pfSense Firewall to Azure. We will use BGP running on top of the VPN IPSEC tunnel to enable our local network and Azure to dynamically exchange routes. This removes the burden of having to declare manually on your VPN gateways which subnets you want to advertise to the other end

First thing to bear in mind is that you cannot have overlapping IP address between your LAN side on the Firewall and the VNET address space. My home router sits on a 192.168.0.0/24 and the pfSense is connected to the home router, using the pfSense WAN port. The Firewall has a LAN address space on 192.168.1.0.24 and has a PC connected to the LAN port of the Firewall

| Parameters to fill  | Values |
| --- | --- | 
| My Home Router Public IP  | 1.2.3.4 |
| LAN subnet behind pfSense (Local VPN Gateway) | 192.168.1.0/24 |
| Azure VNET Address Space  | 10.11.0.0/16  |
| Azure VNET VM Subnet  | 10.11.0.0/24  |
| Azure VNET Gateway Subnet   | 10.11.3.0/24  |
| Azure VPN Gateway Public IP  |  23.97.137.42 |
| Azure VPN Type  | Route-Based |
| Azure VPN BGP ASN  | 65515 |
| Azure Gateway Type  | VPN |
| Azure Local Network Gateway Name  | LocalVPN-pfSense  |
| Azure Local Network Gateway BGP peer address  | 192.168.1.1  |
| Azure Local Network Gateway BGP ASN| 65501 |
| Azure VPN Connection Name  | VPN-conn2pfSense  |
| Azure VPN Shared Key  | mySuperSecretKey123 |

We will start creating a Virtual Network (again make sure the address space you enter doesn't overlap with the space on your local network)

![image of vnet](/images/vnet-azure.PNG)

Followed by the gateway subnet (I decided to use /24 to keep the same subnetting scheme but the recommendation from Microsoft is to use a /27 or /28 for the gateway subnet)

![image_of_gwsubnet](/images/gw-subnet.PNG)

Next, we will create the Virtual Network Gateway. We will chose to create a new public IP address. Also, we will use BGP to exchange routes between Azure and the pfSense firewall, so we need to mark the BGP option when creating the Gateway. We will use a private BGP ASN of 65515

![image_of_vpn-gw](/images/vpn-gw.PNG)

You will find the BGP peer address on your VPN Gateway. This is the local address that BGP will use in your Azure VPN Gateway to initiate a BGP connection to your home gateway

![image_of_bgppeer](/images/bgp-peer.PNG)

Now we are going to create the Local Network Gateway. Azure refers to the VPN device that sits in your home network. You will need to indicate the BGP peer address, your local network behind the Firewall (or local VPN gateway) and a Private BGP ASN (I am using 65501)

![image_of_local-gw](/images/local-gw.PNG)

Once the local gateway is created we will define a connection to our home VPN Gateway. We will use a private shared key to enable the IPSEC VPN to come up. Remember to mark BGP to 'enabled' on your Connection. This is how it looks like when the connection is up and running (assuming at this poit have done the similar on the other end)

![image_of_connection](/images/connection.PNG)

Now, moving to the other end we will use the Web UI on the pfSense firewall to work on the Rules and VPN settings
To configure a new tunnel, a new Phase 1 IPSEC VPN must be created. Remote Gateway will be the public IP address assigned to my Virtual Network Gateway in Azure. Leave 'auto' as IKE key exchange version, selecting WAN as the interface to run the VPN. For the authentication part, use the Pre-Shared Key you have defined. Use the encryption algorithm you need, in my case AES (256 bits), DH group and Hashing algorithm

![image_of_phase1](/images/phase1.png)

We will then move to Phase 2. This phase is what builds the actual tunnel, sets the protocol to use, and sets the length of time to keep the tunnel up when there is no traffic. For remote network, use the VNET address space. Local subnet will the address space on the LAN side of the pFsense

![image_of_phase2](/images/phase2.png)

Apply changes and go to IPSEC Status 

![image_of_ipsec-status](/images/ipsec-status.png)

You will need to create a rule to permit IPSEC traffic coming through your WAN interface

I have also open TCP port 179 on a rule on the IPSEC interface to permit incoming BGP connections from Azure

![image_of_ipsec-rule](/images/ipsec-rule.png)

Now, in order to use BGP on pfSense you will need to install OpenGPD through the Packet Manager
We will use BGP peer groups to define the BGP ASN of the Azure peer

![image_of_bgp-group](/images/bgp-group.png)

With BGP, you only need to declare a minimum prefix to a specific BGP peer over the IPsec S2S VPN tunnel. It can be as small as a host prefix (/32) of the BGP peer IP address of your on-premises VPN device. The point of using BGP over VPN is that you can control dynamically which on-premises network prefixes you want to advertise to Azure to allow your Azure Virtual Network to access

My BGP settings are the following:

![image_of_bgp-settings](/images/bgp-settings.png)

BGP neighbor will be the IP address of the Virtual Gateway on Azure, in my case with IP address 10.11.3.254

![image_of_bgp-neighbor](/images/bgp-neighbor.png)

You can also visualize the whole BGP raw config in pfSense

![image_of_bgp-rawconfig](/images/bgp-rawconfig.png)

Finally, you will be able to see the BGP session coming up after a few minutes

![image_of_bgp-status1](/images/bgp-status1.png)

![image_of_bgp-status2](/images/bgp-status2.png)

To test this, you can simply ping from a computer on the LAN side of the pfSense (192.168.1.0/24) to a VM in Azure on the VNET address space (10.11.0.0/16), and that should work! :)

![image_of_ping](/images/ping.png)

And from the Azure side to a host behind pfSense

![image_of_ping](/images/pingfromazuretopfnse.png)
