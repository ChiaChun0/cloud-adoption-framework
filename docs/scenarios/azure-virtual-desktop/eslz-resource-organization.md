---
title: Design guidance for Azure Virtual Desktop
description: Learn about the resource organization design area and how to apply it to your Azure Virtual Desktop implementation.
author: martinekuan
ms.author: martinek
ms.date: 04/19/2022
ms.topic: conceptual
ms.custom: think-tank, e2e-avd
---

# Resource organization considerations for Azure Virtual Desktop

The structure of your resource storage directly impacts how you implement resource management and governance. This article provides key considerations and recommendations for designing an organization's structure.

Use this guidance to ensure resource organization and segmentation across:

- Management group hierarchies
- Subscriptions
- Resource groups
- Landing zones

## Design considerations

The sections outline key factors to consider when planning your organization's Azure Virtual Desktop structure.

### Number of virtual machines

How many Azure Virtual Desktop virtual machines (VMs) does your organization require?

- Avoid deploying more than 5,000 VMs in a single region. You can accommodate extra user sessions by increasing individual session host VM resources.
- For enterprise environments exceeding 5,000 VMs per subscription in a single region, create multiple Azure subscriptions. Place these subscriptions in a hub-spoke architecture, and connect them through virtual network peering. You can increase the number of VMs by deploying VMs in a different region of the same subscription.

### Regions for host deployment

Which regions should you choose for deploying hosts?

We generally recommend storing all resources in the same Azure region as your Azure Virtual Desktop deployment. The main resources involved are:
- **Metadata (Services Objects)**: Host Pools, Application Groups, and Workspaces.
- **Session Hosts (Virtual Desktops) compute**: Virtual machines, disks, and network interfaces.
- **VNets** (the VNet where the Session Hosts are directly connected)
- **Storage** (for FSLogix user profiles)
- **Other resources**: Azure Compute Galleries, Key Vaults, and images.

### Other considerations

- Deploy session hosts in Azure regions closest to your users to reduces network connectivity and latency and improve performance.
- Ensure compliance with regional regulations and data residency requirements when selecting a region
- Running applications in session hosts located far from services (e.g., session hosts in Central India reaching services in Central US) often increase application latency. Placing session hosts closer to the required resources may reduces this risk (e.g., Central US in this example).
- Avoid mixing session hosts from different Azure regions (e.g., Central India and Central US) in the same host pool as you can't assign users to a session host in a specific Azure Region. Create separate session hosts for each Azure Region instead.

## Design recommendations

This sections offer guidance on managing groups, naming , and tagging in Azure Virtual Desktop.

### Management settings scope

Azure provides four levels of management: management groups, subscriptions, resource groups, and resources. You can apply management settings like policies and role-based access control at any management level. 
- **Management groups** enable management of access, policies, and compliance across multiple subscriptions. All subscriptions in a management group automatically inherit the conditions that are applied to the management group. Group resources logically in management groups to target policies and initiative assignments with Azure Policy. Create management groups under the root-level management group to represent workload types (archetypes) you host, and management groups based on security, compliance, connectivity, and feature needs. If you use this grouping structure, you can apply a set of Azure policies at the management group level for all workloads that require the same security, compliance, connectivity, and feature settings.
- **Subscriptions** logically associate user accounts with the resources they create. Subscriptions serve as a scale unit, allowing component workloads can scale within your platform [subscription limits](/azure/azure-resource-manager/management/azure-subscription-service-limits). Each subscription has limits or quotas on the amount of resources that be mindful of subscription quotas and resource limits during workload design. Organizations can use subscriptions to manage costs and the resources that are created by users, teams, and projects as subscriptions providing a clear management boundary for governance and isolation.
- **Resource groups** are logical containers for deploying and managing Azure resources like Azure Virtual Desktop, virtual machines, and storage.
- **Resources** are instances of services created within a resource group, such as Azure Virtual Desktop.

