---
title: Azure Firewall Demo
description: Azure Firewall Demo
author: ashish-kapila
ms.author: askapila
ms.date: 08/07/2020
ms.topic: conceptual
ms.service: cloud management
---

# Azure Firewall

## Verify Firewall Policy and Threat Intel settings

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure Microsoft directory is current, and the subscriptions CyberSecSOC and Contoso Hotels are selected under subscriptions. In case you are having difficulty selecting the subscription, please refer to <https://aka.ms/AzureDemoAccess>.

    a. Microsoft Tenant:
    
    <p align="left">
    <img src="../media/microsoft-directory.png">
    </p>
        
    b. Subscriptions:
    
    <p align="left">
    <img src="../media/microsoft-contosohotels-subscriptions-v3.png">
    </p>

2. Search for Firewall Policies and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In the left menu, select **Threat Intelligence** and you can see that Threat Intelligence Mode is set to **Alert and Deny**.

4. In Network Rules, you will see five rules, three of these rules are relevant to the following scenarios.

    a. SMB rule allows SMB(445) traffic between VM-WIN10 and VM-WIN2019

    b. RDP rule allows RDP(3389) traffic between VM-WIN10 and VM-WIN2019

    c. SSH rule allows SSH(22) traffic from VM-WIN10, VM-WIN2019 to VM-KALI

