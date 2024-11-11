---
title: Use personal access tokens
titleSuffix: Azure DevOps
ms.custom: ai-video-demo
ai-usage: ai-assisted
description: Learn how to create and manage personal access tokens (PATs) as alternate passwords to authenticate to Azure DevOps.
ms.subservice: azure-devops-security
ms.assetid: d980d58e-4240-47c7-977c-baaa7028a1d8
ms.topic: how-to
ms.author: chcomley
author: chcomley
ms.date: 10/24/2024
monikerRange: '<= azure-devops'
---

# Use personal access tokens

[!INCLUDE [version-lt-eq-azure-devops](../../includes/version-lt-eq-azure-devops.md)]

A Personal Access Token (PAT) serves as an alternative password for authenticating into Azure DevOps. This PAT identifies you and determines your accessibility and scope of access. Therefore, treat PATs with the same level of caution as passwords.

When you use Microsoft tools, your Microsoft account (MSA) or Microsoft Entra ID is a recognized and supported method. If you use non-Microsoft tools that don't support Microsoft or Microsoft Entra accounts, or if you prefer not to share your primary credentials with these tools, PATs are a suitable alternative.

This article guides you through creating, using, modifying, and revoking PATs for Azure DevOps.

You can manage PATs using the following methods:
* **User interface (UI):** Through user settings, as detailed in this article
* [**PAT Lifecycle Management API**](manage-personal-access-tokens-via-api.md)

To establish PATs for non-Microsoft tools, you can use [Git credential managers](../../repos/git/set-up-credential-managers.md) or generate them manually. We recommend reviewing our [authentication guidance](../../integrate/get-started/authentication/authentication-guidance.md) to choose the appropriate authentication mechanism. PATs provide a straightforward alternative for smaller projects that don't require an extensive solution. Without a credential manager, users input their credentials each time.

## Prerequisites

- **Permissions:** 
  - Have permission to access and modify your user settings where PATs are managed.
    - **Check permissions:** To check your permissions, do either of the following processes in Azure DevOps:
      - Go to **your profile** and select **User settings** > **Personal access tokens**. If you can see and manage your PATs here, you have the necessary permissions.
      - Go to your project and select **Project settings** > **Permissions**. Find your user account in the list and check the permissions assigned to you. Look for permissions related to managing tokens or user settings.
  - If your [organization has policies in place](manage-pats-with-policies-for-administrators.md), an Azure DevOps Administrator might need to grant you specific permissions or add you to an allowlist to create and manage PATs.
  - Depending on the tasks you want to perform with the PAT, you might need additional permissions. For example: 
    - Build: Read & execute
    - Code: Read, write, & manage
    - Environment: Read & manage
    - Project and team: Read, write, & manage
    - Variable groups: Read & create
- **Access levels:** Have at least Basic access.
- **Security best practices:** Familiarize yourself with [security best practices](../security/security-best-practices.md) for managing PATs, such as using them only when necessary and regularly rotating them.

[!INCLUDE [personal-access-tokens](../../repos/git/includes/personal-access-tokens.md)]

## Changes to format

