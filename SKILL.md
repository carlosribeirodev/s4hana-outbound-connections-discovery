---
name: s4hana-outbound-connetions-discovery
description: Discover and report outbound HTTP/HTTPS connections in SAP S/4HANA by reading SOAMANAGER consumer logical ports and SM59 HTTP destinations with read-only ABAP SQL MCP tools. Use for complete outbound endpoint inventories, arbitrary domain or host searches, integration audits, active/inactive logical-port analysis, SM59 type G/H checks, and DDIC-based discovery of the release-specific persistence tables.
---

# S/4HANA Outbound Connections Discovery

Inventory outbound SOAP logical ports and SM59 HTTP destinations without changing SAP configuration. Support complete inventories and user-supplied filters such as any domain, hostname, path, destination name, proxy name, protocol, or integration keyword. Treat Ariba as one possible example, never as a built-in restriction.

## Safety

- Use read-only system, DDIC, and ABAP SQL tools only.
- Never create, update, activate, or delete SAP objects or configuration.
- Never expose passwords, tokens, authentication payloads, or secure-store values.
- Report the connected system ID and client context.
- Treat internal tables as release-dependent and confirm them through DDIC first.
- Prefer official SAP Help for product semantics and the live DDIC for persistence details.

## 1. Select the SAP system and scope

List connected SAP systems. Use the requested system or the sole connected system.

Derive filters from the request:

- No filter: inventory every outbound HTTP/S connection.
- Domain/host: match normalized endpoint hosts.
- Name/keyword: search logical-port, proxy, destination, description, host, and path fields; label these semantic matches separately from host matches.
- Protocol/state: filter only after collecting enough data to explain excluded rows.

Use the highest safe `maxRows` needed to avoid truncation.

## 2. Confirm tables through DDIC

Query `DD02L`, `DD02T`, and `DD03L`. Confirm these candidates and their fields before use:

| Table | Expected purpose |
|---|---|
| `SRT_CFG_DIR` | SOAP configuration directory |
| `SRT_CFG_DIR_T` | Configuration descriptions |
| `SRT_RTC_BNDG` | Configuration-key to binding-key mapping |
| `SRT_RTC_DATA` | Runtime properties including endpoint components |
| `SRT_RTC_DATAL` | Long runtime values, if populated |
| `SRT_LP`, `SRT_LP_URL_ASGN` | Older logical-port persistence fallback |
| `RFCDES` | SM59 RFC/HTTP destinations |

Example:

```sql
SELECT fieldname, position, keyflag, rollname, datatype, leng
  FROM dd03l
  WHERE tabname = 'SRT_RTC_DATA'
    AND as4local = 'A'
  ORDER BY position
```

Query table counts. Use the runtime-cache path when `SRT_CFG_DIR` and `SRT_RTC_DATA` are populated. Use `SRT_LP` and `SRT_LP_URL_ASGN` only when they contain the effective configuration.

## 3. Read SOAMANAGER consumer logical ports

For the runtime-cache path:

- Interpret `SRT_CFG_DIR-TYPE = 'CR'` as consumer runtime configuration.
- Treat `STATE = 'A'` as active and report `STATE = 'I'` separately as inactive.
- Use `CONFIG_NAME_EXT` as the logical-port/configuration name.
- Use `DT_OBJ_NAME` as the consumer proxy name.
- Map `SRT_CFG_DIR-CONFIG_KEY` to `SRT_RTC_BNDG-CONFIG_KEY`.
- Map `SRT_RTC_BNDG-BINDING_KEY` to `SRT_RTC_DATA-BINDING_KEY`.

Read the mapping:

```sql
SELECT d~config_name_ext,
       d~dt_obj_name,
       d~config_key,
       d~state,
       d~changed_by,
       d~changed_on,
       b~binding_key
  FROM srt_cfg_dir AS d
  INNER JOIN srt_rtc_bndg AS b
    ON b~config_key = d~config_key
  WHERE d~type = 'CR'
```

Read endpoint components:

```sql
SELECT binding_key, state, record_type, subj_type, prop_name, prop_value
  FROM srt_rtc_data
  WHERE record_type = 'PR'
    AND subj_type = 'BN'
    AND prop_name LIKE 'URL%'
```

Correlate locally by `BINDING_KEY` if the MCP parser rejects a three-table join. Read descriptions from `SRT_CFG_DIR_T` by configuration key and state.

Pivot these properties per binding:

