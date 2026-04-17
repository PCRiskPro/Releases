# PCRiskPro RMM Deployment Guide

**For IT administrators, Managed Service Providers, and Cyber Essentials assessors deploying PCRiskPro across multiple endpoints.**

Version 1.0 | April 2026 | Applies to PCRiskPro 1.9.9-beta10 and later.

---

## 1. About this guide

This guide covers deployment of PCRiskPro in environments with more than a handful of endpoints — typically 10 to 500 machines managed by an MSP or internal IT team. It describes the commands, scripts, and licensing model for:

- Pushing PCRiskPro silently via any major RMM platform
- Automating scans across a client fleet
- Collecting and aggregating results into consolidated reports
- Operating within PCRiskPro's licensing model at scale

**Target audience:**

- MSP technicians deploying PCRiskPro to client endpoints
- Internal IT teams rolling out to corporate desktops
- Cyber Essentials assessors conducting multi-site audits

**What is NOT covered:**

- End-user installation (see the main [README](../README.md))
- Source code, internal architecture, or SDK integration
- Custom development or OEM licensing (contact sales@pcriskpro.com)

---

## 2. Feature Availability Matrix

PCRiskPro is in staged rollout. Some features listed in this guide are **available today**; others are **planned for the v2.0.0 commercial release**. Each section below is labelled accordingly.

| Feature | Available now (v1.9.9-beta10) | Coming in v2.0.0 |
|---------|-------------------------------|------------------|
| Signed installer (Certum OV) | ✅ | ✅ |
| Silent install via `/SILENT` and `/VERYSILENT` flags | ✅ | ✅ |
| GUI-triggered scans and exports | ✅ | ✅ |
| HTML/JSON/Excel export formats | ✅ | ✅ |
| Headless CLI scan mode (`pcriskpro-cli.exe`) | ❌ | ✅ |
| Report Aggregator desktop application | ❌ | ✅ |
| White-label branding (MSP tier) | ❌ | ✅ |
| Lemon Squeezy licence activation | ❌ | ✅ |
| 100-endpoint MSP licence tier | ❌ | ✅ |

**If you are planning a deployment today**, you can use the silent installer for distribution but scans will still require local user action. If you are planning a deployment for the v2.0.0 commercial release (target Q2 2026), all features will be available.

---

## 3. Architecture Overview

PCRiskPro is designed for **decentralised scanning with centralised reporting**. Each endpoint performs its assessment locally; results aggregate later.

```
  ┌────────────────────────────────────────────────────────────────┐
  │                     Your RMM Platform                          │
  │  (ConnectWise Automate, Datto RMM, NinjaOne, Kaseya, etc.)     │
  └──────────────────────┬─────────────────────────────────────────┘
                         │ (1) Push silent installer
                         │ (2) Schedule or trigger CLI scan
                         │ (3) Collect JSON output
                         ▼
  ┌────────────┐      ┌────────────┐       ┌────────────┐
  │ Endpoint 1 │      │ Endpoint 2 │  ...  │ Endpoint N │
  │ PCRiskPro  │      │ PCRiskPro  │       │ PCRiskPro  │
  └─────┬──────┘      └─────┬──────┘       └─────┬──────┘
        │ JSON scan         │ JSON scan          │ JSON scan
        │ output            │ output             │ output
        ▼                   ▼                    ▼
  ┌─────────────────────────────────────────────────────────┐
  │         Centralised Storage (your file server,          │
  │         UNC share, or RMM artifact collection)          │
  └──────────────────────┬──────────────────────────────────┘
                         │
                         ▼
  ┌──────────────────────────────────────────────────────────┐
  │   PCRiskPro Report Aggregator (MSP technician's machine) │
  │   → Consolidated multi-endpoint HTML/Excel report        │
  │   → White-labelled with your branding                    │
  └──────────────────────────────────────────────────────────┘
```

**No server-side infrastructure required.** PCRiskPro does not upload scan data to our servers. All results remain under your control on your own network storage.

---

## 4. Licensing for MSP Deployment

### 4.1 MSP / Partner Tier Overview

The **MSP / Partner** tier is the only licence that permits commercial use and multi-client deployment.

| Licence feature | Solo (£95/yr) | SME (£795/yr) | MSP / Partner (£1,995/yr) |
|-----------------|---------------|---------------|----------------------------|
| Max activated devices | 1 | 10 | 100 |
| Internal business use | ✅ | ✅ | ✅ |
| Commercial resale / consulting | ❌ | ❌ | ✅ |
| White-label branding in reports | ❌ | ❌ | ✅ (v2.0.0) |
| Report Aggregator tool | ❌ | ❌ | ✅ (v2.0.0) |
| RMM deployment guide access | ✅ | ✅ | ✅ |

