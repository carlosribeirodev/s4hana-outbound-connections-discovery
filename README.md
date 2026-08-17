# S/4HANA Outbound Connections Discovery

## Purpose

This personal Codex skill performs a read-only inventory of outbound HTTP/HTTPS connections in SAP S/4HANA. It discovers SOAMANAGER consumer logical ports, active and inactive endpoint URLs, and SM59 HTTP destinations of type G or H.

The skill is not specific to Ariba. It can process every endpoint or filter by any domain, host, path, protocol, destination name, consumer proxy, or integration keyword.

## Prerequisites

- An SAP/ABAP MCP connection to the target S/4HANA system.
- A read-only ABAP SQL tool such as `sap_sql_query`.
- SAP authorization to read DDIC metadata and SOAP/RFC configuration tables.
- Optional internet access for official SAP Help verification.

## Install the skill

### Ask an AI coding agent to install it

If your AI coding agent supports installing Codex skills from GitHub, send this prompt:

```text
Install this Codex skill: https://github.com/carlosribeirodev/s4hana-outbound-connections-discovery.git
```

The repository contains `SKILL.md` at its root. A compatible agent should install it into the personal Codex skills directory as `s4hana-outbound-connections-discovery`. The skill should become available on the next turn.

If the agent needs more explicit instructions, use:

```text
Install the Codex skill from https://github.com/carlosribeirodev/s4hana-outbound-connections-discovery.git. The SKILL.md file is at the repository root. Install it as s4hana-outbound-connections-discovery in my personal Codex skills directory. Do not execute the skill yet; tell me when it is available.
```

### Install manually with Git

PowerShell:

```powershell
$codexRoot = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $env:USERPROFILE '.codex' }
git clone https://github.com/carlosribeirodev/s4hana-outbound-connections-discovery.git (Join-Path $codexRoot 'skills\s4hana-outbound-connections-discovery')
```

macOS or Linux:

```bash
git clone https://github.com/carlosribeirodev/s4hana-outbound-connections-discovery.git "${CODEX_HOME:-$HOME/.codex}/skills/s4hana-outbound-connections-discovery"
```

If that destination already exists, remove or rename the existing installation only after preserving any local changes. After installation, start a new Codex turn. If the skill is not detected, restart the Codex client.

## Execute the skill

Complete inventory:

```text
Use $s4hana-outbound-connections-discovery to inventory all outbound HTTP and HTTPS logical ports in SOAMANAGER and all SM59 HTTP destinations.
```

Filter by any domain:

```text
Use $s4hana-outbound-connections-discovery to find SOAMANAGER and SM59 endpoints whose target host belongs to example.com.
```

Filter by integration or keyword:

```text
Use $s4hana-outbound-connections-discovery to list outbound connections related to SuccessFactors. Separate exact host matches from names or paths that merely contain the keyword.
```

Target a connected system:

```text
Use $s4hana-outbound-connections-discovery on system <SYSTEM_ID>. Include active and inactive logical ports, reconstructed URLs, and SM59 type G/H destinations.
```

## Expected output

The skill reports the connected system, counts by protocol/state, reconstructed SOAMANAGER endpoints, SM59 destination details, requested matches, the confirmed release-specific table path, official SAP documentation, and any completeness limitations.

## Safety

The workflow is read-only. It must not change SOAMANAGER, SM59, ABAP objects, credentials, or secure-store data. Authentication details are excluded from reports.