As of July 2024, we significantly changed the format of PATs issued by Azure DevOps. These changes provide more security benefits and improve secret detection tooling available through our partner offerings, like [GitHub Advanced Security for Azure DevOps](https://devblogs.microsoft.com/devops/github-advanced-security-for-azure-devops-public-preview-starts-now/). This new PAT format follows the recommended format across all Microsoft products. The inclusion of more identifiable bits improves the false positive detection rate of these secret detection tools and enables us to mitigate detected leaks faster.

Key changes:
* **Increased token length:** The new tokens are now **84** characters long, with 52 characters being randomized data. This increased length improves overall entropy, making the tokens more resistant to potential brute force attacks.
* **Fixed signature:** Tokens issued by our service include a fixed `AZDO` signature at positions 76-80.

Action required:
* **Regenerate existing PATs:** We strongly recommend regenerating all PATs currently in use to take advantage of these security enhancements.
* **Integrator support:** Integrators should update their systems to accommodate both the new and existing token lengths.

> [!IMPORTANT]
> Both formats remain valid for the foreseeable future, but we **actively encourage customers to transition to the new 84-character format**. As adoption of the new format increases, we consider retiring the older 52-character format and all tokens issued in that style.

## Related articles

* [Learn about security, authentication, and authorization](../security/about-security-identity.md)
* [Review default permissions and access for Azure DevOps](../security/permissions-access.md)
* [Revoke user PATs (for admins)](admin-revoke-user-pats.md)
* [Manage service principals and managed identities in Azure DevOps](../../integrate/get-started/authentication/service-principal-managed-identity.md)

::: moniker range="azure-devops"

## FAQs

### Q: Why can't I edit or regenerate a PAT scoped to a single organization?

A: Ensure you're signed into the organization where your PAT is scoped. You can ***view*** all of your PATs while signed into any organization in the same Microsoft Entra ID, but you can only ***edit*** organization-scoped tokens when you're signed into the organization to which they're scoped.

### Q: What happens to a PAT if a user account is disabled?

A: When a user is removed from Azure DevOps, the PAT invalidates within 1 hour. If your organization is connected to Microsoft Entra ID, the PAT also invalidates in Microsoft Entra ID, as it belongs to the user. We recommend rotating the PAT to another user or service account to keep services running.

### Q: Is there a way to renew a PAT via REST API?

A: Yes, there's a way to renew, manage, and create PATs using our [PAT Lifecycle Management APIs](manage-personal-access-tokens-via-api.md). For more information, see [Manage PATs using REST API](manage-personal-access-tokens-via-api.md) and [FAQs](manage-personal-access-tokens-via-api.md#q-how-can-i-regeneraterotate-pats-through-the-api-i-saw-that-option-in-the-ui-but-i-dont-see-a-similar-method-in-the-api).

### Q: Can I use basic auth with all Azure DevOps REST APIs?

A: No. You can use basic auth with most Azure DevOps REST APIs, but [organizations and profiles](/rest/api/azure/devops/) only support [OAuth](../../integrate/get-started/authentication/oauth.md). For more information, see [Manage PATs using REST API](manage-personal-access-tokens-via-api.md).

### Q: What happens if I accidentally check my PAT into a public repository on GitHub?

A: Azure DevOps scans for PATs checked into public repositories on GitHub. When we find a leaked token, we immediately send a detailed email notification to the token owner and log an event in your Azure DevOps organization's [audit log](../audit/azure-devops-auditing.md#review-audit-log). Unless you disabled the *Automatically revoke leaked personal access tokens* policy, we immediately revoke the leaked PAT. We encourage affected users to mitigate the issue by [revoking the leaked token](use-personal-access-tokens-to-authenticate.md#revoke-a-pat) and replacing it with a new token.

For more information, see [Revoke leaked PATs automatically](manage-pats-with-policies-for-administrators.md#revoke-leaked-pats-automatically).

### Q: Can I use a personal access token as an ApiKey to publish NuGet packages to an Azure Artifacts feed using the dotnet/nuget.exe command line?

A: No. Azure Artifacts doesn't support passing a personal access token as an ApiKey. When using a local development environment, we recommended installing the [Azure Artifacts Credential Provider](https://github.com/microsoft/artifacts-credprovider) to authenticate with Azure Artifacts. For more information, see the following examples: [dotnet](../../artifacts/nuget/dotnet-exe.md), [NuGet.exe](../../artifacts/nuget/publish.md).
If you want to publish your packages using Azure Pipelines, use the [NuGet Authenticate](/azure/devops/pipelines/tasks/reference/nuget-authenticate-v1) task to authenticate with your feed [example](../../pipelines/artifacts/nuget.md).

### Q: Why did my PAT stop working?

A: PAT authentication requires you to regularly sign into Azure DevOps using the full authentication flow. Signing in once every 30 days is sufficient for many users, but you might need to sign in more frequently depending on your Microsoft Entra configuration. If your PAT stops working, first try signing into your organization and complete the full authentication prompt. If your PAT still doesn't work, check if it expired.

### Q: How do I create access keys that aren't tied to a specific person for deployment purposes?

A: In Azure DevOps, you can create access keys that aren't tied to a specific person by using Service Principals or Manage Identities. For more information, see [Manage service connections](../../pipelines/library/service-endpoints.md), [Use Azure Key Vault secrets in Azure Pipelines](../../pipelines/release/azure-key-vault.md).

::: moniker-end
