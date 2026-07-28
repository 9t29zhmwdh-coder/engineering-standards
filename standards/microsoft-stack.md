# Microsoft Stack Guidelines

Rules that apply when building against M365, Azure or Windows. Azure resource design, naming, monitoring and policy are covered separately in [`azure-integration.md`](azure-integration.md); this document holds the client-side and build-side rules that sit outside it.

## 1. M365 Authentication (Graph API)

- Use the Microsoft Graph SDK rather than hand-rolling authentication
- OAuth 2.0 against Microsoft Entra ID, never Basic Auth
- MFA enforced via Entra ID
- Have an explicit token refresh strategy; a tool that silently stops working after an hour is an auth bug, not a user error

## 2. Azure Deployment

- Infrastructure as code (Bicep, ARM templates or Terraform), never portal click-ops as the source of truth
- Secrets in Azure Key Vault
- Managed Identity wherever the service supports it, in preference to a service principal with a client secret
- Staging before production

## 3. Windows and ARM Targeting

- Build and test for both x86 and ARM
- Visual Studio for desktop applications, VS Code for cloud and web work
- .NET 6 or later (LTS)
- PowerShell 7 or later, so scripts stay cross-platform

The same reasoning applies on Apple platforms, where the release artifact is a universal binary; see [`ci-cd.md`](ci-cd.md) section 6. Building on one architecture and shipping only that architecture is the default behaviour of most toolchains, which is exactly why it has to be checked rather than assumed.

## 4. Package Management

- NuGet for .NET
- npm for Node.js
- pip for Python
- GitHub Packages for private packages

Pin build and lint tooling to exact versions regardless of ecosystem, per [`ci-cd.md`](ci-cd.md) section 2.
