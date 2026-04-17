# PCRiskPro

**Enterprise security risk assessment for UK SMEs, IT consultants, and Managed Service Providers.**

Official installers, datasheets, and product materials are published here. For product information, pricing, and purchases, visit **[pcriskpro.com](https://pcriskpro.com)**.

---

## About PCRiskPro

PCRiskPro is a Windows desktop application that delivers a comprehensive security posture assessment in under 15 minutes, covering four critical domains:

- **Data Discovery** — Automated scanning for sensitive data (PII, credentials, financial records) with OCR and pattern recognition across documents, images, and archives
- **Application Security** — CVE detection across installed software using NVD, OSV, and CISA KEV data sources
- **Cyber Essentials Readiness** — Assessment against the UK Government's five Cyber Essentials control themes
- **System Security** — Advanced Windows hardening checks (TPM, Secure Boot, Credential Guard, BitLocker, and more)

Built for organisations that need credible, evidence-based cyber risk visibility without the complexity and cost of enterprise vulnerability management platforms.

---

## Download

| Resource | Link |
|----------|------|
| **Latest installer** | [Download from Releases](../../releases/latest) |
| All releases | [View release history](../../releases) |

**File**: `PCRiskPro_Setup_<version>.exe` (~170 MB)
**Signed by**: PCRiskpro Limited (Certum OV code signing certificate)

---

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| OS | Windows 10 (64-bit) | Windows 11 (64-bit) |
| CPU | Dual-core 2.0 GHz | Quad-core 2.5 GHz+ |
| RAM | 4 GB | 8 GB |
| Disk space | 1 GB free | 2 GB free |
| Internet | Required for first activation and CVE lookups | — |
| Privileges | Standard user (Admin recommended for full system checks) | — |

**Supported Windows editions:** Home, Pro, Enterprise, Server 2019, Server 2022, Server 2025.

---

## Installation

### Standard install (end users)

1. Download the latest installer from the [Releases page](../../releases/latest)
2. Double-click `PCRiskPro_Setup_<version>.exe`
3. Follow the setup wizard
4. Launch PCRiskPro from the Start menu
5. Enter your licence key when prompted (or continue in trial mode)

### Silent install (IT administrators and MSPs)

PCRiskPro supports unattended installation via command line for RMM deployment and enterprise rollout:

```cmd
PCRiskPro_Setup_<version>.exe /SILENT /NORESTART /LOG=install.log
```

**Common flags:**

| Flag | Purpose |
|------|---------|
| `/SILENT` | Suppress UI, show progress only |
| `/VERYSILENT` | Fully unattended, no UI |
| `/NORESTART` | Do not restart automatically after install |
| `/LOG=<path>` | Write install log to specified path |
| `/DIR=<path>` | Override default install location |
| `/TASKS=desktopicon` | Create desktop shortcut |

**Example — Push via RMM with PowerShell:**

```powershell
$url = "https://github.com/PCRiskPro/Releases/releases/latest/download/PCRiskPro_Setup_1.9.9.exe"
$dest = "$env:TEMP\PCRiskPro_Setup.exe"
Invoke-WebRequest -Uri $url -OutFile $dest
Start-Process -FilePath $dest -ArgumentList "/VERYSILENT /NORESTART" -Wait
```

Full RMM deployment guides for ConnectWise Automate, Datto RMM, NinjaOne, and Kaseya will be published as individual guides closer to commercial launch.

---

## Verifying the Installer

All official releases are digitally signed with a **Certum OV code signing certificate** issued to **PCRiskpro Limited**. To verify:

### Method 1 — Windows File Properties

1. Right-click the installer → **Properties** → **Digital Signatures** tab
2. Confirm signer: **PCRiskpro Limited**
3. Click the signature → **Details** → **View Certificate**
4. Confirm issuer: **Certum Code Signing 2021 CA**

### Method 2 — PowerShell

```powershell
Get-AuthenticodeSignature .\PCRiskPro_Setup_1.9.9.exe | Format-List *
```

Expected output: `Status: Valid`, signed by `CN=PCRiskpro Limited`.

**Never run installers that fail signature verification.** If Windows SmartScreen flags an installer, download a fresh copy from this repository and contact support if the issue persists.

---

## Licensing

PCRiskPro is commercial software available under three tiers:

| Tier | Price | Target customer | Devices per licence |
|------|-------|-----------------|---------------------|
| **Solo** | £95/year | Small businesses (self-assessment) | 1 |
| **SME** | £795/year | Internal IT teams (up to 10 machines) | 10 |
| **MSP / Partner** | £1,995/year | MSPs and IT consultancies (up to 100 client endpoints, white-label reporting, commercial use permitted) | 100 |

A **perpetual free trial** is available with feature limitations (scan visibility capped, export restricted). Purchase a licence to unlock full functionality.

**Purchase and manage licences at [pcriskpro.com](https://pcriskpro.com).**

---

## Resources

### Product materials

- [Product datasheet (PDF)](./datasheets/PCRiskPro_Datasheet_2026.pdf)
- [Brand guidelines (PDF)](./datasheets/PCRiskPro_Brand_Guidelines.pdf)

### Legal documents

- [End User Licence Agreement (EULA)](./legal/EULA.md)
- [Privacy Policy](./legal/PRIVACY_POLICY.md)
- [Third-party licences](./legal/THIRD-PARTY-LICENSES.txt)

### External links

- [Product website](https://pcriskpro.com)
- [Features overview](https://pcriskpro.com/features)
- [Pricing](https://pcriskpro.com/pricing)

---

## Support

| Channel | Contact |
|---------|---------|
| Product website | [pcriskpro.com](https://pcriskpro.com) |
| Sales enquiries | sales@pcriskpro.com |
| Technical support | support@pcriskpro.com |
| Security disclosure | security@pcriskpro.com |

Response targets: Solo tier — 48 hours (business days). SME tier — 24 hours. MSP tier — 8 hours.

---

## Release History

All published releases are listed on the [Releases page](../../releases) with changelog entries covering new features, security updates, bug fixes, and performance improvements.

Subscribe to repository releases (**Watch** → **Custom** → **Releases**) to receive notifications of new versions.

---

## Frequently Asked Questions

**Is this repository open source?**
No. PCRiskPro is commercial software. This repository hosts the signed installer and public-facing documentation only. Source code is not published.

**Why is the installer hosted on GitHub?**
GitHub provides a reliable, globally distributed CDN for software distribution, trusted by IT administrators and verifiable by signature. This removes the need for self-hosted download infrastructure and ensures consistent availability.

**Can I distribute PCRiskPro to my clients?**
Only the MSP / Partner tier permits commercial distribution and resale. Solo and SME tiers are restricted to internal use by the licensee. See the [EULA](./legal/EULA.md) for full terms.

**Does PCRiskPro work offline?**
Most functionality works offline after first activation. An internet connection is required for: initial licence activation, CVE database updates (cached 24 hours), and periodic licence re-validation.

**Which Windows versions are supported?**
Windows 10 and 11 (Home, Pro, Enterprise), Windows Server 2019, 2022, and 2025. Windows 7, 8, and 8.1 are not supported.

**Where is my data stored?**
All scan data, reports, and findings are stored locally on the scanned machine. PCRiskPro does not upload scan results to any remote server. Only licence activation metadata (machine identifier, product version) is transmitted for licence enforcement.

**How do I report a bug or security issue?**
General bugs: support@pcriskpro.com. Security vulnerabilities: security@pcriskpro.com (please do not open public issues for security disclosures).

---

© 2024–2026 PCRiskpro Limited. All rights reserved.
PCRiskPro® is a registered trademark of PCRiskpro Limited.
