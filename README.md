# S/4HANA Outbound Connections Discovery

## Purpose

This personal Codex skill performs a read-only inventory of outbound HTTP/HTTPS connections in SAP S/4HANA. It discovers SOAMANAGER consumer logical ports, active and inactive endpoint URLs, and SM59 HTTP destinations of type G or H.

The skill is not specific to Ariba. It can process every endpoint or filter by any domain, host, path, protocol, destination name, consumer proxy, or integration keyword.

## Prerequisites

- An SAP/ABAP MCP connection to the target S/4HANA system.
- A read-only ABAP SQL tool such as `sap_sql_query`.
- SAP authorization to read DDIC metadata and SOAP/RFC configuration tables.
- Optional internet access for official SAP Help verification.

## Execute the skill

Complete inventory:

```text
Use $s4hana-outbound-connetions-discovery to inventory all outbound HTTP and HTTPS logical ports in SOAMANAGER and all SM59 HTTP destinations.
```

Filter by any domain:

```text
Use $s4hana-outbound-connetions-discovery to find SOAMANAGER and SM59 endpoints whose target host belongs to example.com.
```

Filter by integration or keyword:

```text
Use $s4hana-outbound-connetions-discovery to list outbound connections related to SuccessFactors. Separate exact host matches from names or paths that merely contain the keyword.
```

Target a connected system:

```text
Use $s4hana-outbound-connetions-discovery on system <SYSTEM_ID>. Include active and inactive logical ports, reconstructed URLs, and SM59 type G/H destinations.
```

## Expected output

The skill reports the connected system, counts by protocol/state, reconstructed SOAMANAGER endpoints, SM59 destination details, requested matches, the confirmed release-specific table path, official SAP documentation, and any completeness limitations.

## Safety

The workflow is read-only. It must not change SOAMANAGER, SM59, ABAP objects, credentials, or secure-store data. Authentication details are excluded from reports.
