---
title: "Enabling RDP Access to an Azure VM with Microsoft Entra ID Users and MFA"
date: 2026-06-19
categories: [Azure, Compute, IAM]
tags: [azure, entra-id, rdp, mfa, virtual-machines, rbac, security]
image:
  path: /assets/img/posts/2026-06-19-RDP-Access-Azure-VM-Microsoft-Entra-ID-MFA.png
---
# How to enable RDP access to an Azure VM with Microsoft Entra ID users and MFA

In this article I explain how I configured a Windows virtual machine in Azure to allow RDP sign-ins with users from the Microsoft Entra ID tenant. The goal was for these accounts to log in without creating local users:

- `ana.perez@example.com`
- `carlos.mora@example.com`

The VM used was `vm-demo`, located in the `rg-demo` resource group. Beyond installing the extension and assigning the correct roles, I had to solve a subtle issue related to the device name and RDP web authentication.

## 1. Select the subscription

First I selected the subscription that contained the VM:

```powershell
az account set --subscription 00000000-1111-2222-3333-444444444444
```

Before moving on, I confirmed the VM, its resource group, operating system, state, and managed identity.

## 2. Enable the VM managed identity

Entra ID sign-in requires the VM to have a system-assigned managed identity:

```powershell
az vm identity assign `
  --resource-group rg-demo `
  --name vm-demo
```

In this case, the identity was already enabled.

## 3. Install the AADLoginForWindows extension

The VM needs the official extension that integrates Windows sign-in with Microsoft Entra ID:

```powershell
az vm extension set `
  --resource-group rg-demo `
  --vm-name vm-demo `
  --publisher Microsoft.Azure.ActiveDirectory `
  --name AADLoginForWindows
```

I then verified that provisioning had completed with `Succeeded`:

```powershell
az vm extension list `
  --resource-group rg-demo `
  --vm-name vm-demo `
  --output table
```

The extension also confirmed that the device was securely joined to Entra ID.

## 4. Assign sign-in permissions

Installing the extension does not automatically grant access. Each user needs one of these Azure RBAC roles:

- `Virtual Machine User Login`: allows standard user sign-in.
- `Virtual Machine Administrator Login`: allows administrative sign-in.

To keep things least-privilege, I assigned `Virtual Machine User Login` directly on the VM:

```powershell
$scope = "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/rg-demo/providers/Microsoft.Compute/virtualMachines/vm-demo"

az role assignment create `
  --assignee-object-id 11111111-2222-3333-4444-555555555555 `
  --assignee-principal-type User `
  --role "Virtual Machine User Login" `
  --scope $scope

az role assignment create `
  --assignee-object-id 66666666-7777-8888-9999-aaaaaaaaaaaa `
  --assignee-principal-type User `
  --role "Virtual Machine User Login" `
  --scope $scope
```

Using object IDs avoids ambiguity and makes the assignment work even when the CLI cannot resolve a UPN correctly.

## 5. Verify Entra ID join

I ran `dsregcmd /status` inside the VM using Run Command:

```powershell
az vm run-command invoke `
  --resource-group rg-demo `
  --name vm-demo `
  --command-id RunPowerShellScript `
  --scripts "dsregcmd /status"
```

The important values were:

```text
DeviceAuthStatus : SUCCESS
TpmProtected     : YES
IsDeviceJoined   : YES
TenantId         : 99999999-aaaa-bbbb-cccc-dddddddddddd
```

That confirmed the extension, the device, and the tenant were aligned.

## 6. First attempt: classic authentication

At first I used an RDP file with the traditional format:

```text
full address:s:203.0.113.10
username:s:AzureAD\ana.perez@example.com
targetisaadjoined:i:1
authentication level:i:2
prompt for credentials:i:1
```

RDP still returned `Your credentials did not work`.

I also found a persistent credential for another user associated with the IP:

```powershell
cmdkey /list
cmdkey /delete:TERMSRV/203.0.113.10
```

Removing it was necessary to stop the client from reusing `waradmin`, but it did not solve the issue by itself.

## 7. Diagnose where sign-in was failing

I checked security and Entra-related events inside the VM. Walter's attempt did not appear in event `4625`, which showed that the RDP client was failing during pre-authentication and never reaching Windows with the submitted credentials.

The VM was, however, recording multiple automated Internet attempts against names like `ADMINISTRATOR`, `SYSTEM`, and `BACKUP`. That confirmed that exposing RDP directly to the Internet is risky and needs extra controls.

## 8. Switch to Entra web authentication

To support MFA and Conditional Access, I changed the RDP file to use the web authentication flow:

```text
full address:s:vm-demo.eastus2.cloudapp.azure.com
username:s:ana.perez@example.com
enablerdsaadauth:i:1
targetisaadjoined:i:1
authentication level:i:2
prompt for credentials:i:1
```

The key property is:

```text
enablerdsaadauth:i:1
```

With that setting, RDP opens the Microsoft web sign-in experience. The user was able to enter the account and complete the second factor.

## 9. The device identifier error

After 2FA, this error appeared:

```text
AADSTS293004: The target-device identifier in the request
vm-demo.eastus2.cloudapp.azure.com was not found in the tenant.
```

Authentication was already working, but Entra was trying to locate a device whose name matched the public FQDN. The device registered in the tenant was actually named `vm-demo`.

Changing the file to use an IP address was not valid either: RDP web authentication requires a NetBIOS name or an FQDN, not an IP address.

## 10. Make the device name resolve to the public IP

The solution was to use the exact name registered in Entra inside the RDP file:

```text
full address:s:vm-demo
```

On the client machine I added a local name resolution entry so that name would point to the VM's public IP. This requires an elevated shell:

```powershell
Add-Content `
  -LiteralPath "$env:WINDIR\System32\drivers\etc\hosts" `
  -Value "`r`n203.0.113.10`tvm-demo" `
  -Encoding ascii

ipconfig /flushdns
```

