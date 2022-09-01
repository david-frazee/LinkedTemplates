# Azure Firewall Forced Tunneling Lab Deployment

This ARM deployment includes everything needed to test Azure Firewall in a Forced Tunnel configuration. The environment will also allow testing scenarios where Split Tunneling may need to be applied for internet dependent connections.


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdavid-frazee%2FLinkedTemplates%2Fmain%2FAzureFirewall%2FForceTunnel%2FazfwForceTunnelTemplate.json)  



## PowerShell Deployment:

Please use this location as a reference on how this powershell commandlet works: https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-to-subscription?tabs=azure-powershell#deployment-commands  

*There are 5 parameters total. Some of the parameters have default values that match the dependencies in the template, do not change these. The other 2 are SecureString and require a value.*
* ResourceGroup1 - Do not change the Default Value
* ResourceGroup2 - Do not change the Default Value
* AdminUserName - Default value can be changed
* AdminPassword
* SharedKey 


*Adding some `samples` to give context*
- AdminPassword : "@$tr0NGp@sswOrb22"
- SharedKey : "$3cuR3MyvpN!78"  

**Example Powershell command with some parameters configured:**  
```New-AzSubscriptionDeployment -Name demoSubDeployment -Location centralus -TemplateUri "https://raw.githubusercontent.com/david-frazee/LinkedTemplates/main/AzureFirewall/ForceTunnel/azfwForceTunnelTemplate.json" -AdminPassword "@$tr0NGp@sswOrb22" -SharedKey "$3cuR3MyvpN!78" ```  


## Step-by-step documentation:
If you'd like more detailed step-by-step instructions on how to deploy this lab, follow the instructions below.

## Building the environment to test traffic through the Azure Firewall in Forced Tunnelling Mode
 
We will walk through the steps to build out the environment to configure resources and routing required to force tunnel traffic through the Azure Firewall. 
 
#### Create a Virtual Network Gateway for the Hub Virtual Network  
1.	In the Azure portal, search for Virtual network gateways
2.	Once in the Virtual network gateways blade, select Create. 
3.	For Subscription, select your subscription. 
4.	For Name, type vgw-vnet-hub-secured, and make sure to select the same Region as your virtual network.
5.	For Gateway type, select VPN, and for VPN type, select Route-based.
6.	For SKU, select VpnGw1 and leave Generation as Generation1.
7.	For Virtual network, select the drop-down menu and select vnet-hub-secured. The Subnet will auto-fill. 
8.	For Public IP Address Type, select Standard. 
9.	For Public IP address, select Create new and use pip-vgw-vnet-hub-secured for the Public IP address name.
10.	For Enable active-active mode and Configure BGP, leave as Disabled.
11.	Select Review + create. 
12.	Select Create. 
 
#### Create a Virtual Network Gateway for the On-Premises Virtual Network  
For the on-premises virtual network gateway, follow the steps from the previous section and make the necessary changes listed below.
1.	For Name, type vgw-vnet-onprem, and make sure to select the same Region as your on-premises virtual network.
2.	For Virtual network, select the drop-down menu and select vnet-onprem. The Subnet will auto-fill. 
3.	For Public IP address, select Create new and use pip-vgw-vnet-onprem for the Public IP address name.

#### Create Connection between the 2 Virtual Network Gateways
1.	Navigate to the Configuration blade of the Virtual network gateway, vgw-vnet-hub-secured. Check the button Configure BGP.
2.	We’ll have to change the ASN to have our two gateways communicate. Change the ASN to 65521 and record the BGP peer IP address. Click Save.
3.	Record the Public IP address of the Virtual network gateway from the Overview blade. The Public IP address, ASN, & BGP peer IP address will be needed to configure the next resource, a Local network gateways.
4.	In the Azure portal, search for Local network gateways. Click Create.
5.	We’re creating the Local network gateway that will represent the information for the Virtual network gateway, vgw-vnet-hub-secured. For Subscription, choose your subscription and select rg-fw-azure for the Resource Group.
6.	Select your Region and Name the resource lgw-azure-network.
7.	For Endpoint, select IP address and enter the Public IP address of vgw-vnet-hub-secured. Leave Address Space(s) blank.
8.	Click Advanced and click Yes for Configure BGP settings.
9.	Enter the ASN and BGP peer IP address of vgw-vnet-hub-secured. Click Review + create. Click Create.
10.	Repeat the above steps for the Virtual network gateway, vgw-vnet-onprem. For the ASN, change this to 65522 and name the Local network gateway, lgw-onprem-network.
11.	Navigate to the Connections blade of the Virtual network gateway, vgw-vnet-hub-secured. Click Add.
12.	Name the Connection, cn-lgw-onprem-network-to-vgw-vnet-hub-secured.
13.	For Connection type, select Site-to-site (IPsec). Leave the Virtual network gateway as is and select Local network gateway. Choose lgw-onprem-network.
14.	Enter a Shared key (PSK) and click Enable BGP. Select OK.
15.	Now make a second Connection for vgw-vnet-onprem. Use the same steps as above and Name the Connection cn-lgw-azure-network-to-vgw-vnet-onprem.

