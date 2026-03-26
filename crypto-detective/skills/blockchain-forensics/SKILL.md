# Blockchain Forensics Skill

## API Quick Reference

### Ethereum / EVM chains

```bash
# Get wallet transaction history (Etherscan, no key needed for basic use)
curl "https://api.etherscan.io/api?module=account&action=txlist&address={ADDRESS}&sort=asc"

# Token transfers (ERC-20)
curl "https://api.etherscan.io/api?module=account&action=tokentx&address={ADDRESS}&sort=asc"

# Internal transactions
curl "https://api.etherscan.io/api?module=account&action=txlistinternal&address={ADDRESS}"

# Same pattern for Polygon: api.polygonscan.com, BSC: api.bscscan.com
```

### Bitcoin
```bash
curl "https://blockchain.info/rawaddr/{ADDRESS}?limit=50"
```

### Solana
```bash
curl "https://public-api.solscan.io/account/transactions?account={ADDRESS}&limit=50"
```

### Arkham Intelligence (entity labels)
```bash
curl "https://api.arkhamintelligence.com/v1/address/{ADDRESS}" \
  -H "API-Key: {ARKHAM_API_KEY}"
```

### ENS Resolution
```bash
# Resolve ENS name to address
curl -X POST "https://api.thegraph.com/subgraphs/name/ensdomains/ens" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ domains(where: {name: \"vitalik.eth\"}) { resolvedAddress { id } } }"}'
```

## Investigation Methodology

### Step 1: Initial Sweep
1. Fetch full tx history for the target wallet
2. Identify the timeframe of interest (when did the incident occur?)
3. List all outbound transactions after the incident

### Step 2: Follow the Hops
- For each destination address: check if it's a known exchange, contract, or fresh wallet
- If fresh wallet → follow it recursively (max 5 hops before calling it a dead end)
- If exchange → note which exchange, calculate amount received → this is where tracing typically ends (CEX has KYC)
- If mixer/bridge → flag as obfuscation attempt

### Step 3: Entity Attribution
- Check Arkham labels for each address
- Check if address matches known exchange hot wallets (maintain list in memory)
- Check OFAC sanctions list
- Search web for address in known scam databases

### Step 4: Report
Format findings as:
```
INCIDENT REPORT
===============
Victim wallet: 0x...
Incident time: YYYY-MM-DD HH:MM UTC
Total lost: X ETH / $X USD

FUND FLOW
---------
[T+0m]  0x... (victim) → 0x... (unknown) — 2 ETH
[T+2m]  0x... → 0x... (Uniswap) — swapped ETH→USDC
[T+5m]  0x... → 0x... (Binance hot wallet) — 3,200 USDC

ASSESSMENT
----------
Funds reached Binance within 5 minutes.
Recommend reporting to Binance compliance with TX hashes.
Case reference TXs: 0x..., 0x...

CONFIDENCE: Medium (exchange attribution based on public hot wallet list)
```

## Red Flags to Watch For

- Funds moved within 60 seconds of receipt (automated drain)
- Immediate swap to stablecoin (converting before tracing)
- Multiple intermediate wallets with no prior history (burner chain)
- Tornado Cash contract interactions: `0x722122dF12D4e14e13Ac3b6895a86e84145b6967` (ETH mainnet)
- Cross-chain bridge contracts (Stargate, Hop, Across)

## Known Exchange Hot Wallets (sample)
- Binance: 0x28C6c06298d514Db089934071355E5743bf21d60
- Coinbase: 0x71660c4005BA85c37ccec55d0C4493E66Fe775d3
- Kraken: 0x2910543Af39abA0Cd09dBb2D50200b3E800A63D2
(Maintain full list in workspace/memory/exchange-wallets.json)
