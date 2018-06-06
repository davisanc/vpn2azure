## pfSense VPN to Azure with BGP to allow dynamic discovery of your networks

This post explains how to set up a VPN connection from an open-source pfSense Firewall to Azure. We will use BGP running on top of the VPN IPSEC tunnel to enable our local network and Azure to dynamically exchange routes. This removes the burden of having to declare manually on your VPN gateways which subnets you want to advertise to the other end

First thing to bear in mind is that you cannot have overlapping IP address between your LAN side on the Firewall and the VNET address space. My home router sits on a 192.168.0.0/24 and the pfSense is connected to the home router, using the pfSense WAN port. The Firewall has a LAN address space on 192.168.1.0.24 and has a PC connected to the LAN port of the Firewall

| Parameters to fill  | Values |
| --- | --- | 
| My Home Router Public IP  | 1.2.3.4 |
| LAN subnet behind pfSense (Local VPN Gateway) | 192.168.1.0/24 |
| Azure VNET Address Space  | 10.11.0.0/16  |
| Azure VNET VM Subnet  | 10.11.0.0/24  |
| Azure VNET Gateway Subnet   | 10.11.3.0/24  |
| Azure VPN Gateway Public IP  |   |
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