***Note***
***Since we cannot broadcast 0.0.0.0/0 in this environment, we’ll have to run a manual step using Azure PowerShell/CloudShell to enable force tunneling on the VPN gateways themselves. Open a CloudShell session in the Azure Portal and enter these commands.***

**$LocalGateway = Get-AzLocalNetworkGateway -Name "lgw-onprem-network" -ResourceGroupName "rg-fw-onprem"**
**$VirtualGateway = Get-AzVirtualNetworkGateway -Name "vgw-vnet-hub-secured" -ResourceGroupName "rg-fw-azure"**
**Set-AzVirtualNetworkGatewayDefaultSite -GatewayDefaultSite $LocalGateway -VirtualNetworkGateway $VirtualGateway**

We should then see that the gateway vgw-vnet-hub-secured has learned a 0.0.0.0/0 route. It will not show that the Next hop is the BGP peer IP of the vgw-vnet-onprem, but if we defined the correct Local network gateway in the above command, the traffic will traverse the tunnel.

#### Create the On-premises Firewall and Configure the Policy
For the on-premises firewall, we’ll use the same steps from configuring the Azure Firewall with defined changes. 
1.	Open the rg-fw-onprem resource group and select the vnet-onprem virtual network. In the left column, select Firewall.
2.	Select Click here to add a new firewall. 
3.	For Resource group, select rg-fw-onprem, and for Name, type azfw-vnet-onprem.
4.	For Region, select the same location of the virtual network and leave Availability zone as None.
5.	For Firewall tier, select Standard and keep Firewall management on Use a Firewall Policy to manage this firewall.
6.	For Firewall policy, select Add new. 
7.	Under Create a new Firewall Policy, for Policy name, type pol-azfw-vnet-onprem and for Region, select the same location used previously. 
8.	For Policy tier, select Standard and select OK.
9.	For Choose a virtual network, select Use existing and select vnet-onprem in the Virtual network drop-down.
10.	 For Public IP address, select Add new. 
11.	 Under Add a public IP, for Name, type pip-azfw-vnet-onprem and select OK.
12.	 Select Review + create 
13.	 Select Create. 
14.	 Navigate to the pol-azfw-vnet-onprem firewall policy and select the Network rules blade. Select Add a rule collection.
15.	 For Name, type east-west. Leave Rule collection type as Network. Make the Priority 1000, leave Rule collection action as Allow and leave Rule collection group as DefaultNetworkRuleColletionGroup.
16.	 For the Rule Name, type onprem-to-azure. Leave Source type as IP Address and enter 10.100.0.0/24 as Source. For Protocol, select Any and for Destination Ports, enter *. Leave Destination Type as IP Address and type 192.168.2.0/24 in Destination.
17.	 Create a second rule with Name azure-to-onprem. Leave Source type as IP Address and enter 192.168.2.0/24 as Source. For Protocol, select Any and for Destination Ports, enter *. Leave Destination Type as IP Address and type 10.100.0.0/24 in Destination.
18.	 Click Add.

#### Set up Peering between Hub & Spoke VNets
1.	Open the rg-fw-azure resource group and select the vnet-hub-secured virtual network. 
2.	In the left column, select Peerings and select +Add. 
3.	For Peering link name, type vnet-hub-secured-to-vnet-spoke-workers. 
4.	For Traffic to remote virtual network and Traffic forwarded from remote virtual network, leave the default selection. 
5.	For Virtual network gateway or Route Server, select Use this virtual network’s gateway or Route Server.  
6.	Now we’ll configure the Remote virtual network. For Peering link name, type vnet-spoke-workers-to-vnet-hub-secured. 
7.	For Subscription, select your subscription and for Virtual network, select vnet-spoke-workers.
8.	For Traffic to remote virtual network and Traffic forwarded from remote virtual network, leave the default selection.
9.	For Virtual network gateway or Route Server, select Use the remote virtual network’s gateway or Route Server.
10.	Select Add. 
 

