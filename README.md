
# ALADDIN Developer API

**Base URL:** `https://statesofglory.com`

ALADDIN exposes a read-only REST API so developers can build bots, scripts, and tools on top of live AHD market data — without managing their own scraping or game API rate limits.

Data is refreshed every hour by the ALADDIN pipeline. All endpoints return the latest completed snapshot.

---

## Getting Access

API keys are issued by the ALADDIN admin. Contact **stereotypicalsam** on Discord with a description of what you're building.

Your key looks like this:
```
alad_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**The key is shown once at creation and cannot be recovered.** If you lose it, a new one has to be issued.

---

## Authentication

Include your key as a Bearer token on every request:

```
Authorization: Bearer alad_your_key_here
```

**Example:**
```bash
curl https://statesofglory.com/api/v1/snapshot \
  -H "Authorization: Bearer alad_your_key_here"
```

Missing or invalid keys return `401`. Revoked keys return `403`.

---

## Rate Limiting

Each key is capped at **120 requests per minute** by default.

If you exceed the limit, you'll get:
```
HTTP 429 Too Many Requests
Retry-After: 14
```

The `Retry-After` value is the number of seconds until the current window resets. The window is per UTC minute — it resets at the top of the next minute, not 60 seconds from your first request.

---

## Endpoints

### `GET /api/v1/snapshot`

Metadata about the most recent pipeline run.

```bash
curl https://statesofglory.com/api/v1/snapshot \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "fetchedAt": "2026-04-29T02:05:14",
  "playerCount": 84,
  "corpCount": 61,
  "totalWealth": 4821930000.0,
  "gini": 0.63
}
```

Use `fetchedAt` to know how fresh the data is before making further calls.

---

### `GET /api/v1/corps`

All corporations from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/corps \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 61,
  "corps": [
    {
      "corpId": "abc123",
      "sequentialId": 42,
      "name": "Nexus Corp",
      "type": "corporation",
      "typeLabel": "Corporation",
      "countryId": "US",
      "homeState": "New York",
      "currencyCode": "USD",
      "sharePrice": 148.50,
      "totalShares": 1000000,
      "publicFloat": 420000,
      "marketCap": 148500000.0,
      "marketCapUsd": 148500000.0,
      "totalRevenue": 9200000.0,
      "income": 1400000.0,
      "incomeUsd": 1400000.0,
      "liquidCapital": 3100000.0,
      "liquidCapitalUsd": 3100000.0,
      "priceChange1h": 0.8,
      "priceChange24h": -2.1,
      "priceChange48h": 1.4,
      "avgSectorGrowth": 3.2,
      "dividendRate": 0.04,
      "isNatcorp": false,
      "isNationalized": false,
      "ceoName": "Kaldr",
      "ceoSequentialId": 17,
      "ceoVacant": false,
      "shareholderCount": 8,
      "sectorCount": 3
    }
  ]
}
```

---

### `GET /api/v1/corps/{corp_id}`

Single corporation with the last 20 turns of financial history.

```bash
curl https://statesofglory.com/api/v1/corps/abc123 \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "corp": { ...same fields as above... },
  "history": [
    {
      "turn": 388,
      "sharePrice": 141.20,
      "marketCap": 141200000.0,
      "liquidCapital": 2900000.0,
      "revenue": 8800000.0,
      "income": 1200000.0,
      "dividendRate": 0.04,
      "creditRating": "BBB",
      "creditComposite": 58.4,
      "marketingStrength": 72.1
    },
    ...
  ]
}
```

History is ordered oldest → newest. Returns `404` if the corp ID doesn't exist in the latest snapshot.

---

### `GET /api/v1/players`

All players from the latest snapshot, sorted by wealth rank.

