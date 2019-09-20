
Azure ExpressRoute is an Enterprise-class network platform that provides you with a scalable, high-performance solution for connecting your private network to Microsoft’s network.  You can connect to Microsoft directly using ExpressRoute Direct, or you can connect to Microsoft through a mature ecosystem of partners using ExpressRoute.  You can configure up to two different network peering types to allow an ExpressRoute circuit to connect to private workloads running in your Azure VNets and to public workloads available on the Microsoft Network.
This deployment guide is focused on helping you deploy and optimize the Azure private peering, which enables connectivity between your private network and your Azure VNets over ExpressRoute.  First, we will review the fundamentals of ExpressRoute and study the key components and steps involved in creating your first ExpressRoute circuit.  Next, we will discuss the value of the VNet-based “hub and spoke” architecture and explore how route advertisement works when you connect ExpressRoute to this footprint.  Finally, we’ll learn how the Azure VPN Gateway can be added to provide an extra layer of path redundancy. 

# ExpressRoute Fundamentals

Azure ExpressRoute, as an end-to-end solution, is comprised of several key components. Some are physical, and some are logical. Some are managed by Microsoft, and some by you. The key components of the ExpressRoute platform are listed below. If you are comfortable with configuring and initializing an ExpressRoute circuit, feel free to skip this section.

### 1.	The first component is a pair of physical Enterprise-class routers, physically located in over 50 strategic peering locations and network exchanges worldwide 
a.	These “Microsoft Enterprise Edge” Routers (MSEEs henceforth) represent the physical boundary, or provider edge, of the MSFT private backbone for ExpressRoute customers.  
b.	This pair of routers is provided as part of a high-availability architecture with a printed 99.95% SLA.
c.	If you are an ExpressRoute Direct customer, your first step is to reserve a set of physical ports on the MSEE pair for your service. This begins with an LOA process directly with Microsoft Azure. 
d.	You will then need to arrange for the peering location staff to connect a pair of short haul, single mode fibers (cross-connects) between your router ports and the MSEE router ports. ExpressRoute Direct supports both 10 Gbps fibers and 100 Gbps fibers. 
e.	If you prefer to provision ExpressRoute through a partner, you will be provided with dedicated bandwidth over a set of cross-connects managed by the partner and Microsoft. No LOA process with Microsoft is required. Instead, you should initiate your ExpressRoute order with the partner first. This is outlined in the ExpressRoute workflows for circuit provisioning solution. 
 
### 2.	The second component is an ExpressRoute circuit, which you create and configure by using the Azure Portal, Azure PowerShell, or the Azure Command Line
a.	An ExpressRoute circuit is a logical, scalable, point-to-point connection between your private network and the Microsoft network.
b.	ExpressRoute circuits are comprised of key-value pairs that define and characterize their overall capabilities, such as throughput, connectivity range, data egress metering, etc. 
c.	ExpressRoute circuit SKUs differ between partner-based ExpressRoute and ExpressRoute Direct. 
d.	In the peering location, the ExpressRoute circuit begins as set of tagged VLANs running over the cross-connects between the MSEE routers and the customer or partner routers.
e.	If you are an ExpressRoute Direct customer, you can choose either a single 802.1Q tag or a double 802.1Q tag (QnQ).  Else, you will need to work with the ExpressRoute partner to determine which kind of VLAN tagging you would like to receive.
f.	A completed circuit will show a status of “Enabled”, and the billing cycle will begin. 
g.	For provider ExpressRoute, the circuit will stay in a “not provisioned” state until you pass them your service key and they complete their logial paring. The circuit will then transition to a “provisioned” state. This represents a successful connection at layer 1 and layer 2. 

### 3.	The third component is an ExpressRoute peering, which is how you configure BGP over your ExpressRoute circuit using the Azure Portal, Azure Powershell, or the Azure Command Line 
a.	There are fundamental differences between the Azure private peering and the Microsoft peering in terms of capabilities and limitations.
b.	This guide is focused on helping you deploy the Azure private peering. This solution will help you complete the Azure private peering which will allow you to connect, or link, one or more VNets your circuit.
c.	When one or more peerings are complete, and BGP is in an established state over both cross-connects, the provider status will move to an “established” state. This represents a successful layer 3 connection between the BGP neighbors.
d.	Remember that for each peering type, you will always have two sets of BGP neighbors, one set on the primary fiber, and one set on the secondary fiber.  Both need to be established for the ExpressRoute circuit to function correctly.
e.	In the provider model, the BGP neighbors opposite of the MSEEs can be maintained on your own edge routers, or by the provider, on edge routers they manage for you. With ExpressRoute Direct, the BGP neighbors will reside in routers that you provision and maintain within the chosen peering location. 
f.	The MSEE router pairs support active-active BGP neighbors, active-passive BGP neighbors, and bi-directional forwarding detection (BFD).
g.	There is a third type of peering, called the Azure public peering, which is a legacy feature that has been deprecated.  All the features of the Azure public peering were improved and merged into the Microsoft peering. 

### 4.	The fourth component is a group of organized routes to advertise from Microsoft to your private network. 
a.	For the Microsoft peering, these organized routes are configured by you by utilizing route filters.  This topic, and the Microsoft peering as a whole, will be discussed in further detail in a companion deployment guide. 
b.	For the Azure private peering, these organized routes arrive in the form of connected VNets.  The routes that are advertised from Azure over the private peering will be the direct result of two factors:
i.	The network address blocks that you have chosen to assign to a VNet
ii.	The VNets that you have chosen to connect to your ExpressRoute circuit
c.	To connect (or link) a VNet to your ExpressRoute circuit, you must first deploy an ExpressRoute gateway in a special managed subnet called the GatewaySubnet. 
d.	The connection is created between your ExpressRoute gateway and your circuit by configuring the connection object within Azure. 
e.	The connection will complete the end-to-end BGP configuration of your ExpressRoute circuit, and route advertisement will begin. Under the covers, your ExpressRoute gateway will form a BGP neighbor relationship to the MSEEs. This relationship is managed for you by Azure as part of the ExpressRoute service. 
f.	Once BGP routes are being advertised from Azure to your private network, the circuit will transition from a provisioned state to an established state.
g.	Congratulations, your ExpressRoute deployment is complete and ready to go!

To summarize, you need to have completed the following steps to have a functional ExpressRoute deployment: 

1.	Complete layer 1 connectivity via LOA process with ExpressRoute Direct, or an ExpressRoute deployment request with your selected partner.
2.	Complete layer 2 connectivity by creating an ExpressRoute circuit.
3.	Complete layer 3 BGP neighbor connectivity by configuring an ExpressRoute peering. 
4.	Complete layer 3 route advertisement by attaching a group of organized routes to your circuit. For the Azure private peering, this requires connecting your ExpressRoute gateway in the VNet to your circuit.
Figure1 below highlights the completed components of this deployment.  When all the above steps are finished, BGP routes advertised from your private network should be flowing into the VNet, and BGP routes from the VNet should be flowing through the ExpressRoute circuit to your BGP edge device. 

Figure 1, BPG route advertisement through the Azure private peering

![alt text](https://github.com/jgmitter/images/blob/master/ERBW.jpg)




# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