**Note:** Solo and SME tiers installed on client machines by an MSP constitute a breach of the End User Licence Agreement. MSPs must hold an MSP/Partner tier to legitimately offer PCRiskPro as a managed service.

### 4.2 Activation Model (v2.0.0+)

When v2.0.0 ships, PCRiskPro will use **one licence key with N activations** enforced server-side via Lemon Squeezy. The MSP holds a single key; each endpoint consumes one activation slot.

- First-time install activates the licence against the endpoint's machine hash
- Subsequent launches validate the cached token offline
- When a client machine is retired, the MSP deactivates it via the Lemon Squeezy customer portal to free a slot
- If the 100-endpoint cap is reached, additional activations can be purchased (MSP Plus tier — contact sales@pcriskpro.com)

### 4.3 Current Beta Licensing

In the current beta (v1.9.9-beta10), there are no enforced activation limits. Commercial licensing infrastructure activates with v2.0.0. Do not deploy to production client environments using the beta — wait for the stable release.

---

## 5. Silent Installation (Available Now)

The PCRiskPro installer is built with Inno Setup and supports all standard silent-install flags.

### 5.1 Command-Line Flags

| Flag | Purpose |
|------|---------|
| `/SILENT` | Suppress wizard UI, show only a progress bar |
| `/VERYSILENT` | Fully unattended — no UI whatsoever |
| `/SUPPRESSMSGBOXES` | Suppress all message boxes (use with `/SILENT`) |
| `/NORESTART` | Prevent automatic system restart after install |
| `/LOG=<path>` | Write detailed install log to the specified path |
| `/LOG` | Write log to temp directory with auto-generated name |
| `/DIR=<path>` | Override default install directory |
| `/TASKS=<tasklist>` | Pre-select optional tasks (e.g., `desktopicon`) |
| `/TASKS=""` | Deselect all optional tasks |
| `/NOCANCEL` | Prevent user from cancelling during silent install |
| `/LOADINF=<path>` | Load install settings from an .inf file |

### 5.2 Example — Fully Unattended Install

```cmd
PCRiskPro_Setup_1.9.9.exe /VERYSILENT /NORESTART /LOG="C:\ProgramData\PCRiskPro\install.log"
```

This installs silently with no UI, no auto-restart, and logs the install process for troubleshooting.

### 5.3 Example — Install with Custom Directory

```cmd
PCRiskPro_Setup_1.9.9.exe /VERYSILENT /DIR="C:\PCRiskPro" /TASKS=""
```

Installs to a non-default location with no desktop shortcut.

### 5.4 Verifying a Silent Install Succeeded

Check the exit code of the installer process:

| Exit code | Meaning |
|-----------|---------|
| 0 | Successful installation |
| 1 | Setup failed to initialise |
| 2 | User pressed Cancel (should not occur with `/NOCANCEL`) |
| 3 | Install failed during execution |
| 4 | Install cancelled by another process |
| 5 | Install requires restart (use with `/NORESTART`) |

PowerShell example to capture and verify:

```powershell
$proc = Start-Process -FilePath "PCRiskPro_Setup_1.9.9.exe" `
                      -ArgumentList "/VERYSILENT /NORESTART" `
                      -Wait -PassThru
if ($proc.ExitCode -eq 0) {
    Write-Host "PCRiskPro installed successfully"
} else {
    Write-Host "Install failed with exit code: $($proc.ExitCode)"
    Exit $proc.ExitCode
}
```

### 5.5 Verifying Installation Integrity

After installation, verify the binary was signed by PCRiskpro Limited:

```powershell
$installed = "${env:ProgramFiles}\PCRiskPro\PCRiskPro.exe"
$sig = Get-AuthenticodeSignature $installed
if ($sig.Status -eq "Valid" -and $sig.SignerCertificate.Subject -like "*PCRiskpro Limited*") {
    Write-Host "Signature valid"
} else {
    Write-Host "WARNING: Signature check failed"
    Exit 1
}
```

---

## 6. Headless CLI Scan Mode (v2.0.0 Preview)

**Availability:** Planned for v2.0.0 commercial release. The information below is the intended specification — subject to refinement during development.

### 6.1 Purpose

`pcriskpro-cli.exe` will be a second executable bundled alongside the main GUI that runs scans from the command line without opening any windows. This enables RMM platforms to trigger scans via scheduled tasks or remote commands.