```bash
curl https://statesofglory.com/api/v1/players \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 84,
  "players": [
    {
      "characterId": "69ef1355c89cc52094f9e214",
      "sequentialId": 7,
      "name": "Kaldr",
      "state": "New York",
      "country": "US",
      "corporation": "Nexus Corp",
      "rank": 1,
      "totalWealth": 48200000.0,
      "cashValue": 12100000.0,
      "stockValue": 31400000.0,
      "bondValue": 4700000.0,
      "portfolioValue": 36100000.0,
      "wealthChange24h": 1240000.0,
      "rankChange24h": 2,
      "stockHoldings": 4,
      "bondHoldings": 2,
      "historyTurns": 31,
      "borderKey": "US",
      "avatarUrl": "https://..."
    }
  ]
}
```

---

### `GET /api/v1/players/{character_id}`

Single player detail. Use `characterId` from the players list as the path parameter.

```bash
curl https://statesofglory.com/api/v1/players/69ef1355c89cc52094f9e214 \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "player": { ...same fields as above... }
}
```

Returns `404` if the character ID doesn't exist in the latest snapshot.

---

### `GET /api/v1/players/{character_id}/history`

Wealth history for a single player — last 50 turns of net worth, cash, stock, and bond values. Useful for spotting whether a party member is growing, stable, or in decline.

```bash
curl https://statesofglory.com/api/v1/players/69ef1355c89cc52094f9e214/history \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "characterId": "69ef1355c89cc52094f9e214",
  "name": "Kaldr",
  "currentWealth": 48200000.0,
  "currentRank": 1,
  "history": [
    {
      "turn": 370,
      "totalValue": 41000000.0,
      "stockValue": 28000000.0,
      "bondValue": 4200000.0,
      "cashValue": 8800000.0,
      "liquidCashValue": 6100000.0,
      "savingsCashValue": 2700000.0
    },
    ...
  ]
}
```

History is ordered oldest → newest.

---

### `GET /api/v1/elections`

All active elections from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/elections \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/elections?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 2,
  "elections": [
    {
      "country": "US",
      "type": "presidential",
      "status": "active",
      "candidates": [
        { "name": "Kaldr", "votes": 412, "party": "Republican Alliance" },
        { "name": "OtherPlayer", "votes": 388, "party": "Democratic Front" }
      ]
    }
  ]
}
```

---

### `GET /api/v1/government`

Government officials and formation for each country from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/government \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/government?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 4,
  "governments": [
    {
      "country": "US",
      "countryName": "United States",
      "officials": [
        { "name": "Kaldr", "role": "President", "party": "Republican Alliance", "influence": 8420 }
      ],
      "governmentFormation": {
        "pmName": "Kaldr",
        "coalitionSeats": 218,
        "totalSeats": 435,
        "partyBreakdown": [...]
      }
    }
  ]
}
```

---

### `GET /api/v1/influence`

Political influence leaderboard — top 25 players per country, ranked by influence score.

```bash
curl https://statesofglory.com/api/v1/influence \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/influence?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 25,
  "leaders": [
    {
      "country": "US",
      "rank": 1,
      "name": "Kaldr",
      "influence": 8420,
      "profileUrl": "https://www.ahousedividedgame.com/character/17"
    }
  ]
}
```

---

### `GET /api/v1/shareholders`

Full ownership table from the latest snapshot. Shows every stake in every corporation.

```bash
curl https://statesofglory.com/api/v1/shareholders \
  -H "Authorization: Bearer alad_your_key_here"
```