The resulting entry was:

```text
203.0.113.10    vm-demo
```

The final RDP file looked like this:

```text
full address:s:vm-demo
username:s:ana.perez@example.com
enablerdsaadauth:i:1
targetisaadjoined:i:1
authentication level:i:2
prompt for credentials:i:1
```

When I opened it again, RDP showed the web sign-in page, asked for the second factor, and finally granted access.

## Result

The VM ended up accepting all of these simultaneously:

- Local Windows accounts, using `VMNAME\user`.
- Entra ID users authorized through Azure RBAC.
- Web authentication with MFA and Conditional Access policies.

These mechanisms are not mutually exclusive. The Entra extension adds an alternative authentication path without removing the existing local accounts.

## Lessons learned

1. Installing `AADLoginForWindows` is not enough: the RBAC sign-in roles must also be assigned.
2. The role must be assigned to the user, group, or identity at a scope that includes the VM.
3. Persistent RDP credentials can make the client use a different account than the one specified in the file.
4. If the attempt does not show up in VM events, it is probably failing in the client or during pre-authentication NLA.
5. `enablerdsaadauth:i:1` enables the Microsoft web flow, MFA, and Conditional Access.
6. With web authentication, the name used by RDP must correctly identify the device registered in Entra.
7. An IP address is not a valid identifier for this flow.

## Security recommendations

During troubleshooting I observed automated attempts against the public RDP port. For a production environment it is better not to expose `3389` to the Internet and instead use one or more of these measures:

- Azure Bastion.
- VPN or private connectivity.
- Network Security Groups restricted to known admin IPs.
- Just-In-Time VM Access from Microsoft Defender for Cloud.
- MFA and Conditional Access policies.
- RBAC roles with the minimum required privilege.

Finally, an `.rdp` file is plain text. It can contain names, addresses, and connection options, but it should not include passwords, tokens, or other secrets.

## Bonus: allow a user to start and deallocate the VM only

I also needed to let `ana.perez@example.com` manage the VM state in the portal, but only in a very narrow way: she should be able to view the resource group, select the VM, and start or deallocate it. She should not be able to create, edit, delete, or modify anything else in the resource group.

The approach was the same least-privilege pattern I used for the other user:

- Assign `Reader` at the `rg-demo` scope so the user can browse the resource group and open the VM.
- Create a custom role that allows only `read`, `instanceView/read`, `start/action`, and `deallocate/action` on the VM.
- Assign that custom role directly at the VM scope.

The custom role definition looked like this:

```json
{
  "Name": "VM Start Deallocate Operator",
  "IsCustom": true,
  "Description": "Can read, start, and deallocate an Azure virtual machine. Cannot create, modify, or delete resources.",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/instanceView/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-1111-2222-3333-444444444444"
  ]
}
```

In practice, that means the user can open the VM blade and trigger power-state changes, but cannot change the VM configuration, create new resources, or delete existing ones in the resource group.

## Execute this with your Agents

If I had to hand this workflow to an agent, I would compress it into a tiny operating playbook that favors direct action over explanation:

```text
Goal: enable Entra RDP sign-in to a Windows Azure VM, then give one user only VM start/deallocate rights.

Inputs:
- subId
- tenantId
- rgName
- vmName
- publicIp
- deviceName
- rdpUsers[]
- stateUser

Rules:
1. Reader on rgName lets a user browse the resource group.
2. Virtual Machine User Login or Virtual Machine Administrator Login must be assigned at VM scope for RDP.
3. Do not use the public IP as the Entra device identifier.
4. For web RDP auth, full address must be deviceName or a resolvable FQDN.
5. If the error is AADSTS293004, fix the RDP name to match the Entra device name.
6. If classic RDP fails, remove cached TERMSRV creds and switch to web auth.
7. For start/deallocate only, use Reader on the RG plus a custom VM role with only:
   - Microsoft.Compute/virtualMachines/read
   - Microsoft.Compute/virtualMachines/instanceView/read
   - Microsoft.Compute/virtualMachines/start/action
   - Microsoft.Compute/virtualMachines/deallocate/action
8. Never assign Contributor, Virtual Machine Contributor, or Owner for this use case.

Sequence:
1. az account set --subscription <subId>
2. Ensure managed identity exists on the VM.
3. Install AADLoginForWindows.
4. Verify the extension succeeded.
5. Assign the RDP login role to each allowed user at VM scope.
6. Run dsregcmd /status to confirm Entra join.
7. Create the RDP file with:
   - full address:s:<deviceName>
   - username:s:<upn>
   - enablerdsaadauth:i:1
   - targetisaadjoined:i:1
8. Map deviceName to publicIp on the client if needed.
9. Flush DNS.
10. Open RDP.

Failure triage:
- "Your credentials did not work" -> clear TERMSRV creds, use web auth.
- AADSTS293004 -> mismatch between full address and Entra device name.
- 0x1080 -> use FQDN or NetBIOS name, not IP.
- No VM security event -> client-side or pre-auth failure.
```

That is the shortest useful version of the workflow: assign the right RBAC roles, use the right RDP auth mode, resolve the device name correctly, and keep the state-only access path locked down to exactly start and deallocate.