### 6.2 Planned Command Structure

```cmd
pcriskpro-cli.exe --scan <module> --output <path> [options]
```

### 6.3 Planned Arguments (Subject to Change)

| Argument | Purpose |
|----------|---------|
| `--scan all` | Run all four modules (Data Discovery, App Security, Cyber Essentials, System Security) |
| `--scan data-discovery` | Run Data Discovery only |
| `--scan app-security` | Run App Security only |
| `--scan cyber-essentials` | Run Cyber Essentials only |
| `--scan system-security` | Run System Security only |
| `--output <path>` | Path for JSON result file |
| `--format json\|html\|both` | Output format (default: json) |
| `--quiet` | Suppress console output |
| `--license <key>` | Activate licence if not already activated |
| `--scan-path <path>` | For Data Discovery: path to scan |
| `--timeout <minutes>` | Maximum scan duration before abort (default: 30) |
| `--verbose` | Detailed logging to stderr |

### 6.4 Planned Exit Codes

| Exit code | Meaning |
|-----------|---------|
| 0 | Scan completed successfully, results written to output |
| 1 | Scan completed with findings above CRITICAL threshold |
| 2 | Invalid arguments |
| 3 | Licence not valid or activated |
| 4 | Scan timed out |
| 5 | Insufficient privileges (some checks require admin) |
| 10 | Internal scanner error |

Non-zero exit codes allow RMM platforms to alert when endpoints return critical findings.

### 6.5 Example Usage (v2.0.0)

```powershell
# Full scan to network share
pcriskpro-cli.exe --scan all --output "\\fileserver\scans\$env:COMPUTERNAME.json" --quiet

# Data Discovery only, specific path, verbose logging
pcriskpro-cli.exe --scan data-discovery --scan-path "C:\Users\$env:USERNAME\Documents" --output "C:\temp\scan.json" --verbose

# App Security scan with explicit timeout
pcriskpro-cli.exe --scan app-security --output "C:\temp\app-sec.json" --timeout 15
```

---

## 7. Report Aggregator (v2.0.0 Preview)

**Availability:** Planned for v2.0.0 commercial release.

### 7.1 Purpose

A standalone desktop application bundled with the MSP/Partner tier that reads a folder of JSON scan files produced by `pcriskpro-cli.exe` and generates a consolidated portfolio report across all endpoints.

### 7.2 Planned Capabilities

- Read 1–500 JSON scan files from a folder or UNC share
- Group endpoints by client or tag (inferred from folder structure or JSON metadata)
- Produce consolidated HTML report with:
  - Portfolio-level risk summary
  - Per-client breakdown
  - Per-endpoint detail (drill-down)
  - Trend analysis (comparing current vs previous scan cycles)
- Produce consolidated Excel workbook with sortable/filterable tables
- Apply MSP branding (logo, colours, company name) to all outputs
- Output standalone PDF variants for client delivery

### 7.3 Planned Workflow