**Filter by corporation** (cap table for one corp):
```bash
curl "https://statesofglory.com/api/v1/shareholders?corp_id=abc123" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Filter by holder** (everything one player or corp owns):
```bash
curl "https://statesofglory.com/api/v1/shareholders?holder_id=69ef1355c89cc52094f9e214" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 3,
  "shareholdings": [
    {
      "corpId": "abc123",
      "corpName": "Nexus Corp",
      "corpCurrency": "USD",
      "holderType": "character",
      "holderId": "69ef1355c89cc52094f9e214",
      "holderName": "Kaldr",
      "shares": 210000.0,
      "sharesPct": 21.0,
      "sharePrice": 148.50,
      "stakeValue": 31185000.0,
      "stakeValueUsd": 31185000.0,
      "isImperial": false
    }
  ]
}
```

`holderType` is either `"character"` (a player) or `"corporation"` (a corp holding shares in another corp).

---

## Field Reference

### Corp fields

| Field | Type | Description |
|-------|------|-------------|
| `corpId` | string | Unique game ID |
| `sequentialId` | int | Short numeric ID (used in game URLs) |
| `name` | string | Corporation name |
| `typeLabel` | string | Human-readable type (e.g. "Corporation", "National Corp") |
| `countryId` | string | Country code (US, UK, DE, JP) |
| `sharePrice` | float | Current share price in local currency |
| `marketCapUsd` | float | Market cap converted to USD |
| `incomeUsd` | float | Net income converted to USD |
| `priceChange24h` | float | % price change over last 24h |
| `dividendRate` | float | Dividend yield (e.g. 0.04 = 4%) |
| `ceoVacant` | bool | `true` if the CEO seat is empty |
| `isNatcorp` | bool | Government-owned national corp |
| `isNationalized` | bool | Previously private corp, now nationalized |

### Player fields

| Field | Type | Description |
|-------|------|-------------|
| `characterId` | string | Unique game ID |
| `sequentialId` | int | Short numeric ID (used in game URLs) |
| `rank` | int | Wealth rank (1 = richest) |
| `totalWealth` | float | Net worth in USD |
| `cashValue` | float | Liquid cash in USD |
| `stockValue` | float | Total stock portfolio value in USD |
| `bondValue` | float | Total bond holdings value in USD |
| `wealthChange24h` | float | Absolute wealth change in USD over 24h |
| `rankChange24h` | int | Rank positions gained (positive = moved up) |

---

## Error Responses

All errors return JSON with a `detail` field:

```json
{ "detail": "Invalid API key." }
```

| Status | Meaning |
|--------|---------|
| `401` | Missing, malformed, or invalid API key |
| `403` | API key has been revoked |
| `404` | Requested resource not found in the latest snapshot |
| `429` | Rate limit exceeded — check `Retry-After` header |
| `503` | No snapshot available yet (pipeline hasn't run) |

---

## Tips

**Avoid hammering the API.** Data only updates once per hour. Cache the response locally and re-fetch after `fetchedAt` is more than 60 minutes old.

**Use `snapshot` first.** Check `GET /api/v1/snapshot` to get the current `snapshotId` and `fetchedAt` before pulling large datasets — this lets you skip re-fetching if the data hasn't changed since your last pull.

**Find a player's corp stake in one call:**
```bash
# All shares held by a specific player
curl "https://statesofglory.com/api/v1/shareholders?holder_id=<characterId>" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Find who controls a corp:**
```bash
# Full cap table sorted by stake %
curl "https://statesofglory.com/api/v1/shareholders?corp_id=<corpId>" \
  -H "Authorization: Bearer alad_your_key_here"
```

---

## Python Example

```python
import requests

API_KEY = "alad_your_key_here"
BASE    = "https://statesofglory.com"
HEADERS = {"Authorization": f"Bearer {API_KEY}"}

# Get latest snapshot metadata
snap = requests.get(f"{BASE}/api/v1/snapshot", headers=HEADERS).json()
print(f"Data as of: {snap['fetchedAt']}")

# Get all players
players = requests.get(f"{BASE}/api/v1/players", headers=HEADERS).json()
for p in players["players"][:5]:
    print(f"#{p['rank']}  {p['name']}  ${p['totalWealth']:,.0f}")

# Get cap table for a specific corp
corp_id = "abc123"
holders = requests.get(
    f"{BASE}/api/v1/shareholders",
    headers=HEADERS,
    params={"corp_id": corp_id}
).json()
for h in holders["shareholdings"]:
    print(f"{h['holderName']}  {h['sharesPct']:.1f}%")
```

---

*Questions or issues? Contact stereotypicalsam on Discord.*
