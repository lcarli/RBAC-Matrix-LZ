## RBAC Matrix Project

### Overview
The "RBAC Matrix" project introduces a comprehensive framework designed to establish a clear and scalable structure of role assignments within Azure landing zones. This initiative aims to guide organizations in mapping Active Directory (AD) groups to specific Azure roles, specifying the scope and responsibilities of these roles, and ensuring effective governance and security within their cloud environments.

![RBAC Matrix Diagram](/Images/RBAC-Matrix.jpg "RBAC Matrix Diagram")

For the complete Visio diagram, click here: [Visio link]

### Role Definitions and Assignments

#### Built-in Azure Roles and Custom Roles
Roles within the RBAC Matrix are categorized into two main types: **built-in Azure roles** and **custom roles**. Custom roles, denoted by a prefix of 'CUSTOM' in their naming convention, are tailored to meet specific organizational needs that go beyond the default Azure role configurations.

Example of a custom role definition:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "roleName": {
      "value": "Network Administrator"
    },
    "roleDescription": {
      "value": "Manages platform-wide global connectivity, such as virtual networks, UDRs, NSGs, NVAs, VPNs, Azure ExpressRoute, and others"
    },
    "location": {
      "value": "canadaeast"
    },
    "managementGroupId": {
      "value": "mg-root"
    },
    "scope": {
      "value": "managementGroup"
    },
    "actions": {
      "value": [
        "*/read",
        "Microsoft.Network/*",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Support/*"
      ]
    },
    "notActions": {
      "value": []
    },
    "dataActions": {
      "value": []
    },
    "notDataActions": {
      "value": []
    }
  }
}
```

This role includes enhanced privileges tailored to managing network configurations securely and efficiently across Azure services.

Some custom roles used in the sample:
| Administrative role or area       | Description                                                                                     | Actions                                                                                                                                                  | NotActions                                                                                           |
|-----------------------------------|-------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Network Administrator             | Manages platform-wide global connectivity, such as virtual networks, UDRs, NSGs, NVAs, VPNs, Azure ExpressRoute, and others.                            | `*/read`, `Microsoft.Network/*`, `Microsoft.Resources/deployments/*`, `Microsoft.Support/*`                                                               |                                                                                                      |
| Security Administrator            | Security Administrator role with a horizontal view across the entire Azure estate and the Key Vault purge policy.                                      | `*/read`, `*/register/action`, `Microsoft.KeyVault/locations/deletedVaults/purge/action`, `Microsoft.PolicyInsights/*`, `Microsoft.Authorization/*`, `Microsoft.Security/*`, `Microsoft.Support/*` |                                                                                                      |
| AppGateway Health Probe Reader    | Delegated role for the subscription owner.                                                                                 | `Microsoft.Network/applicationGateways/getBackendHealthOnDemand/action`, `Microsoft.Network/applicationGateways/backendhealth/action`                   |                                                                                                      |
| Workload VM Contributor           | Contributor role for the Virtual Machines into Landing Zone scope.                                                           | `Microsoft.Network/virtualNetworks/joinLoadBalancer/action`, `Microsoft.KeyVault/vaults/deploy/action`, `Microsoft.Network/networkSecurityGroups/join/action`, `Microsoft.Network/virtualNetworks/subnets/join/action`, `Microsoft.OperationalInsights/workspaces/listKeys/action` |                                                                                                      |

#### Role Assignment Table

The following table provides an example of how roles are assigned to various AD groups within different scopes:

| Scope             | GroupName                    | Role                                  | Others Roles                                                               | PIM | Is temporary? | Is SPN? |
|-------------------|------------------------------|---------------------------------------|----------------------------------------------------------------------------|-----|---------------|---------|
| mg-root           | AzureCloud-Owner             | Owner                                 |                                                                            | X   | X             |         |
| mg-root           | AzureCloud-Contributor       | Contributor                           |                                                                            | X   |               |         |
| mg-root           | Azure-Reader                 | Reader                                |                                                                            |     |               |         |
| mg-platform-prod  | AzurePlatform-Contributor    | Contributor                           |                                                                            |     | X             |         |
| mg-root           | NetworkAdmin                 | Custom - Network Administrator        |                                                                            |     |               |         |
| mg-{org}-sandbox  | Sandbox-Contributor          | Contributor                           |                                                                            |     |               |         |
| mg-root           | SecurityAdmin                | Custom - Security Administrator       |                                                                            |     |               |         |
| mg-{org}-security | SecurityOperations           | -                                     | Could Have Tier 1 and 2 were tier 1 is Microsoft Sentinel Responder and...|     |               |         |
| mg-root           | FinOps-Reader                | Billing Reader                        |                                                                            |     |               |         |
| mg-root           | Database-Contributor         | Custom - Database Administrator       |                                                                            |     |               |         |
| mg-management-prod| Management-Contributor       | Contributor                           |                                                                            |     | X             |         |
| mg-connectivity-prod | Connectivity-Contributor  | Contributor                           |                                                                            |     | X             |         |
| mg-identity-prod  | Identity-Contributor         | Contributor                           |                                                                            |     | X             |         |
| mg-{org}-prod     | AzureProd-Reader             | Reader                                |                                                                            |     |               |         |
| mg-{org}-dev      | AzureDev-Reader              | Reader                                |                                                                            |     |               |         |
| *Workload RGs     | {Workload}-LeadTeam          | Contributor - Dev \| Reader - Prod    |                                                                            |     |               |         |
| *Workload RGs     | {Workload}-OpsTeam           | Reader - Dev \| Reader - Prod         | Workload VM Contributor                                                    |     |               |         |
| mg-root           | Backup-Contributor           | Custom - Backup Contributor           |                                                                            |     |               |         |
| mg-root           | Policy-Contributor           | Custom - Policy Contributor           |                                                                            |     |               |         |
| mg-root           | NetworkOperations            | Custom - Network Operations           |                                                                            |     |               |         |
| mg-root           | Monitoring-Contributor       | Custom - Monitoring Contributor       |                                                                            |     |               |         |
| mg-{org}-prod     | DevOps-SPN-Prod-Admin        | Owner                                 |                                                                            |     | X             |         |
| mg-{org}-prod     | DevOps-SPN-Prod              | Contributor                           |                                                                            |     | X             |         |
| mg-{org}-dev      | DevOps-SPN-Dev-Admin         | Owner                                 |                                                                            |     | X             |         |
| mg-{org}-dev      | DevOps-SPN-Dev               | Contributor                           |                                                                            |     | X             |         |
| mg-{org}-sandbox  | DevOps-SPN-Sandbox           | Contributor                           |                                                                            |     | X             |         |
| {org}-{workload}-Prod | DevOps-SPN-{Workload}-Prod| Contributor                           |                                                                            |     | X             |         |
| {org}-{workload}-Dev | DevOps-SPN-{Workload}-Dev  | Contributor                           |                                                                            |     | X             |         |


### Managing Temporary Roles

Temporary roles are critical during the transitional phase where an organization is implementing DevOps Infrastructure as Code (IAC) practices. These roles are assigned to support operations temporarily and are phased out once the IAC framework is fully functional.

### Service Principals (SPNs)

Service Principals, particularly those used by DevOps during the deployment pipeline, play a vital role in managing Azure resources. The SPN DevOps are specifically configured to interact with Azure services securely during automated deployments, ensuring that operations are executed with minimal human intervention.

In the configuration of Service Principal Names (SPNs) in Azure, it is common practice to associate each SPN with a specific environment to ensure effective management of access and security. In our specific scenario, we utilize two types of SPNs: an "admin" SPN and a "normal" SPN. The key distinction between them lies in the levels of access granted within Azure. The normal SPN holds the role of "CONTRIBUTOR", allowing it to make modifications to resources within the scope of its permissions, but without the ability to change roles or access permissions. On the other hand, the admin SPN is designated as "OWNER", which confers broader capabilities including the management of Role-Based Access Control (RBAC), allowing it to add or modify permissions as needed.

Furthermore, it is advisable that regular users only have "reader" permissions in Azure environments, restricting them to viewing resources without making changes. This approach reinforces the practice of Infrastructure as Code (IaC), where all modifications to the environments are performed through code, providing a more controlled and auditable management of resources.


## FAQ

#### Why to many roles on the Root Management Group?

In a zero-trust architecture, roles are designed to grant only the minimum access necessary for infrastructure maintenance on a global scale. Having multiple specific roles at the Root Management Group in Azure, such as "Backup Contributor," "Database Contributor," and "Policy Contributor," allows for granular access control. This approach not only adheres to the principle of least privilege but also simplifies management and enhances security by clearly defining responsibilities and limiting the potential for security breaches. These roles enable consistent enforcement of policies across all resources, ensuring that each user has exactly the access they need, no more, no less.
