# OFAC Sanctions Check Skill

## What It Is
The OFAC SDN (Specially Designated Nationals) list is the US Treasury's list of sanctioned individuals, entities, and cryptocurrency addresses. Interacting with sanctioned addresses is illegal in many jurisdictions.

## How to Check

### Method 1: OFAC API (free, no key)
```bash
# Search by crypto address
curl "https://api.ofac.io/v1/search?query={ADDRESS}&apiKey=free"
```

### Method 2: Download & grep (more reliable)
```bash
# Download the full SDN list (digital currency addresses)
curl "https://www.treasury.gov/ofac/downloads/sanctions/1.0/sdn_advanced.xml" -o /tmp/ofac.xml
grep -i "{ADDRESS}" /tmp/ofac.xml
```

### Method 3: Chainalysis free oracle (Ethereum)
The Chainalysis sanctions oracle is a free on-chain contract:
- Contract: `0x40C57923924B5c5c5455c48D93317139ADDaC8fb` (ETH mainnet)
- Call `isSanctioned(address)` → returns bool

## EU Sanctions
```bash
# EU consolidated sanctions list
curl "https://webgate.ec.europa.eu/fsd/fsf/public/files/xmlFullSanctionsList_1_1/content" -o /tmp/eu_sanctions.xml
grep -i "{ADDRESS}" /tmp/eu_sanctions.xml
```

## Reporting Format
```
SANCTIONS CHECK RESULT
======================
Address: 0x...
OFAC SDN: [MATCH / NO MATCH]
EU Sanctions: [MATCH / NO MATCH]
Chainalysis Oracle: [SANCTIONED / CLEAR]

Note: Sanctions lists update daily. This check reflects data as of [DATE].
Always verify with official sources before taking legal action.
```

## Important Disclaimer
Always include in reports:
> "This is informational only. Sanctions compliance is a legal matter. Consult qualified legal counsel before making compliance decisions."