5. In Application rules, you will see four rules.

    a. First rule is called NetSec-TechCommunity and this allow access to a specific URL which is our [Azure Network Security TechCommunity]( https://techcommunity.microsoft.com/t5/azure-network-security/bd-p/AzureNetworkSecurity). This shows the firewall premium capability of URL filtering using TLS inspection

    b. Second Rule is called DTF-InternetAccess which allows unrestricted internet access from a specific internal range

    c. Third Rule is called Allowed-WebCategories which allow access to Search Engines and News category and has TLS inspection enabled. To test this rule, you can access any search engine or news site like www.bing.com

    d. Last rule is called Allow-WindowsUpdate which uses the FQDN tag capability to allow access from internal network to windows update. This rule allows windows update traffic from internal VM's

6. In left menu, click on **Secured virtual networks** and you can see that VN-HUB is configured as hub virtual network.

## Scenario: Threat intel in action

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC and Contoso Hotels are selected under subscriptions.

2. Search for Virtual Machines and select VM-Win2019.

3. On VM-Win2019 blade, under Operations, select Bastion.

4. Username and password to login on VM-Win2019 is stored in Key Vault **CH-SecOpsKV** under **secrets**. You can find the username under **win2019username** and password under **win2019password**.

5. Once logged in, open browser and browse to <http://testmaliciousdomain.eastus.cloudapp.azure.com/>.

6. You will then see the message that threat intel has blocked the request.
    HTTP  request from 10.0.27.4:61992 to testmaliciousdomain.eastus.cloudapp.azure.com:80. Action: Deny. ThreatIntel: This is a test indicator for a Microsoft owned domain.

## Scenario: Controlling Access between Spoke Vnets (Network Rule)

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC and Contoso Hotels are selected under subscriptions.

2. Search for Virtual Machines and select VM-Win2019.

3. On VM-Win2019 blade, under Operations, select Bastion.

4. Username and password to login on VM-Win2019 is stored in Key Vault **CH-SecOpsKV** under **secrets**. You can find the username under **win2019username** and password under **win2019password**.

5. Now open command prompt and try pinging VM-Win10 (10.0.27.4).

6. You will see request time out as VM-Win10 is on a different VNet and there is no network rule on Azure firewall to allow ICMP traffic.

7. Now try RDP to VM-Win10 and that will succeed as there is a network rule which allow RDP traffic to VM-Win10.

### Verify Firewall Network rule which allow the traffic

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected.

2. Search for Firewall Policy and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In left menu, select **Threat Intel** and you can see that Threat Intelligence Mode is set to **Alert and Deny**.

4. In left menu, select **Rules** and you would be able to see DNAT rules, Network rules and Application rules.

5. In Network Rules, you will see five rules, three of these rules are relevant to the above scenarios.

    a. SMB rule allows SMB(445) traffic between VM-WIN10 and VM-WIN2019

    b. RDP rule allows RDP(3389) traffic between VM-WIN10 and VM-WIN2019

    c. SSH rule allows SSH(22) traffic from VM-WIN10, VM-WIN2019 to VM-KALI

## Scenario: Secure internet access using Azure Firewall (Application Rule)

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC and Contoso Hotels are selected under subscriptions.

2. Search for Virtual Machines and select VM-Win2019.

3. On VM-Win2019 blade, under Operations, select Bastion.

4. Username and password to login on VM-Win2019 is stored in Key Vault **CH-SecOpsKV** under **secrets**. You can find the username under **win2019username** and password under **win2019password**.

5. In the browser, try browsing to [Azure Network Security TechCommunity]( https://techcommunity.microsoft.com/t5/azure-network-security/bd-p/AzureNetworkSecurity) and you can successfully load the page as we have an Application rule in Firewall Policy which allow this URL with TLS inspection enabled.

6. In the same browser, if you click on the padlock to see the certificate for TechCommunity blog, you will notice that the certificate is issued by AzureFirewallCA which means that Azure Firewall is enabled for TLS inspection for this URL.

7. Now try browsing <https://www.microsoft.com> and you will see a message that the request is denied by the azure firewall.

    *HTTP  request from 10.0.28.4:53955 to <https://www.microsoft.com:443>. Action: Deny. No rule matched. Proceeding with default action*

### Verify Firewall Application rule which allow the traffic

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected under subscriptions.

2. Search for Firewall Policy and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In left menu, under **Settings** and you would be able to see DNAT rules, Network rules and Application rules.

4. In Application rules, you will see four rules.

    a. First rule is called NetSec-TechCommunity and this allow access to a specific URL which is our [Azure Network Security TechCommunity]( https://techcommunity.microsoft.com/t5/azure-network-security/bd-p/AzureNetworkSecurity) This shows the firewall premium capability of URL filtering using TLS inspection

    b. Second Rule is called DTF-InternetAccess which allow unrestricted internet access from a specific internal range

    c. Third Rule is called allowed-webcategories which allow access to Search Engine and News category and has TLS inspection enabled. To test this rule, you can access any search engine or news site like www.bing.com

    d. Last rule is called allow-windowsupdate which uses the FQDN tag capability to allow access from internal network to windows update. This rule allows windows update traffic from internal VM's

### Verify TLS Inspection configuration for Azure Firewall Premium

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected under subscriptions.

2. Search for Firewall Policy and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In left menu, under **Settings** and you would see **TLS Inspection**.

4. You will see that TLS inspection is enabled and then you will see **Key Vault**.

5. **Key Vault** KV-SOC-NS-IDPS is used to store the certificate which is used by Azure Firewall for TLS inspection. Azure Firewall is using the Managed Identity called **FW-KV-TLS**.

6. You will not be able to see the certificate as you do not have permission on the Key Vault.

### Verify IDPS configuration for Azure Firewall Premium

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected under subscriptions.

2. Search for Firewall Policy and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In left menu, under **Settings** and you would see **IDPS**.

4. You will see IDPS is enabled and configured in **Alert** mode.

5. Under **Signature Rules**, you can customize the IDPS mode for specific signatures. We have configured **2008983** SignatureID as denied.

6. Under **Bypass list**, You can configure IDPS to not filter traffic to any of the IP addresses, ranges, and subnets.

## Scenario: Test Firewall Premium IDPS capability

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC and Contoso Hotels are selected under subscriptions.

2. Search for Virtual Machines and select VM-Win2019.

3. On VM-Win2019 blade, under Operations, select Bastion.

4. Username and password to login on VM-Win2019 is stored in Key Vault **CH-SecOpsKV** under **secrets**. You can find the username under **win2019username** and password under **win2019password**.

5. Open CMD and run the following CURL command.

    **curl -A "BlackSun" <https://www.bing.com> --ssl-no-revoke**

6. You will notice that connection is blocked even though IDPS is configured to **Alert** only. If you check the signature rules you will notice that we have specifically configured signatureID **2008983** to deny mode. This the signature which blocks request from suspicious user agent including blacksun.

7. If you run the same CURL command and use a different known user agent then the request will be allowed.

    **curl -A "Mozilla/5.0" <https://www.bing.com> --ssl-no-revoke**

## Diagnostic setting and logging of Azure Firewall

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected under subscription.

2. Search for Azure Firewall.

3. Select **SOC-NS-FW** in resource group of **SOC-NS**.

4. In left Menu, under Monitoring, select diagnostic settings and verify the settings.

5. You will see that AzureFirewallApplicationRule and AzureFirewallNetworkRule log types were configured to be collected and forwarded to the CyberSecuritySOC Log Analytics workspace.

6. Click on CyberSecuritySOC under Log Analytics Workspace which will open the Log analytics workspace blade for CyberSecuritySOC.

7. In the left Menu, select **logs** and copy/paste the following query and click on **Run**.

    ```kusto
    AzureDiagnostics
    | where Category contains "AzureFirewallApplicationRule"
    ```

8. This will show you all the logs related to Application rules including web categories.

9. Now in the query input box copy/paste the following query and click on **Run**.

    ```kusto
    AzureDiagnostics
    | where Category contains "AzureFirewallNetworkRule"
    ```

10. This will show you all the logs related to Network rules including Threat Intel Logs. Expand a log and you can see information about the request.

11. Now in the query input box copy/paste the following query and click on **Run**.

    ```kusto
    AzureDiagnostics
    | where OperationName == "AzureFirewallIDSLog"
    ```

12. This will show you all the logs related to IDPS capability of azure firewall premium. Expand a log and you can see information about the request.

## Resource Specific tables and logging of Azure Firewall

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected under subscription.

2. Search for Azure Firewall.

3. Select **SOC-NS-FW** in resource group of **SOC-NS**.

4. In left Menu, under Monitoring, select diagnostic settings and verify the settings. You should see a Diagnostic setting named "VN-HUBStructuredLogs".

5. You will see the following Logs selected to be collected and forwarded to the CyberSecuritySOC Log Analytics workspace: 
   - Azure Firewall Network Rule Hit
   - Azure Firewall Application Rule Hit
   - Azure Firewall Nat Rule Hit
   - Azure Firewall ThreatIntel Hit
   - Azure Firewall Idps Signature Hit
   - Azure Firewall Dns query Hit
   - Azure Firewall Fqdn Resolution Failure Hit
   - Azure Firewall Network Rule Aggregation Hit (Used for Policy Analytics)
   - Azure Firewall Application Rule Aggregation Hit (Used for Policy Analytics)
   - Azure Firewall Nat Rule Aggregation Hit (Used for Policy Analytics)

6. You will also see under Destination details, that the Destination table has **Resource specific** selected. This ensures that we are using the resource specific log tables.

7. Click on CyberSecuritySOC under Log Analytics Workspace which will open the Log analytics workspace blade for CyberSecuritySOC.

8. In the left Menu, select **logs** and copy/paste the following query and click on **Run**.

    ```kusto
    AZFWApplicationRule
    ```

9. This will show you all the logs related to Application rules including web categories.

10. Now in the query input box copy/paste the following query and click on **Run**.

    ```kusto
    AZFWNetworkRule
    ```

11. This will show you all the logs related to Network rules. Expand a log and you can see information about the request.

12. Now in the query input box copy/paste the following query and click on **Run**.

    ```kusto
    AZFWThreatIntel
    ```

13. This will show you all the logs related to Threat Intelligence. Expand a log and you can see information about the request.

14. Now in the query input box copy/paste the following query and click on **Run**.

    ```kusto
    AZFWIdpsSignature
    ```

15. This will show you all the logs related to IDPS Signatures. Expand a log and you can see information about the request.

## Scenario: Use Policy Analytics to provide insights, centralized visibiltiy, and control to Azure Firewall

1. Login to the Microsoft Tenant, with your Microsoft Alias <https://aka.ms/AzureDemo> and ensure CyberSecSOC is selected.

2. Search for Firewall Policy and select **SOC-NS-FWPolicy-prem** in resource group of **SOC-NS**.

3. In left menu, select **Policy Analytics** and you should see the Insights tab show 6 different dashboards, aggregating insights, and highlighting relevant policy information. The Insights tab will be able to quickly show us max limitations for the Policy, rules that may need immediate attention, and potentially malicious sources hitting the Azure Firewall's rules.

4. The first dashboard covers the max policy limits for the Azure Firewall Policy. You can see that our rule count, unique source & destination IPs, IP groups and DNAT rules are well below the max limits.

5. The second dashboard covers "Rules with multiple IP addresses". You'll see that the dashboard shows that "Your rules are all good." Click on the **Gears** icon and slide the Source IP addresses and Destination IP addresses to the value of 2, then select Apply. Policy Analytics shows us that we have 2 rules for both values that match the criteria. Click on "See recommendations" and notice how the recommended action is to convert the individual IPs to IP Groups.

6. The third dashboard covers "Rules with low utilization". You may see around 5 rules that have no utilization in the last 30 days. Click on "See recommendations" and click on the "Recommended action" for the rule **SMB**. The actions available to us are to "Move the rule to a lowest priority rule collection group", or to "Delete the rule".

7. The fourth dashboard covers "Duplicate rules". Click on the **Gears** icon and you'll see different values that you can show recommendations on. We should be able to see at least 1 Redundant rule in the dashboard for this environment. Click on "See recommendations" and then select the "Redundant rules" tab to identify what rules are redundant. Our **SMB-Duplicate** rule should be flagged as the rule that needs review and possible actions taken to optimize the overall performance of the Azure Firewall.

8. The fifth dashboard covers "Generic rules", where they may be a wildcard (*) entered into the source or destination field. Clicking on the recommendation will take us to the Single-rule analysis tab. Select the "Recommended action for the rule **NetSec-TechCommunity** and select Agree. The Single-rule analysis will run, and you will be able to see a Rule summary as well as Rule recommendations and matching traffic. We have the option to then "Delete the rule" or to "Move the rule to a lowest priority rule collection group".

9. The final dashboard covers "Potentially malicious sources", where it filters traffic that was identified through Threat Intelligence or IDPS. Clicking on "See recommendations", the actions are to create new rules to block this potentially malicious traffic.

10. Now we can navigate through the other Tabs to check on the different traffic flows for each rule collection. Click on DNAT rules, Network rules, Application rules, and Traffic flows to see the different traffic that is being processed per rule collection group and per rule. 

## Azure Firewall Workbook using Microsoft Sentinel

1. You can use our custom workbook for Azure Firewall in Microsoft Sentinel.

2. Login to the Microsoft Tenant, with your Microsoft Alias <https://ms.portal.azure.com> and ensure CyberSecSOC is selected under subscriptions.

3. Search for Sentinel and select CyberSecuritySOC.

4. Now in left Menu, under **Threat Management** select **Workbooks**.

5. You will see a workbook called Azure Firewall under **my workbooks**.

6. Click on View saved Workbook.

7. Click and open the workbook and you can visualize the firewall logs in dashboard.
