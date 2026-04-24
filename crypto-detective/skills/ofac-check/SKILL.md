# Sanctions Check Skill

## What This Is
Screen a crypto address against the major economic-sanctions lists used globally: OFAC (US), EU, UK HMT/OFSI, UN Security Council, Canada SEMA. Interacting with sanctioned addresses can be illegal in the jurisdictions that maintain each list. This skill produces a SIGNAL, not a compliance verdict — always hand final determinations to qualified counsel.

## Cardinal rule: exact match, not substring

Never grep a sanctions XML for a substring of an address. A partial match can link an innocent wallet to a sanctioned one because checksum-case or truncated-prefix collisions happen. Always:

1. Parse the XML or JSON into a structured list.
2. Compare **case-insensitively for EVM** (normalise both sides to lowercase-hex), **case-sensitively for Tron / Solana / BTC**.
3. Verify the address encoding is valid on its own chain BEFORE comparison (see blockchain-forensics skill).

## Checks in order (cheap → expensive, first hit stops)

### 1 · Chainalysis Sanctions API (free, no key, fastest)
```bash
curl -fsS --max-time 10 \
  "https://public.chainalysis.com/api/v1/address/${ADDR}"
# Returns {"identifications":[{"category":"sanctions","name":"...","description":"...","url":"..."}]}
# Empty "identifications" array = clean. Non-empty = flagged.
```
This is Chainalysis's curated public feed of sanctioned addresses. It lags the underlying OFAC list by minutes-to-hours but is the single fastest check.

### 2 · Chainalysis Sanctions Oracle (on-chain, free, EVM only)
Deployed at one of two addresses across most major EVM chains — call `isSanctioned(address)` via `eth_call`:
- `0x40C57923924B5c5c5455c48D93317139ADDaC8fb` — Ethereum, Polygon, BSC, Avalanche, Arbitrum, Optimism, Celo, Fantom, Blast
- `0x3A91A31cB3dC49b4db9Ce721F50a9D076c8D739B` — Base

```bash
# Using cast (foundry) — single key covers all EVM chains via Etherscan V2 RPC lookup
cast call 0x40C57923924B5c5c5455c48D93317139ADDaC8fb \
  "isSanctioned(address)(bool)" "${ADDR}" \
  --rpc-url "${ETH_RPC_URL}"
```

### 3 · OFAC SDN (authoritative US list)
Two valid download options:
```bash
# Full SDN (large, ~80 MB XML)
curl -fsS --max-time 60 \
  "https://www.treasury.gov/ofac/downloads/sanctions/1.0/sdn_advanced.xml" \
  -o /tmp/ofac_sdn.xml

# Community-maintained digital-currency extract (much smaller, easier to diff)
# Maintained at github.com/0xB10C/ofac-sanctioned-digital-currency-addresses
curl -fsS --max-time 20 \
  "https://raw.githubusercontent.com/0xB10C/ofac-sanctioned-digital-currency-addresses/lists/sanctioned_addresses_ETH.json" \
  -o /tmp/ofac_eth.json
# Variants: _BTC, _TRX, _XMR, _USDT_ETH, _USDC_ETH, _XRP, _SOL, _LTC, _BCH, _ETC, _ARB, _DASH, _BSC
```

Exact match via jq (not grep):
```bash
# EVM address — case-insensitive
jq -r --arg a "$(echo "$ADDR" | tr '[:upper:]' '[:lower:]')" \
  '.[] | ascii_downcase | select(. == $a)' /tmp/ofac_eth.json
```

For parsing `sdn_advanced.xml`, extract addresses via XPath targeting `<Feature>` elements whose `FeatureType` is a Digital Currency variant. Do NOT grep — the XML contains addresses inside other people's ID fields, biographical text, etc., which produce false positives.

### 4 · EU consolidated financial sanctions
```bash
curl -fsS --max-time 60 \
  "https://webgate.ec.europa.eu/fsd/fsf/public/files/xmlFullSanctionsList_1_1/content" \
  -o /tmp/eu_sanctions.xml
```
Crypto-specific designations are thinner than OFAC's but growing.

### 5 · UK HMT / OFSI consolidated list
```bash
curl -fsS --max-time 60 \
  "https://ofsistorage.blob.core.windows.net/publishlive/2022format/ConList.xml" \
  -o /tmp/uk_ofsi.xml
```
UK maintained Tornado-Cash designations even after the US 2025-03-21 delisting — always check separately.

### 6 · UN Security Council consolidated list
```bash
curl -fsS --max-time 60 \
  "https://scsanctions.un.org/resources/xml/en/consolidated.xml" \
  -o /tmp/un_sanctions.xml
```
Mostly entity/individual designations; crypto address designations are rare but exist.

### 7 · Canada SEMA list
Consolidated list available at `international.gc.ca`. JSON/XML structure varies by update — fetch the page and parse the attached data file rather than relying on a stable API URL.

## Reporting Format

Every report lists each source independently and states the exact timestamp of the data fetched. Never collapse multiple sources into a single "sanctioned? yes/no".

```
SANCTIONS CHECK RESULT
======================
Address: 0x… (chain: Ethereum, EIP-55: 0x…)
Checked at: 2026-04-24T10:04:17Z

Source                          Result     List-updated
------------------------------  ---------  ------------------
Chainalysis Sanctions API       NO MATCH   live query
Chainalysis Oracle (ETH)        CLEAR      on-chain, block 19823451
OFAC SDN (sdn_advanced.xml)     NO MATCH   2026-04-24 (fetched today)
OFAC digital-currency extract   NO MATCH   2026-04-24
EU consolidated                 NO MATCH   2026-04-23
UK HMT / OFSI                   NO MATCH   2026-04-23
UN Security Council             NO MATCH   2026-04-22
Canada SEMA                     NO MATCH   2026-04-22

Tornado Cash proximity (any)    NONE within 3 hops
  (Reminder: US OFAC sanctions on Tornado Cash contracts were lifted 2025-03-21
   by the Treasury following Van Loon v. Treasury (5th Cir. 2024-11-26). UK HMT
   and some non-US jurisdictions may still maintain designations on specific
   TC-linked addresses — always cross-check those lists.)

Assessment: address is not present on any checked list as of the fetch times above.
Confidence: HIGH for direct-match absence. MEDIUM-LOW for "no association" — nothing
in this output precludes behavioural, cluster-level, or indirect exposure. Always
combine with flow analysis (see blockchain-forensics).
```

## Important caveats

- **This is informational only.** Sanctions compliance is a legal matter. Consult qualified counsel before making any compliance decision (freeze, report, off-board, file SAR).
- **Lists change daily.** Every report must include the fetch timestamp of each source.
- **Absence is not innocence.** Direct-match "clear" results do not rule out proximity to sanctioned clusters, unsanctioned-but-malicious activity, or jurisdictions not covered here.
- **Tornado Cash (US status, 2025-03-21):** OFAC lifted its designation on the Tornado Cash contracts. The Chainalysis oracle may still surface legacy associations as risk signals, and non-US regimes may still sanction specific TC-linked addresses — do not read a clean OFAC result as a global green light.
- **Indirect exposure is a separate analysis.** A wallet that received funds from a sanctioned address (or sent to one) is not itself sanctioned, but many compliance frameworks treat N-hop proximity as elevated risk. Report exposure hops numerically — never conflate.