- `URLProtocol`
- `URLHost`
- `URLPort`
- `URLPath`

Keep `http` and `https` for an HTTP inventory. Construct:

```text
protocol://host[:non-default-port]/path
```

Preserve the configured port in structured results. Count logical ports, not property rows. Do not deduplicate ports merely because they share an endpoint.

## 4. Apply general endpoint filters

Normalize hosts to lowercase and remove a trailing dot.

For a domain `example.com`, use the DNS-safe test:

```text
host == "example.com" OR host ends with ".example.com"
```

If the user explicitly requests substring containment, also provide substring results, but distinguish them from DNS-suffix matches.

Do not confuse metadata URIs with network endpoints. Values in `SubjectNamespace`, `externalRequestNamespace`, `externalResponseNamespace`, WSDL namespaces, or schema URIs may contain a requested domain without identifying the target host. Determine endpoint-domain matches from `URLHost` only.

Support any domain or endpoint family. Examples include SAP Managed Gateway, SAP Cloud Integration, SuccessFactors, Salesforce, banks, tax services, internal hosts, or custom partner APIs. For product-related discovery, separately classify name/path matches and exact host matches.

## 5. Read SM59 HTTP destinations

Confirm `RFCDES` fields via `DD03L`. HTTP types are:

- `G`: HTTP connection to an external server.
- `H`: HTTP connection to an ABAP system.

Settings are serialized across:

```text
RFCOPTIONS, RFCOPTIONT, RFCOPTIONU, RFCOPTIONV,
RFCOPTION1 ... RFCOPTION9, RFCOPTIONA, RFCOPTIONB
```

Search every segment case-insensitively. Run separate queries if a wide `OR` expression exceeds MCP parser limits:

```sql
SELECT rfcdest, rfctype, rfcoptions
  FROM rfcdes
  WHERE ( rfctype = 'G' OR rfctype = 'H' )
    AND lower( rfcoptions ) LIKE '%example.com%'
```

When inventorying all type G/H destinations, concatenate segments in the order above and parse only non-secret keys:

| Key | Meaning |
|---|---|
| `H` | Target host |
| `I` | Service number/port |
| `N` | Path prefix |
| `s` | SSL activation |
| `G` | Proxy host |
| `g` | Proxy port |

Apply domain matching to parsed target host `H`. Do not output serialized authentication fields.

## 6. Validate completeness

```sql
SELECT type, state, COUNT( * ) AS row_count
  FROM srt_cfg_dir
  GROUP BY type, state
```

- Compare active `CR` counts with reconstructed HTTP/S port counts.
- Explain gaps such as PI/local ports, missing URL properties, missing bindings, or inactive rows.
- Check for `maxRows` truncation.
- Search every relevant `RFCDES` continuation before reporting zero matches.
- Keep active, inactive, exact-host, substring, and semantic/name matches separate.

## 7. Check official SAP documentation

Use official SAP documentation to validate that SOAMANAGER configures consumer logical ports and that a logical port contains the access URL for one service endpoint. Validate that SM59 type G is HTTP to an external server and type H is HTTP to an ABAP system.

Primary references:

- `https://help.sap.com/docs/SAP_NETWEAVER_701/6f43fe6c6c4b1014a88dfde574176a62/9ec7a3591dc74a679bbc9716354e42af.html`
- `https://help.sap.com/docs/ABAP_PLATFORM_NEW/684cffda9cbc4187ad7dad790b03b983/479bcd5b65a9484be10000000a421138.html`
- `https://help.sap.com/docs/SAP_NETWEAVER_740/864321b9b3dd487d94c70f6a007b0397/48d7ad896b57154ee10000000a421937.html`

Verify current URLs when browsing is available. State that internal table identification came from the target system's DDIC when public SAP Help does not document persistence tables.

## 8. Report results

Lead with counts for active HTTP, active HTTPS, inactive ports, exact SOAMANAGER host matches, and SM59 type G/H matches.

Then provide:

1. Exact host/domain matches.
2. Complete logical-port inventory grouped by endpoint when requested.
3. Inactive configurations separately.
4. SM59 destination, type, host, port, path, and SSL status.
5. Optional semantic/name matches clearly labeled as such.
6. Confirmed tables, relationships, filters, and documentation.
7. Limitations or unexplained count differences.

Never infer that an integration does not exist solely because a particular domain is absent; it may use middleware or a newer provider domain.
