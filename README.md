<p align="center">
  <img src="Assets/AppLogo.png" alt="IWService Health Checker logo" width="120">
</p>

<h1 align="center">IWService Health Checker</h1>

<p align="center">
  A Windows troubleshooting tool for validating Microsoft Intune IWService discovery, authentication, and read access.
</p>

## Overview

IWService Health Checker helps investigate whether a Windows device and signed in user can locate and communicate with the tenant specific Microsoft Intune IWService endpoint.

The tool uses Windows Web Account Manager, known as WAM, to request tokens and performs read only checks against the IWService Applications and Devices paths. It also shows the tenant routing information, token metadata, certificate details, response status, latency, returned item counts, and any raw error returned during the check.

This is an unofficial community troubleshooting tool. It is not a Microsoft product and is not supported by Microsoft.

## What the tool checks

| Check | Purpose |
|---|---|
| WAM sign in | Requests a token through the Windows account already connected to the device |
| Discovery only | Resolves the tenant specific IWService URL without calling the Applications or Devices path |
| Check Applications | Calls the IWService Applications path and reports the HTTP result and number of returned applications |
| Check Devices | Calls the IWService Devices path and reports the HTTP result and number of returned devices |
| Certificate diagnostics | Locates the Intune MDM certificate and records its subject and thumbprint |
| Routing diagnostics | Shows how the IWService endpoint was discovered and which tenant specific ASU endpoint was selected |

All IWService requests made by this tool are read only. The tool does not create, update, delete, retire, or wipe Intune objects.

## Requirements

1. Windows 10 version 2004 or later, or Windows 11
2. An x64 device
3. The [.NET 8 x64 Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
4. A work or school account available through Windows WAM
5. Network access to the required Microsoft sign in and Intune service endpoints
6. For the best discovery results, an Intune enrolled device with its MDM certificate and Intune Management Extension logs available

Administrator permissions are normally not required. Access to the returned Intune data still depends on the selected account and token context.

## Download and run

1. Download the latest release archive.
2. Extract the complete archive to a local folder.
3. Keep all included files and folders together.
4. Start `IwServiceHealthChecker.exe`.
5. Windows SmartScreen may display a warning because the current executable is not digitally signed. Review the downloaded file and publisher source before choosing to run it.

Do not start the executable directly from inside the ZIP file.

## Using the tool

### 1. Select a WAM token profile

The application contains two token profiles.

| Profile | Usage |
|---|---|
| Company Portal to IWService | Use this profile for IWService discovery and the Applications and Devices checks |
| Intune portal extension to Microsoft Intune | Provides a Microsoft Intune token profile for token comparison and diagnostics. IWService checks are not enabled for this profile |

### 2. Sign in

Select **Sign in**. WAM first attempts silent token acquisition. When silent acquisition is not possible, Windows can display an interactive account prompt.

The tool does not request or store your password.

### 3. Run a check

Select one of the following actions.

| Action | Result |
|---|---|
| Discovery only | Resolves the IWService URL and reports the discovery source |
| Check Applications | Resolves the endpoint, sends a read request to Applications, and counts returned items |
| Check Devices | Resolves the endpoint, adds the device ID from the WAM token when available, sends a read request to Devices, and counts returned items |

### 4. Review or copy the result

The result view can include:

1. Check type
2. Tenant and account
3. Tenant specific IWService URL
4. HTTP status and latency
5. Applications or devices returned
6. Discovery source
7. MDM certificate thumbprint
8. Device ID
9. Token audience, client ID, and expiry
10. Raw error details
11. Log file location

Use **Copy result** for the selected result. Use **Copy diagnostics** for a broader troubleshooting summary.

## IWService discovery

The tool can obtain the tenant specific IWService endpoint from several local and service based sources:

1. A manually supplied IWService URL or host override
2. Certificate based Intune LocationService discovery using the device MDM certificate
3. Recent Intune Management Extension logs containing a SideCar or IWService ASU endpoint
4. Bearer token based Intune LocationService discovery
5. Intune enrollment information stored below the local OMADM registry path

When an Intune Management Extension SideCar endpoint is found, the tool converts the tenant ASU host into the corresponding IWService path.

The tool intentionally does not call the generic `fef.manage.microsoft.com` endpoint when no tenant specific ASU endpoint can be discovered. This avoids testing an endpoint that may not represent the device tenant routing path.

## Optional endpoint override

A complete IWService URL or tenant host can be entered in the override field.

Example:

```text
https://fef.amsub0102.manage.microsoft.com/TrafficGateway/TrafficRoutingService/IWService/StatelessIWService
```

Use an override only when the expected tenant endpoint is already known. An incorrect host can produce misleading authentication or routing results.

## Logs and diagnostics

Logs are written to:

```text
%LOCALAPPDATA%\IwServiceHealthChecker\Logs
```

Each run creates a log named similar to:

```text
IwServiceHealthChecker_20260712_103000.log
```

The log records token metadata, including the profile, account, tenant ID, audience, client ID, requested resource, and token expiry. It does not record the access token value.

Logs and copied diagnostics can contain tenant IDs, account names, device IDs, service URLs, certificate thumbprints, and raw server errors. Review and redact this information before posting diagnostics publicly.

## Common results

### WAM account provider not found

Confirm that the Windows session has a connected work or school account. On an Entra joined device, also verify the device registration and Primary Refresh Token state with:

```powershell
dsregcmd /status
```

### No tenant ASU discovered

Confirm that the device is Intune enrolled and that the Intune MDM certificate is present. The tool can also use recent Intune Management Extension logs. When the correct tenant URL is known, test it with the endpoint override.

### HTTP 401 or 403

Confirm that the Company Portal to IWService profile is selected and that the correct organizational account was used. A valid token does not guarantee that every account can read every IWService path.

### Certificate discovery failed

The device may not be enrolled, the enrollment certificate may be missing, or the certificate may not have an accessible private key. Review the log for the certificate stores and registry hints checked by the tool.

### Network, TLS, or timeout errors

Check proxy configuration, TLS inspection, firewall rules, and access to Microsoft authentication and Intune endpoints. Compare the requested URL and raw error in the result details.

## Current package

The current repository package contains the compiled x64 application and its required runtime files. It does not contain the C# source project.

Application version: `1.0.0`

Target framework: `.NET 8`

Windows target: `10.0.19041.0`

Windows App SDK: `1.6.250205002`

## Responsible use

Use this tool only with devices, accounts, and tenants you are authorized to troubleshoot. IWService is an internal Microsoft Intune service surface and can change without notice. A successful result confirms the tested path at that moment; it does not guarantee the health of every Company Portal or Intune workflow.

## License

This project is licensed under the [MIT License](LICENSE).

Copyright 2026 Rudy Ooms

