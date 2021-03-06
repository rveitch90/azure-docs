---
title: FAQ and known issues with Managed Service Identity (MSI) for Azure Active Directory
description: Known issues with Managed Service Identity for Azure Active Directory.
services: active-directory
documentationcenter: 
author: BryanLa
manager: mtillman
editor: 
ms.service: active-directory
ms.devlang: 
ms.topic: article
ms.tgt_pltfrm: 
ms.workload: identity
ms.date: 12/15/2017
ms.author: bryanla
ROBOTS: NOINDEX,NOFOLLOW
---

# FAQ and known issues with Managed Service Identity (MSI) for Azure Active Directory

[!INCLUDE[preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

## Frequently asked questions

### Does MSI work with Azure Cloud Services?

No, there are no plans to support MSI in Azure Cloud Services.

### Does MSI work with the Active Directory Authentication Library (ADAL) or the Microsoft Authentication Library (MSAL)?

No, MSI is not yet integrated with ADAL or MSAL. For details on acquiring an MSI token using the MSI REST endpoint, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](msi-how-to-use-vm-msi-token.md).

### What are the supported Linux distributions?

The following Linux distributions support MSI: 

- CoreOS Stable
- CentOS 7.1
- RedHat 7.2
- Ubuntu 15.04

Other Linux distributions are currently not supported and extension might fail on unsupported distributions.

The extension works on CentOS 6.9. However, due to lack of system support in 6.9, the extension will not auto restart if crashed or stopped. It restarts when the VM restarts. To restart the extension manually, see [How do you restart the MSI extension?](#how-do-you-restart-the-msi-extension)

### How do you restart the MSI extension?
On Windows and certain versions of Linux, if the extension stops, the following cmdlet may be used to manually restart it:

```powershell
Set-AzureRmVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Where: 
- Extension name and type for Windows is: ManagedIdentityExtensionForWindows
- Extension name and type for Linux is: ManagedIdentityExtensionForLinux

## Known issues

### "Automation script" fails when attempting schema export for MSI extension

When Managed Service Identity is enabled on a VM, the following error is shown when attempting to use the “Automation script” feature for the VM, or its resource group:

![MSI automation script export error](~/articles/active-directory/media/msi-known-issues/automation-script-export-error.png)

The Managed Service Identity VM extension does not currently support the ability to export its schema to a resource group template. As a result, the generated template does not show configuration parameters to enable Managed Service Identity on the resource. These sections can be added manually by following the examples in [Configure a VM Managed Service Identity by using a template](msi-qs-configure-template-windows-vm.md).

When the schema export functionality becomes available for the MSI VM extension, it will be listed in [Exporting Resource Groups that contain VM extensions](~/articles/virtual-machines/windows/extensions-export-templates.md#supported-virtual-machine-extensions).

### Configuration blade does not appear in the Azure portal

If the VM Configuration blade does not appear on your VM, then MSI has not been enabled in the portal in your region yet.  Check again later.  You can also enable MSI for your VM using [PowerShell](msi-qs-configure-powershell-windows-vm.md) or the [Azure CLI](msi-qs-configure-cli-windows-vm.md).

### Cannot assign access to virtual machines in the Access Control (IAM) blade

If **Virtual Machine** does not appear in the Azure portal as a choice for **Assign access to** in **Access Control (IAM)** > **Add permissions**, then Managed Service Identity has not been enabled in the portal in your region yet. Check again later.  You can still select the Managed Service Identity for the role assignment by searching for the MSI’s Service Principal.  Enter the name of the VM in the **Select** field, and the Service Principal appears in the search result.

### VM fails to start after being moved from resource group or subscription

If you move a VM in the running state, it continues to run during the move. However, after the move, if the VM is stopped and restarted, it will fail to start. This issue happens because the VM is not updating the reference to the MSI identity and continues to point to it in the old resource group.

**Workaround** 
 
Trigger an update on the VM so it can get correct values for the MSI. You can do a VM property change to update the reference to the MSI identity. For example, you can set a new tag value on the VM with the following command:

```azurecli-interactive
 az  vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
This command sets a new tag "fixVM" with a value of 1 on the VM. 
 
By setting this property, the VM updates with the correct MSI resource URI, and then you should be able to start the VM. 
 
Once the VM is started, the tag can be removed by using following command:

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```

## Known issues with User Assigned MSI *(Private Preview Feature)*

- The only way to remove all user assigned MSIs is by enabling the system assigned MSI. 
- Provisioning of the VM extension to a VM might fail due to DNS lookup failures. Restart the VM, and try again. 
- Azure CLI: `Az resource show` and `Az resource list` will fail on a VM with a user assigned MSI. As a workaround, please use `az vm/vmss show`
- Azure Storage tutorial is only available in Central US EUAP at the moment. 
- When a User Assigned MSI is granted access to a resource, the IAM blade for that resource shows "Unable to access data." As a workaround, use the CLI to view/edit role assignments for that resource.
- Creating a user assigned MSI with an underscore in the name, is not supported.
- When adding a second user assigned identity, the clientID might not be available to requests tokens for it. As a mitigation, restart the MSI VM extension with the following two bash commands:
 - `sudo bash -c "/var/lib/waagent/Microsoft.ManagedIdentity.ManagedIdentityExtensionForLinux-1.0.0.8/msi-extension-handler disable"`
 - `sudo bash -c "/var/lib/waagent/Microsoft.ManagedIdentity.ManagedIdentityExtensionForLinux-1.0.0.8/msi-extension-handler enable"`
- The VMAgent on Windows does not currently support User Assigned MSI. 
- Assigning a role to an MSI to access a resource currently doesn't require special permissions. 
- When a VM has a user assigned MSI but no system assigned MSI, the portal UI will show MSI as enabled. To enable the system assigned MSI, use an Azure Resource Manager template, an Azure CLI, or an SDK.