1. Technician opens Report Aggregator
2. Selects source folder (e.g., `\\fileserver\pcriskpro\client-acme\2026-04\`)
3. Reviews imported endpoints (list view)
4. Applies branding (logo, primary colour, footer text)
5. Clicks **Generate Report** → HTML + Excel + PDF produced
6. Delivers to client as part of managed service

### 7.4 Licence Gating

Report Aggregator will validate the MSP/Partner tier licence at launch. Solo and SME tier licences will see an error message directing to the upgrade page. This prevents workaround use of cheaper tiers for commercial reselling.

---

## 8. RMM Platform Integration Examples

Each example below shows how to deploy PCRiskPro via a specific RMM. All examples assume the installer is hosted at the GitHub URL or your own internal repository.

### 8.1 ConnectWise Automate (formerly LabTech)

**Script structure:**

```powershell
# ConnectWise Automate script: PCRiskPro Silent Install
# Recommended schedule: one-time deployment, monthly scan refresh

$installerUrl = "https://github.com/PCRiskPro/Releases/releases/download/v1.9.9-beta10/PCRiskPro_Setup_1.9.9.exe"
$installerPath = "$env:TEMP\PCRiskPro_Setup.exe"
$licenseKey = "@LicenseKey@"  # ConnectWise Automate parameter

# Download
Invoke-WebRequest -Uri $installerUrl -OutFile $installerPath -UseBasicParsing

# Verify signature
$sig = Get-AuthenticodeSignature $installerPath
if ($sig.Status -ne "Valid") {
    Write-Error "Installer signature invalid"
    Exit 1
}

# Install silently
$proc = Start-Process -FilePath $installerPath `
                      -ArgumentList "/VERYSILENT /NORESTART /LOG=`"C:\ProgramData\PCRiskPro\install.log`"" `
                      -Wait -PassThru

# Clean up
Remove-Item $installerPath -Force

Exit $proc.ExitCode
```

Import this as a PowerShell script in Automate's Script Manager. Assign to target group and run.

### 8.2 Datto RMM

**Component XML structure** (for upload to Component Library):

```xml
<Component>
  <Name>PCRiskPro - Silent Install</Name>
  <Description>Installs PCRiskPro silently on endpoint</Description>
  <Category>Applications</Category>
  <Runtime>PowerShell</Runtime>
  <Script>
    # Same PowerShell as section 8.1
  </Script>
</Component>
```

Upload via ComStore → My Custom Components. Deploy via Automation Policy or on-demand.

### 8.3 NinjaOne

**Custom Script (PowerShell):**

NinjaOne's custom scripts run via the Policy Editor → Scheduled Scripts or Run Script on-demand.

```powershell
# NinjaOne Custom Script: Install PCRiskPro
# Variables:
#   - Set $licenseKey as a Custom Field in device settings

$installerUrl = "https://github.com/PCRiskPro/Releases/releases/download/v1.9.9-beta10/PCRiskPro_Setup_1.9.9.exe"
$installerPath = "$env:TEMP\PCRiskPro_Setup.exe"

try {
    Invoke-WebRequest -Uri $installerUrl -OutFile $installerPath -UseBasicParsing -ErrorAction Stop

    $args = "/VERYSILENT /NORESTART /LOG=`"C:\ProgramData\PCRiskPro\install.log`""
    $proc = Start-Process -FilePath $installerPath -ArgumentList $args -Wait -PassThru

    if ($proc.ExitCode -eq 0) {
        Write-Output "PCRiskPro installed successfully on $env:COMPUTERNAME"
    } else {
        throw "Installer returned exit code $($proc.ExitCode)"
    }
} catch {
    Write-Error "PCRiskPro install failed: $_"
    Exit 1
} finally {
    Remove-Item $installerPath -ErrorAction SilentlyContinue
}
```

### 8.4 Kaseya VSA

Kaseya's Agent Procedures support file download + execute pattern natively.

1. **Agent Procedures** → **New Procedure** → Name: `Install PCRiskPro`
2. Add step: **Get File** → URL: GitHub release URL → Save to: `#vAgentConfiguration.AgentTempDir#\PCRiskPro_Setup.exe`
3. Add step: **Execute Shell Command** → Command: `"#vAgentConfiguration.AgentTempDir#\PCRiskPro_Setup.exe" /VERYSILENT /NORESTART`
4. Add step: **Delete File** → Path: `#vAgentConfiguration.AgentTempDir#\PCRiskPro_Setup.exe`
5. Save and assign to target agents via Machine Groups

### 8.5 Generic PowerShell (Any RMM)

If your RMM supports running arbitrary PowerShell, this script works universally:

```powershell
# Generic PCRiskPro Silent Install — works on any RMM with PowerShell support

param(
    [string]$LicenseKey = "",
    [string]$InstallerUrl = "https://github.com/PCRiskPro/Releases/releases/download/v1.9.9-beta10/PCRiskPro_Setup_1.9.9.exe"
)

$ErrorActionPreference = "Stop"
$installerPath = "$env:TEMP\PCRiskPro_Setup_$(Get-Date -Format 'yyyyMMdd').exe"
$logPath = "C:\ProgramData\PCRiskPro\install_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

# Ensure log directory exists
New-Item -ItemType Directory -Force -Path (Split-Path $logPath) | Out-Null

# Download installer
Write-Host "Downloading installer from $InstallerUrl..."
Invoke-WebRequest -Uri $InstallerUrl -OutFile $installerPath -UseBasicParsing

# Verify Authenticode signature
$sig = Get-AuthenticodeSignature $installerPath
if ($sig.Status -ne "Valid" -or -not ($sig.SignerCertificate.Subject -like "*PCRiskpro Limited*")) {
    Remove-Item $installerPath -Force
    throw "Installer signature verification failed — aborting install"
}

# Install silently
Write-Host "Installing PCRiskPro silently..."
$proc = Start-Process -FilePath $installerPath `
                      -ArgumentList "/VERYSILENT /NORESTART /LOG=`"$logPath`"" `
                      -Wait -PassThru

# Cleanup
Remove-Item $installerPath -Force

if ($proc.ExitCode -eq 0) {
    Write-Host "PCRiskPro installed successfully"
    Exit 0
} else {
    Write-Error "Install failed with exit code $($proc.ExitCode). See log: $logPath"
    Exit $proc.ExitCode
}
```

---

## 9. Scheduled Scanning (v2.0.0)

Once `pcriskpro-cli.exe` ships in v2.0.0, you can schedule regular scans using Windows Task Scheduler or your RMM's scheduled task feature.

**Example — weekly scan via scheduled task (v2.0.0):**

```powershell
# Create a weekly scan scheduled task
$action = New-ScheduledTaskAction -Execute "C:\Program Files\PCRiskPro\pcriskpro-cli.exe" `
                                   -Argument "--scan all --output \\fileserver\pcriskpro\$env:COMPUTERNAME.json --quiet"

$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 2AM

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName "PCRiskPro Weekly Scan" `
                       -Action $action -Trigger $trigger -Principal $principal
```

The MSP then runs the Report Aggregator weekly/monthly to generate portfolio reports from the collected JSON files.

---

## 10. Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| Installer returns exit code 1 | Insufficient disk space or permissions | Ensure target machine has ≥2 GB free disk and script runs with elevation |
| SmartScreen blocks installer | First-run warning for new-ish certificate | Select "More info" → "Run anyway", or pre-approve via AppLocker/MOTW removal |
| Silent install hangs | Antivirus scanning the installer | Temporarily whitelist `PCRiskPro_Setup.exe` in your AV; add `C:\Program Files\PCRiskPro\` to exclusions |
| Installer downloaded but signature check fails | Corrupted download or MITM | Retry download over a different network; verify SHA-256 against GitHub release page |
| Install logs mention "access denied" | Running as non-admin user | Run RMM script with elevated privileges |
| Scans return empty results | Required admin rights for some checks | Ensure scheduled task runs as SYSTEM or with elevation |
| Licence activation fails | Activation limit reached (v2.0.0) | Log in to Lemon Squeezy customer portal, deactivate unused slots |

---

## 11. Frequently Asked Questions

**Can I deploy PCRiskPro to 500 endpoints across 20 clients?**
Yes — purchase the MSP/Partner tier (£1,995/year) which covers up to 100 activated endpoints. For larger deployments, contact sales@pcriskpro.com about MSP Plus pricing.

**Do endpoints need to be internet-connected?**
Only for first-time licence activation (when v2.0.0 ships), CVE database updates (cached 24 hours), and periodic licence re-validation. Scanning itself works offline.

**What data leaves the endpoint?**
For licence activation: machine identifier and product version. For CVE lookups: installed application names and versions (queried against NVD/OSV). **No scan results, file content, or sensitive data leave the endpoint.**

**Can I customise the installer or bundle it with other software?**
No. The installer is Authenticode-signed and must not be modified. MSP/Partner tier licences include white-label branding of **reports** (logo, colours), which is distinct from modifying the installer itself.

**Does PCRiskPro interfere with other security software?**
PCRiskPro is a passive scanner — it does not monitor processes, intercept files, or install drivers. It does not conflict with EDR, AV, or DLP products. A one-off scan of a typical endpoint takes 5–15 minutes and uses modest CPU.

**What happens when I retire a client machine?**
Uninstall PCRiskPro via Windows Apps & Features (or silent uninstall via `C:\Program Files\PCRiskPro\unins000.exe /VERYSILENT`). Then in the Lemon Squeezy customer portal (v2.0.0+), deactivate that endpoint's slot to free it for a new machine.

**Can I automate scan-and-email workflows?**
In v2.0.0, `pcriskpro-cli.exe` exits with distinct codes for clean, critical-finding, and error states. Your RMM or scheduled task can act on the exit code to trigger email alerts, tickets, or escalations.

**Is there an API for integration?**
Not in v2.0.0. The CLI + JSON output is the supported integration pattern. API surface may be added in a future release based on customer demand — contact us if this is a blocker.

---

## 12. Support

| Purpose | Contact |
|---------|---------|
| Sales and partner enquiries | sales@pcriskpro.com |
| Technical support (MSP tier) | support@pcriskpro.com — response target 8 business hours |
| Security disclosure | security@pcriskpro.com |
| Product website | [pcriskpro.com](https://pcriskpro.com) |
| Release notifications | Subscribe to the [Releases page](https://github.com/PCRiskPro/Releases/releases) |

---

© 2024–2026 PCRiskpro Limited. All rights reserved. This guide is published under the terms of the [End User Licence Agreement](../legal/EULA.md). Content subject to change as v2.0.0 development progresses.