Below is an example of the recommended structure and resource groups to create and use as administrative domains and for lifecycle management in each Azure region.
- Components
    - Azure Virtual Desktop Service Objects: Create a resource group for Azure Virtual Desktop Service Objects from Host Pool VMs.  Service objects like Workspaces, Host Pools and Application Groups.
    - Networking: Typically created as part of the Cloud Adoption Framework Landing Zone.
    - Storage: If not already created as part of Cloud Adoption Framework, create a resource group for storage accounts.
    - Session hosts compute: Create a Resource Group for virtual machines, disks, and network interfaces as these have a different lifecycle from Azure Virtual Desktop Service Objects.
    - Shared Resources: Create a Resource Group for shared resources such as custom VM images, this encourages self-service so you could have a subscription for each business line, for instance.
- Basic Structure:
    - Subscription AVD-Shared-Resources
        - rg-avd-<_Azure-Region_>-shared-resources
    - Subscription AVD LZ
        - rg-avd-<_Azure-Region_>-<_Workload_>-service-objects
        - rg-avd-<_Azure-Region_>-<_Workload_>-pool-compute
        - rg-avd-<_Azure-Region_>-<_Workload_>-network
        - rg-avd-<_Azure-Region_>-<_Workload_>-storage

:::image type="content" source="../../../docs/scenarios/azure-virtual-desktop/media/avd-resource-management-1.png" alt-text="Screenshot that shows the AVD Shared Resources subscription." lightbox="../../../docs/scenarios/azure-virtual-desktop/media/avd-resource-management-1.png":::

:::image type="content" source="../../../docs/scenarios/azure-virtual-desktop/media/avd-resource-management-2.png" alt-text="Screenshot that shows the AVD Service Objects and compute subscription." lightbox="../../../docs/scenarios/azure-virtual-desktop/media/avd-resource-management-2.png":::

### Naming standards

A consistent naming standard helps organize resources, streamline management, enable cost tracking, and ensure effective governance. Your naming strategy should include business and operational details in resource names.
- Business details should include the organizational information to identify teams. Use the resource's short name along with the names of the business owners who are responsible for the resource costs.
- Operational details should include the information helpful for IT teams to identify the workload, application, environment, criticality, and other information that's useful for managing resources.

A well-structured naming system allows for rapid resource identification for both management and accounting purposes. Consistency across resources helps identify any deviations from agreed-upon policies. Consider whether to align your cloud naming conventions with existing IT naming standards or to create separate, unique conventions for the cloud. [Prescriptive guidance for resource tagging](../../govern/guides/complex/prescriptive-guidance.md#resource-tagging) outlines how patterns can support governance practices. 

### Resource tags

Tags also help evaluate regulatory compliance by logically organizing Azure resources into categories.

Each tag consists of a name and a value, providing context about the associated workload, application, operational requirements, or ownership. For example, you can apply the tag name _environment_ with the value _production_ to categorize all production resources.

Common uses for tags include:

- **Metadata and documentation**: Tags like _ProjectOwner_ help administrators easily view resource details.
- **Automation**: Tags such as _ShutdownTime_ or _DeprovisionDate_ can trigger actions through scripts.
- **Cost optimization**: Tags can be used to allocate resources to specific teams who are responsible for the costs. In [Microsoft Cost Management](/azure/cost-management-billing/), you can filter reports by cost center tag to track charges by team or department.

## Next steps
Learn more about Azure Virtual Desktop organization and management, see:
- [Azure Virtual Desktop resource organization](/azure/architecture/example-scenario/azure-virtual-desktop/azure-virtual-desktop#azure-limitations)
- [Organize your Azure resources effectively] (https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources)
- [Naming and tagging in Azure](../../ready/azure-best-practices/resource-naming-and-tagging-decision-guide.md)

Learn about management and monitoring for an Azure Virtual Desktop enterprise-scale scenario.

> [!div class="nextstepaction"]
> [Management and monitoring](./eslz-management-and-monitoring.md)