***Notes:***
***•	When Forced Tunneling is enabled, DNAT rules are no longer supported due to asymmetric routing. This can be resolved with a User-Defined Route on the AzureFirewallSubnet Route Table configuration.*** 
***•	Creating Azure Firewall with Availability Zones that use newly created Public IPs is currently not supported. Zonal Public IPs created beforehand may be used without issue or you can use Azure PowerShell, CLI, and ARM Templates for the deployment. For more information about these known issues, see Known Issues.***

#### Create Route Tables for environment  
We’ll be creating 4 Route Tables in this step. 1 for the Spoke Network to force traffic to the Azure Firewall; 1 for the Azure Firewall to force traffic to on-premises; 1 for the on-premises virtual network gateway; and 1 for the on-premises network to return traffic back to its respective firewall. 
1.	In the Azure portal, search for Route table and select Create.
2.	For Subscription, select your subscription and for Resource group, select rg-fw-azure for the first 2 route tables.
3.	For Region, select the same location that you used previously. 
4.	For Name, type route-spoke-snets for the first route table, and route-fw-snet for the second. 
5.	For Propagate gateway routes, select No for route-spoke-snets and select Yes for route-fw-snet. This ensures on-premises routes learned from an ExpressRoute Gateway or VPN Gateway are not learned by the network interfaces in the spoke network. 
6.	Select Review + create. Select Create.
7.	For our other 2 route tables, we’ll be selecting the Resource group, rg-fw-onprem.
8.	For Region, select the same location that you used previously. 
9.	For Name, type route-onprem-snets for the first route table, and route-gateway-snets for the second.
10.	Leave Propagate gateway routes to Yes on both.

#### Configure User-Defined Routes to force traffic to Azure Firewall  
1.	Open the rg-fw-azure resource group and select the route-spoke-snets route table. 
2.	In the left column, select Routes and select Add.
3.	For Route name, type send-all-to-fw. Select IP Addresses for Address prefix destination and type 0.0.0.0/0 for Destination IP addresses/CIDR ranges.
4.	For Next hop type, select Virtual appliance and for Next hop address, type the private IP of the Azure Firewall. For this environment, it will be 192.168.0.68. Select Add.
5.	In the left column, select Subnets and select Associate.
6.	Under Associate subnet, for Virtual network, select vnet-spoke-workers and for Subnet, select snet-trust-workers. Select OK.
7.	Open the rg-fw-azure resource group and select the route-fw-snet route table
8.	In the left column, select Routes and select Add.
9.	For Route name, type send-to-onprem. Select IP Addresses for Address prefix destination and type 0.0.0.0/0 for Destination IP addresses/CIDR ranges.
10.	 For Next hop type, select Virtual network gateway and select Add.
11.	 Select Add again. For Route name, type DNAT. Select IP Addresses for Address prefix destination and type in your Public IP for Destination IP addresses/CIDR ranges. This will allow you to remote into the Virtual Machine for testing.
12.	 For Next hop type, select Internet and select Add.
13.	 In the left column, select Subnets and select Associate.
14.	 Under Associate subnet, for Virtual network, select vnet-hub-secured and for Subnet, select AzureFirewallSubnet. Select OK.
15.	 Open the rg-fw-onprem resource group and select the route-onprem-snets route table.
16.	 In the left column, select Routes and select Add.
17.	 For Route name, type to-fw. Select IP Addresses for Address prefix destination and type 192.168.2.0/24 for Destination IP addresses/CIDR ranges.
18.	 For Next hop type, select Virtual appliance and for Next hop address, type the private IP of the Azure Firewall. For this environment, it will be 10.100.0.132. Select Add.
19.	 In the left column, select Subnets and select Associate.
20.	 Under Associate subnet, for Virtual network, select vnet-onprem and for Subnet, select snet-onprem-workers. Select OK.
21.	 Open the rg-fw-onprem resource group and select the route-gateway-snets route table.
22.	 In the left column, select Routes and select Add.
23.	 For Route name, type to-snet. Select IP Addresses for Address prefix destination and type 10.100.0.64/28 for Destination IP addresses/CIDR ranges.
24.	 For Next hop type, select Virtual appliance and for Next hop address, type the private IP of the Azure Firewall. For this environment, it will be 10.100.0.132. Select Add.
25.	 Select Add again. For Route name, type to-fw. Select IP Addresses for Address prefix destination and type 13.89.172.22/32 for Destination IP addresses/CIDR ranges.
26.	 For Next hop type, select Virtual appliance and for Next hop address, type the private IP of the Azure Firewall.
27.	 In the left column, select Subnets and select Associate.
28.	 Under Associate subnet, for Virtual network, select vnet-onprem and for Subnet, select GatewaySubnet. Select OK.

