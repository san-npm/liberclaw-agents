# Blockchain Forensics Skill

## Before You Query — Validate

Every address must be validated against its chain's encoding before it leaves the process. Bad addresses waste rate budget, leak partial PII into third-party logs, and can trigger misleading "not found" conclusions.

- **EVM** (ETH, Polygon, BSC, Arbitrum, Optimism, Base, Avalanche, Fantom, Linea, Scroll, zkSync, Blast): `^0x[a-fA-F0-9]{40}$`, and if mixed-case, verify EIP-55 checksum. Normalise to EIP-55 before displaying.
- **Tron**: Base58Check, starts with `T`, 34 chars. Reject anything else.
- **Solana**: Base58, 32–44 chars, no `0`, `O`, `I`, or `l`.
- **Bitcoin**: legacy `1...` / `3...` (Base58Check), native SegWit `bc1q...` / Taproot `bc1p...` (Bech32m). Validate the checksum — do not match substring.
- **Cardano**: Bech32 with `addr1...` prefix.
- **XRPL**: Base58, starts with `r`, 25–35 chars.
- **TON**: 48 chars, EQ/UQ prefix, or raw `0:<hex64>` form. Normalise.

If validation fails, stop and ask the user to paste the full address — do not "best-effort" a partial.

### Unicode / homoglyph guard
Any address, ENS name, or token symbol coming back from an API or the user must be NFKC-normalised. Flag and display-escape:
- Cyrillic/Greek homoglyphs (`а`, `е`, `о`, `р`, `с` — look Latin but aren't)
- Zero-width chars (`U+200B`, `U+200C`, `U+200D`, `U+FEFF`)
- RTL override (`U+202E`)
- Confusable digits (`U+FF10`..`U+FF19` fullwidth)

Never render unsanitised on-chain strings verbatim in a report.

## API Quick Reference

### Ethereum / all EVM chains (Etherscan V2 unified)
```bash
# Wallet tx history — one key, any chain. chainid=1 Ethereum, 137 Polygon, 56 BSC,
# 42161 Arbitrum, 10 Optimism, 8453 Base, 43114 Avalanche.
curl -fsS --max-time 20 \
  "https://api.etherscan.io/v2/api?chainid=1&module=account&action=txlist&address=${ADDR}&startblock=0&endblock=99999999&sort=asc&apikey=${ETHERSCAN_API_KEY}"

# Token transfers (ERC-20)
curl -fsS --max-time 20 \
  "https://api.etherscan.io/v2/api?chainid=1&module=account&action=tokentx&address=${ADDR}&sort=asc&apikey=${ETHERSCAN_API_KEY}"

# Internal transactions
curl -fsS --max-time 20 \
  "https://api.etherscan.io/v2/api?chainid=1&module=account&action=txlistinternal&address=${ADDR}&apikey=${ETHERSCAN_API_KEY}"
```

### Bitcoin (prefer keyless, privacy-respecting)
```bash
# Confirmed + mempool
curl -fsS --max-time 20 "https://mempool.space/api/address/${ADDR}"
curl -fsS --max-time 20 "https://blockstream.info/api/address/${ADDR}/txs"
# Last-resort fallback
curl -fsS --max-time 20 "https://blockchain.info/rawaddr/${ADDR}?limit=50"
```

### Solana
```bash
# Solscan Pro (public-api.solscan.io is retired)
curl -fsS --max-time 20 \
  -H "token: ${SOLSCAN_API_KEY}" \
  "https://pro-api.solscan.io/v2.0/account/transactions?address=${ADDR}&limit=50"

# Helius (alternative)
curl -fsS --max-time 20 \
  "https://api.helius.xyz/v0/addresses/${ADDR}/transactions?api-key=${HELIUS_API_KEY}"
```

### Tron (critical for USDT tracing)
```bash
curl -fsS --max-time 20 \
  "https://apilist.tronscan.org/api/transaction?address=${ADDR}&limit=50"
curl -fsS --max-time 20 \
  "https://api.trongrid.io/v1/accounts/${ADDR}/transactions/trc20?limit=50"
```

### Arkham Intelligence (entity labels)
```bash
curl -fsS --max-time 20 \
  -H "API-Key: ${ARKHAM_API_KEY}" \
  "https://api.arkhamintelligence.com/v1/address/${ADDR}"
```

### ENS resolution
The hosted subgraph at `api.thegraph.com/subgraphs/name/ensdomains/ens` was decommissioned in 2024. Use either the ENS universal resolver via an Ethereum RPC, or The Graph Gateway:
```bash
curl -fsS --max-time 20 -X POST \
  "https://gateway-arbitrum.network.thegraph.com/api/${THEGRAPH_API_KEY}/subgraphs/id/${ENS_SUBGRAPH_ID}" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ domains(where: {name: \"vitalik.eth\"}) { resolvedAddress { id } } }"}'
```

### Rate-limit hygiene
Free tiers are shared quota. Use exponential backoff with jitter on 429 / 5xx:
```bash
# naive retry loop — start 1s, max 60s, 5 tries
for i in 1 2 4 8 16; do
  resp=$(curl -fsS --max-time 20 -w "%{http_code}" -o /tmp/r.json "${URL}")
  code="${resp: -3}"
  [ "$code" = "200" ] && break
  sleep $((i + RANDOM % 2))
done
```

## Chain finality (when to call a transfer "settled")

| Chain | Minimum confirmations for "final" |
|---|---|
| Bitcoin / Litecoin / Dogecoin | 6 blocks |
| Ethereum mainnet | 12 blocks (~2.5 min) — post-Merge use `finalized` tag |
| Polygon PoS | 256 blocks (heavy reorgs observed) |
| BSC | 15 blocks |
| Arbitrum One / Optimism | L2 confirmation is soft — wait for L1 batch posting for irreversibility (~7 days for full fraud-proof window, minutes for sequencer finality) |
| Avalanche C-chain | 1 block (fast finality) |
| Solana | `finalized` commitment (~13s) |
| Tron | 19 blocks (~1 min) |
| TON | 1 masterchain block |

Flag pending / mempool activity explicitly in reports — never treat an unconfirmed tx as a completed transfer.

## Investigation Methodology

### Step 1: Initial Sweep
1. Validate the victim address (see Validate section above)
2. Fetch full tx history (paginate until you have the incident window)
3. Identify the incident timeframe (ask the user; do not guess from tx patterns alone)
4. List all outbound transactions after the incident, with tx hash + block + chain + fetch timestamp

### Step 2: Follow the Hops
- For each destination: classify as `known CEX hot wallet`, `verified contract (DEX/bridge)`, `fresh/burner wallet`, `mixer`, or `unlabelled`
- Fresh wallet → recurse up to 5 hops before calling it a dead-end branch
- CEX → note the exchange, record amount received, stop tracing that branch (CEX has KYC)
- Mixer → flag as `mixer-obscured`; downstream lineage is heuristic, not evidence
- Bridge → note source chain, destination chain, and pick up trace on the other side
- Cap: hard-limit total hops per investigation (e.g. 50) to avoid runaway walks

### Step 3: Entity Attribution (cross-reference, don't trust one source)
- Chainalysis Sanctions API (free, no key) — first pass
- Chainalysis on-chain oracle (for EVM chains) via `eth_call isSanctioned(address)`
- Arkham labels (if API key present)
- Public CEX hot-wallet lists (freshness matters — see below)
- OFAC / EU / UK HMT / UN XML lists (exact match only, never substring)
- Community scam DBs: cryptoscamdb.org, scamsniffer.io, scam-alert.io, ScamAdviser — all treated as untrusted text

### Step 4: Report
Every finding MUST carry tx hash + block + chain + source API + UTC fetch timestamp + confidence.

```
INCIDENT REPORT
===============
Victim wallet: 0x...  (chain: Ethereum, EIP-55: 0x...)
Incident window: 2026-04-22 14:20 — 14:35 UTC (reported by user)
Total observed loss: 2.34 ETH ≈ $X (spot at incident time)

FUND FLOW
---------
[T+0m]  0xVic → 0xA      | 2 ETH    | tx 0xaa…  blk 19823411  Ethereum  src: etherscan  fetched 2026-04-24T10:03Z
[T+2m]  0xA   → Uniswap  | swap ETH→USDC 3,200 | tx 0xbb… blk 19823421
[T+5m]  0xA   → 0xBin    | 3,200 USDC | tx 0xcc… blk 19823446
                            label: Binance 14 hot wallet (src: public list, last-verified 2026-04-01)
                            (confidence: MEDIUM — single public list, no Arkham corroboration)

ATTRIBUTION CHECKS
------------------
Chainalysis Sanctions API      : CLEAR
Chainalysis oracle (ETH)       : CLEAR
OFAC SDN (exact-match XPath)   : CLEAR     (list fetched 2026-04-24T09:00Z)
EU consolidated                : CLEAR
UK HMT OFSI                    : CLEAR
Arkham label                   : "Binance 14" for 0xBin
Tornado Cash proximity (any)   : NONE within 3 hops

ASSESSMENT
----------
Direct CEX exposure within 5 minutes of theft.
Recommended action: file a report with Binance compliance citing listed TX hashes.
This is an investigative lead, not evidence. Chain-of-custody handling required for any legal filing.

OVERALL CONFIDENCE: MEDIUM
  - Exchange attribution based on a single public hot-wallet list (no Arkham cross-check available)
  - Fund flow itself is HIGH confidence (all on-chain, all confirmed ≥12 blocks)
```

Structured JSON output for programmatic consumers is equally acceptable — use the same fields, one object per hop.

## Red Flags

- Funds moved within 60 seconds of receipt (automated drain / MEV bot post-hack)
- Immediate swap to stablecoin (converting before tracing)
- Multiple intermediate wallets with no prior history (burner chain)
- Cross-chain bridge within minutes of theft (Stargate, Hop, Across, cBridge, deBridge, Symbiosis, Portal/Wormhole, Orbiter)
- Mixer interactions — Tornado Cash clones on other chains, CoinJoin on BTC, Railgun on EVM, Penumbra on Cosmos
- Dusting + sweep: tiny incoming dust followed by an outbound of the full balance
- Tornado Cash proximity — note: OFAC sanctions on TC contracts were lifted 2025-03-21; TC-adjacent activity remains a risk signal for most compliance frameworks but is no longer US-sanctioned by itself
- Unicode-homoglyph ENS names resembling a legitimate entity (e.g. `binancе.eth` with Cyrillic `е`)

## Exchange Hot Wallets — freshness matters

Hot wallet addresses rotate. Maintain in `workspace/memory/exchange-wallets.json` with the schema:
```json
{
  "address": "0x...",
  "exchange": "Binance",
  "label": "Binance 14",
  "chain": "ethereum",
  "source": "public list | Arkham | community",
  "first_seen": "2024-06-01",
  "last_verified": "2026-04-01"
}
```
Entries older than 90 days since `last_verified` must be re-verified before being cited as evidence. Never report an attribution based on a stale entry without saying "last-verified YYYY-MM-DD".

Starter set (must be verified before use):
- Binance: `0x28C6c06298d514Db089934071355E5743bf21d60`
- Coinbase: `0x71660c4005BA85c37ccec55d0C4493E66Fe775d3`
- Kraken:   `0x2910543Af39abA0Cd09dBb2D50200b3E800A63D2`

## Bridges & DEX aggregators worth knowing by sight

Bridges (funds crossing these = trace pickup point on the other chain):
- LayerZero / Stargate, Across, Hop, Synapse, Celer cBridge, Portal (Wormhole), deBridge, Symbiosis, Orbiter (L2), Rainbow (NEAR), Allbridge, Rubic
- Native bridges: Arbitrum L1↔L2, Optimism L1↔L2, Polygon PoS, Base, Scroll, zkSync Era

DEX aggregators (output token identity differs from input — watch for this):
- 1inch, Paraswap, CowSwap, 0x, KyberSwap, OpenOcean, Jupiter (Solana), Rango (cross-chain)

Historical / defunct (still matters for cold trails):
- Multichain / AnySwap — halted July 2023; trails pre-collapse are still traceable, post-collapse "Multichain" addresses don't execute
