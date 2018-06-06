## pfSense VPN to Azure with BGP to allow dynamic discovery of your networks

This post explains how to set up a VPN connection from an open-source pfSense Firewall to Azure. We will use BGP running on top of the VPN IPSEC tunnel to enable our local network and Azure to dynamically exchange routes. This removes the burden of having to declare manually on your VPN gateways which subnets you want to advertise to the other end

First thing to bear in mind is that you cannot have overlapping IP address between your LAN side on the Firewall and the VNET address space. My home router sits on a 192.168.0.0/24 and the pfSense is connected to the home router, using the pfSense WAN port. The Firewall has a LAN address space on 192.168.1.0.24 and has a PC connected to the LAN port of the Firewall

| Parameters to fill  | Values |
| --- | --- | 
| My Home Router Public IP  | 1.2.3.4 |
| LAN subnet behind pfSense  | 192.168.0.0/24 |
| Azure VNET Address Space  | 10.11.0.0/16  |
| Azure VNET VM Subnet  | 10.11.0.0/24  |
| Azure VNET Gateway Subnet   | 10.11.3.0/24  |
| Azure VPN Gateway Public IP  |   |
| Azure VPN Type  | Route-Based |
| Azure Gateway Type  | VPN |
| Azure Local Network Gateway Name  | LocalVPN-pfSense  |
| Azure VPN Connection Name  | VPN-conn2pfSense  |
| Azure VPN Shared Key  | mySuperSecretKey123 |

We will start creating a Virtual Network (again make sure the address space you enter doesn't overlap with the space on your local network), followed by the gateway subnet (I decided to use /24 to keep the same subnetting scheme but the recommendation from Microsoft is to use a /27 or /28 for the gateway subnet).

![image of vnet](/images/create_vnet.PNG)

Next, we will create the Virtual Network Gateway