***Note: When Forced Tunneling mode is not enabled, you’ll find that applying a user-defined route to force all traffic on the AzureFirewallSubnet will not work. This is due to the service management traffic requiring direct access to the internet. Creating a dedicated subnet called “AzureFirewallManagementSubnet” and enabling Forced Tunneling mode upon creation will allow for a successful deployment.***
***If you are using ExpressRoute with default route (0.0.0.0/0) broadcasted, this step is not supported. You can learn more about this routing configuration under the Virtual network gateway section in User-defined Custom routes.***

#### Create a Log Analytics Workspace and enable Diagnostic Settings for both Azure Firewalls
1.	In the Azure portal, search for Log Analytics workspaces and select Create.
2.	For Subscription, select your subscription and for Resource group, select rg-fw-azure
3.	For Name, type law-soc and select the same Region you’ve used for the rest of the environment. Click Review + Create, Create.
4.	Navigate to the azfw-vnet-hub-secured firewall and click on the Diagnostic settings blade. Click on +Add diagnostic setting.
5.	For Diagnostic setting name type diagnosticSettings. Under Logs, select allLogs and under Destination details, select Send to Log Analytics workspace.
6.	Select the workspace we just created and leave Destination table as Azure diagnostics. Click Save.
7.	Repeat steps 4-6 for azfw-vnet-onprem, using diagnosticSettings-onprem for the Name.

Now that we’ve created the necessary resources and configured the environment, we can now deploy 2 Virtual Machines to their respective virtual networks/subnets. Deploy 1 Windows virtual machine in the vnet-spoke-workers/snet-trust-workers environment, and 1 Windows virtual machine in the vnet-onprem/snet-onprem-workers environment. You’ll be able to use the DNAT rule created in the pol-azfw-vnet-hub policy to remote into the VM in the snet-trust-workers environment to test the routing.



## What is included with the Azure Firewall Forced Tunnel Deployment Template  

<p align="center">
<img src="https://github.com/david-frazee/LinkedTemplates/blob/main/AzureFirewall/Media/AzFwForceTunnel.png">
</p>

| Resource |  Purpose |
|----------|---------|
| Virtual Network-1 |  VN1(Hub) has 2 Subnets 10.0.25.0/26 & 10.0.25.64/26 peered to VN1 and VN2 (Enabled with DDoSProtection)|
| Virtual Network-2 |  VN2(Spoke1) has 2 Subnets 10.0.27.0/26 & 10.0.27.64/26 peered to VN2 |
| Virtual Network-3 |  VN3(Spoke2) has 2 Subnets 10.0.28.0/26 & 10.0.28.64/26 peered to VN1 |
| PublicIPAddress-1 |  Static Public IP address for Application gateway |
| PublicIPAddress-2 |  Static Public IP address for Azure firewall |
| Virtual Machine-1 | Windows 10 Machine connected to VN2(subnet1) |
| Virtual Machine-2 | Kali Linux Box connected to VN2(subnet2) |
| Virtual Machine-3 | Server 2019 Machine connected to VN3(subnet1) |
| Network Security Group-1 | Pre-configured NSG to Virtual Networks associated to VN2 subnets |
| Network Security Group-2 | Pre-configured NSG to Virtual Networks associated to VN3 subnets |
| Route Table | Pre-configured RT Associated to VN2 and VN3 subnets with default route pointing to Azure firewall private IP address |
| Application Gateway v2 (WAF) | Pre-configured to publish webapp on HTTP on Public Interface|
| Azure Firewall Premium with Firewall Manager | Pre-configured with RDP(DNAT) rules to 3 VM's and allow search engine access(application rules) from VM's. Network rule configured to allow SMB, RDP and SSH access between VM's. Azure firewall is deployed in Hub Virtual Network managed by Firewall manager |
| Frontdoor | Pre-configured designer with Backend pool as Applicaion gateway public interface  |
| WebApp(PaaS) | Pre-configured app for Frontdoor and Application Gateway WAF testing |  


> This build has diagnostic settings enabled by default; it requires a Log Analytics workspace for logs to be collected. https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace  


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.



