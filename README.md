
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

Each key is capped at **100 requests per hour** by default. Since ALADDIN data refreshes once per hour, there is no benefit to polling more frequently than that — the response will be identical until the next pipeline run.

If you exceed the limit, you'll get:
```
HTTP 429 Too Many Requests
Retry-After: 1823
```

The `Retry-After` value is the number of seconds until the current window resets. The window is per UTC hour — it resets at the top of the next hour, not 3600 seconds from your first request.

**Practical usage:** fetch `/api/v1/snapshot` first to check `fetchedAt`. If it hasn't changed since your last pull, skip re-fetching everything else — you'll burn your hourly quota for no new data.

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
  "gini": 0.63,
  "gameTurn": 438,
  "gameYear": 2029,
  "nextTurnAt": "2026-04-29T06:00:00.000Z",
  "turnIsProcessing": false
}
```

Use `fetchedAt` to know how fresh the data is before making further calls. `nextTurnAt` tells you when the game will process the next turn — useful for scheduling refetches.

---

### `GET /api/v1/corps`

All corporations from the latest snapshot, including credit rating data.

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
      "sectorCount": 3,
      "creditRating": "BBB",
      "creditScore": 58.4,
      "totalDebt": 4200000.0,
      "netWorth": 18700000.0
    }
  ]
}
```

`creditRating` is `null` if ALADDIN has not yet computed a rating for the corp (newly listed or insufficient history). Prediction signals (`BUY`, `SELL`, etc.) are available in the dashboard but are not currently exposed via the developer API.

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
  "corp": {
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
    "sectorCount": 3,
    "creditRating": "BBB",
    "creditScore": 58.4,
    "totalDebt": 4200000.0,
    "netWorth": 18700000.0
  },
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
    }
  ]
}
```

History is ordered oldest → newest, up to 20 turns. Returns `404` if the corp ID doesn't exist in the latest snapshot.

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
      "avatarUrl": "https://www.ahousedividedgame.com/avatars/..."
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `portfolioValue` | `stockValue + bondValue` — does not include cash |
| `wealthChange24h` | Absolute USD change in `totalWealth` since 24 hours ago |
| `rankChange24h` | Positions gained on the wealth leaderboard (positive = moved up) |
| `stockHoldings` | Number of distinct corporations the player holds shares in |
| `bondHoldings` | Number of distinct bond positions held |
| `historyTurns` | How many turns of wealth history ALADDIN has recorded for this player — useful for knowing whether trend data is meaningful |
| `borderKey` | Country code used for flag display (`US`, `UK`, `DE`, `JP`, `IE`, `BR`, `CN`) — equivalent to `country` for most players |

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
  "player": {
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
    "avatarUrl": "https://www.ahousedividedgame.com/avatars/..."
  }
}
```

Returns `404` if the character ID doesn't exist in the latest snapshot.

---

### `GET /api/v1/players/{character_id}/history`

Wealth history for a single player — up to the last 50 turns of net worth, cash, stock, and bond values. If fewer than 50 turns of history exist, returns whatever is available (no 404 — the player object is still returned with an empty or short `history` array).

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
    }
  ]
}
```

History is ordered oldest → newest. `liquidCashValue` is cash in the player's wallet; `savingsCashValue` is cash held in a savings account. `totalValue = liquidCashValue + savingsCashValue + stockValue + bondValue`.

---

### `GET /api/v1/shareholders`

Full ownership table from the latest snapshot. Returns every stake in every corporation. Supports filtering by corporation or holder.

```bash
# All shareholdings
curl https://statesofglory.com/api/v1/shareholders \
  -H "Authorization: Bearer alad_your_key_here"

# Cap table for one corp — who owns what %
curl "https://statesofglory.com/api/v1/shareholders?corp_id=abc123" \
  -H "Authorization: Bearer alad_your_key_here"

# All holdings by one player or corp
curl "https://statesofglory.com/api/v1/shareholders?holder_id=69ef1355c89cc52094f9e214" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Query params:** `?corp_id=<id>` · `?holder_id=<id>` — both are optional and can be combined.

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

| Field | Description |
|-------|-------------|
| `holderType` | `"character"` (a player) or `"corporation"` (a corp holding shares in another corp) |
| `shares` | Raw share count held |
| `sharesPct` | Percentage of total shares (`shares / totalShares * 100`) |
| `stakeValue` | Value in the corp's local currency (`shares × sharePrice`) |
| `stakeValueUsd` | Same value converted to USD |
| `isImperial` | `true` for special government-granted imperial shares that cannot be sold |

Results are sorted by corp, then by `sharesPct` descending (largest holder first within each corp).

---

### `GET /api/v1/elections`

All elections from the latest snapshot — active, upcoming, and recently completed.

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
      "electionType": "presidential",
      "status": "active",
      "state": null,
      "seatId": null,
      "endTime": "2026-05-02T00:00:00Z",
      "candidates": [
        {
          "characterName": "Kaldr",
          "party": "republican-alliance",
          "partyName": "Republican Alliance",
          "partyColor": "#c0392b",
          "votes": 412
        }
      ]
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `electionType` | `"presidential"`, `"senate"`, `"house"`, `"governor"`, `"senate_runoff"`, etc. |
| `status` | `"active"` (voting open), `"upcoming"` (not yet started), `"completed"` (results final) |
| `state` | State/region name for sub-national races; `null` for national races |
| `seatId` | Internal seat identifier for house/senate races; `null` for presidential/governor |
| `endTime` | ISO 8601 timestamp when voting closes; may be `null` for upcoming elections |

---

### `GET /api/v1/government`

Government officials for each country — one row per cabinet/executive position.

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
  "count": 18,
  "governments": [
    {
      "country": "US",
      "countryName": "United States",
      "characterName": "Kaldr",
      "characterId": "69ef1355c89cc52094f9e214",
      "role": "President",
      "section": "Executive",
      "party": "republican-alliance",
      "partyName": "Republican Alliance",
      "partyColor": "#c0392b",
      "profileUrl": "https://www.ahousedividedgame.com/character/17",
      "isVacant": false
    }
  ]
}
```

Each item in `governments` is a single official position. `isVacant: true` means the seat exists but no player holds it. Use `/api/v1/elections/composition` for coalition seat counts and parliament breakdown.

---

### `GET /api/v1/influence`

Political influence leaderboard — all players per country, ranked by influence score. Includes current influence, national influence, and favorability for each player.

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
      "politicalInfluence": 84.2,
      "nationalPoliticalInfluence": 237.8,
      "favorability": 71.4,
      "profileUrl": "https://www.ahousedividedgame.com/character/17"
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `politicalInfluence` | Normalised current influence score (0–100) within the current state or district |
| `nationalPoliticalInfluence` | Raw national influence score — comparable across all players in the country |
| `favorability` | Public favorability rating (0–100) |

---

### `GET /api/v1/compass`

Political compass positions for all parties and players. Positions use `economicPosition` (left/right) and `socialPosition` (progressive/conservative) on a **−5 to +5 scale**.

> **Note:** Player `economicPosition` and `socialPosition` are each player's personal political values — distinct from and independent of their party's position.

```bash
curl https://statesofglory.com/api/v1/compass \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/compass?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "partyCount": 8,
  "playerCount": 42,
  "parties": [
    {
      "country": "US",
      "partyId": "republican-alliance",
      "name": "Republican Alliance",
      "abbreviation": "RA",
      "color": "#c0392b",
      "economicPosition": 3,
      "socialPosition": 2,
      "memberCount": 14,
      "playerCount": 10,
      "treasury": 2400000.0,
      "chair": { "name": "Kaldr", "characterId": "69ef..." }
    }
  ],
  "players": [
    {
      "country": "US",
      "rank": 1,
      "name": "Kaldr",
      "profileUrl": "https://www.ahousedividedgame.com/character/17",
      "party": "Republican Alliance",
      "partyColor": "#c0392b",
      "politicalInfluence": 84.2,
      "nationalPoliticalInfluence": 237.8,
      "favorability": 71.4,
      "infamy": 0.0,
      "funds": 5718706,
      "actions": 10,
      "donorBaseLevel": 53,
      "isCeo": true,
      "ceoOf": "Nexus Corp",
      "investorRank": 2,
      "activeElection": { "electionId": "69f1...", "electionType": "senate", "electionState": "CA", "enteredAt": "2026-04-28T23:00:00.000Z" },
      "economicPosition": 3,
      "socialPosition": 2
    }
  ]
}
```

**Compass scale (−5 to +5):**

| economicPosition | Label |
|-----------------|-------|
| −5 to −3 | Far Left |
| −2 to −1 | Centre-Left |
| 0 | Centre |
| 1 to 2 | Centre-Right |
| 3 to 5 | Far Right |

| socialPosition | Label |
|---------------|-------|
| −5 to −3 | Very Progressive |
| −2 to −1 | Moderate Progressive |
| 0 | Neutral |
| 1 to 2 | Traditional |
| 3 to 5 | Very Conservative |

Players with no compass data yet will have `economicPosition: null` and `socialPosition: null` (populated on first pipeline run after key issuance).

---

### `GET /api/v1/seats`

Current and projected seat distributions for all legislative chambers.

```bash
curl https://statesofglory.com/api/v1/seats \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/seats?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 6,
  "chambers": [
    {
      "country": "US",
      "chamber": "senate",
      "chamberName": "U.S. Senate",
      "totalSeats": 100,
      "inGeneral": false,
      "current": [
        { "party": "2", "partyName": "Republican Party", "partyColor": "#EF4444", "seats": 42 },
        { "party": "1", "partyName": "Democratic Party", "partyColor": "#084bbe", "seats": 33 }
      ],
      "projected": [
        { "party": "2", "partyName": "Republican Party", "partyColor": "#EF4444", "seats": 44 },
        { "party": "1", "partyName": "Democratic Party", "partyColor": "#084bbe", "seats": 31 }
      ]
    }
  ]
}
```

Chambers covered: US senate, US house, UK commons, JP shugiin, JP sangiin, DE bundestag.

`current` = seats held right now. `projected` = model projection based on active elections.

---

### `GET /api/v1/elections/detail`

Active and upcoming election details with per-candidate compass positions (richer than `/elections` — sourced from the bot API).

```bash
curl https://statesofglory.com/api/v1/elections/detail \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/elections/detail?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 5,
  "elections": [
    {
      "country": "US",
      "electionId": "69e72ef4fd958232862495d3",
      "electionType": "senate",
      "state": "NE",
      "stateName": "Nebraska",
      "status": "active",
      "startTime": "2026-04-27T00:00:00.000Z",
      "endTime": "2026-05-01T00:00:00.000Z",
      "phase": { "inPrimary": false, "inGeneral": true, "isUpcoming": false, "isEnded": false },
      "incumbent": { "name": "Anthony Soprano", "party": "Republican Party" },
      "candidates": [
        {
          "characterName": "Anthony Soprano",
          "party": "Republican Party",
          "partyColor": "#EF4444",
          "economicPosition": 1,
          "socialPosition": 1.5,
          "favorability": 60.0,
          "politicalInfluence": 23.1,
          "primaryScore": 75.2,
          "sharePct": 0,
          "endorsementCount": 0,
          "campaignFunds": null
        }
      ],
      "votes": null
    }
  ]
}
```

Only active and upcoming elections are returned. `votes` is populated once an election is finalized.

---

### `GET /api/v1/market/unowned`

Unowned market opportunity by sector type — revenue available for corporations to capture.

```bash
curl https://statesofglory.com/api/v1/market/unowned \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one sector
curl "https://statesofglory.com/api/v1/market/unowned?sector=energy" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 13,
  "sectors": [
    {
      "sectorType": "energy",
      "totalItems": 106,
      "states": [
        {
          "stateId": "CA",
          "stateName": "California",
          "countryId": "US",
          "totalMarket": 19500000,
          "ownedRevenue": 14200000,
          "unownedRevenue": 5300000
        }
      ]
    }
  ]
}
```

`unownedRevenue` is the daily revenue currently not captured by any corporation. Sector types: `financial`, `energy`, `defense`, `retail`, `technology`, `chemical_industries`, `healthcare`, `agriculture`, `automobiles`, `manufacturing`, `media`, `real_estate`, `construction`, `telecommunications`, `entertainment`, `logistics`, `extraction`.

---

### `GET /api/v1/states`

State and region detail for all 86 game states — officials, population, and voting system. Covers US (50 states), UK (12 regions), JP (8 regions), DE (16 Länder).

```bash
curl https://statesofglory.com/api/v1/states \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/states?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 86,
  "states": [
    {
      "country": "US",
      "stateId": "CA",
      "name": "California",
      "region": "West",
      "population": 39538223,
      "votingSystem": "fptp",
      "officials": [
        {
          "officeType": "senate",
          "characterId": "69e40a3a990c1761d957590c",
          "characterName": "14th Emperor",
          "party": "Democratic Party",
          "partyColor": "#084bbe",
          "isNPP": false
        },
        {
          "officeType": "governor",
          "characterId": null,
          "characterName": "Donna Bishop",
          "party": "Democratic Party",
          "partyColor": "#084bbe",
          "isNPP": true
        }
      ]
    }
  ]
}
```

`officeType` values include: `senate`, `house`, `governor`, `stateSenate`, `bundestag`, `ministerPresident`, `shugiin`, `sangiin`, `regionalCouncil`, `commons`, `lords`. `isNPP` is `true` for NPC/placeholder officials.

---

### `GET /api/v1/parties/{party_seq_id}/members`

All members of a party with their influence scores. Use `sequentialId` from `/api/v1/macro/parties` as the path parameter.

Results are sorted by `partyInfluence` descending — most influential first.

```bash
curl https://statesofglory.com/api/v1/parties/1/members \
  -H "Authorization: Bearer alad_your_key_here"

# If the same sequential ID exists in multiple countries, specify one
curl "https://statesofglory.com/api/v1/parties/1/members?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "country": "US",
  "partySeqId": 1,
  "partyName": "Republican Alliance",
  "count": 14,
  "members": [
    {
      "charId": "69ef1355c89cc52094f9e214",
      "charSeqId": 17,
      "name": "Kaldr",
      "homeState": "New York",
      "currentOffice": { "type": "president", "state": null, "senateClass": null },
      "partyInfluence": 8420
    }
  ]
}
```

`currentOffice` is `null` if the member holds no office. When present, `type` is one of `president`, `senator`, `governor`, `representative`, etc.

---

### `GET /api/v1/congress/members`

All congress members from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/congress/members \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to US Senate only
curl "https://statesofglory.com/api/v1/congress/members?country=US&chamber=senate" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 100,
  "members": [
    {
      "country": "US",
      "chamber": "senate",
      "charId": "69ef1355c89cc52094f9e214",
      "name": "Kaldr",
      "party": "republican-alliance",
      "partyName": "Republican Alliance",
      "state": "New York",
      "officeType": "senator",
      "isNpp": false,
      "isVacant": false,
      "electedAt": "2026-01-15T00:00:00"
    }
  ]
}
```

`isNpp` is `true` for non-player positions (AI-controlled). `isVacant` is `true` for empty seats.

---

### `GET /api/v1/congress/leaders`

Current congressional leadership positions from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/congress/leaders \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 4,
  "leaders": [
    {
      "name": "Kaldr",
      "role": "Speaker",
      "party": "republican-alliance",
      "partyName": "Republican Alliance",
      "state": "New York"
    }
  ]
}
```

---

### `GET /api/v1/congress/cabinet-nominations`

Active cabinet nominations from the latest snapshot.

```bash
curl https://statesofglory.com/api/v1/congress/cabinet-nominations \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 2,
  "nominations": [
    {
      "nominee": "PlayerName",
      "position": "Secretary of State",
      "status": "pending"
    }
  ]
}
```

---

### `GET /api/v1/elections/composition`

Current and projected seat counts by party for each chamber.

```bash
curl https://statesofglory.com/api/v1/elections/composition \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "currentHouse": [
    { "party": "republican-alliance", "partyName": "Republican Alliance", "seats": 230 },
    { "party": "democratic-front", "partyName": "Democratic Front", "seats": 205 }
  ],
  "currentSenate": [ ... ],
  "projectedHouse": [ ... ],
  "projectedSenate": [ ... ]
}
```

`currentHouse`/`currentSenate` reflect seated members. `projectedHouse`/`projectedSenate` factor in active elections and polling.

---

### `GET /api/v1/macro/parties`

Full party metadata including ideological compass positions, leadership, member counts, and treasury.

```bash
curl https://statesofglory.com/api/v1/macro/parties \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/macro/parties?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 8,
  "parties": [
    {
      "country": "US",
      "partyId": "republican-alliance",
      "sequentialId": 1,
      "name": "Republican Alliance",
      "abbreviation": "RA",
      "color": "#c0392b",
      "economicPosition": 3,
      "socialPosition": 2,
      "memberCount": 14,
      "playerCount": 10,
      "treasury": 2400000.0,
      "chair": { "name": "Kaldr", "characterId": "69ef..." },
      "viceChair": null,
      "isDefault": false
    }
  ]
}
```

---

### `GET /api/v1/macro/approval`

Government approval ratings per country with turn-by-turn history.

```bash
curl https://statesofglory.com/api/v1/macro/approval \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/macro/approval?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 7,
  "approvals": [
    {
      "country": "US",
      "approval": 54.2,
      "history": [
        { "turn": 385, "approval": 51.0, "net": 2.1 },
        { "turn": 386, "approval": 52.8, "net": 1.8 }
      ]
    }
  ]
}
```

---

### `GET /api/v1/macro/central-bank`

Central bank data per country — interest rates, inflation, GDP growth, balance sheet, and 240-turn histories.

```bash
curl https://statesofglory.com/api/v1/macro/central-bank \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one country
curl "https://statesofglory.com/api/v1/macro/central-bank?country=US" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Response:**
```json
{
  "snapshotId": 412,
  "count": 7,
  "centralBanks": [
    {
      "country": "US",
      "bankName": "Federal Reserve",
      "currencyCode": "USD",
      "primeRate": 5.25,
      "effectiveRate": 5.0,
      "currentInflation": 3.1,
      "inflationBreakdown": { "housing": 1.2, "energy": 0.4, ... },
      "interestRateHistory": [ { "turn": 385, "rate": 5.0 }, ... ],
      "inflationHistory": [ { "turn": 385, "inflation": 3.0 }, ... ],
      "gdpGrowthHistory": [ { "turn": 385, "growth": 2.1 }, ... ],
      "balanceSheet": { ... },
      "bankFinancials": { ... }
    }
  ]
}
```

---

### `GET /api/v1/macro/forex`

Forex rate history — per-currency turn-by-turn exchange rates against the base.

```bash
curl https://statesofglory.com/api/v1/macro/forex \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one currency, last 50 turns
curl "https://statesofglory.com/api/v1/macro/forex?currency=GBP&limit=50" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Query params:** `?currency=GBP` · `?limit=240` (default 240, max 500)

**Response:**
```json
{
  "count": 7,
  "currencies": {
    "USD": [ { "turn": 385, "rate": 1.0 }, ... ],
    "GBP": [ { "turn": 385, "rate": 0.79 }, ... ]
  }
}
```

---

### `GET /api/v1/macro/market-cap`

Global stock market capitalisation history with per-sector breakdown.

```bash
curl "https://statesofglory.com/api/v1/macro/market-cap?limit=50" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Query params:** `?limit=200` (default 200, max 500)

**Response:**
```json
{
  "count": 200,
  "points": [
    {
      "turn": 385,
      "marketCap": 4821930000.0,
      "high": 4950000000.0,
      "low": 4700000000.0,
      "bySector": {
        "financial": 820000000.0,
        "technology": 610000000.0,
        "energy": 480000000.0
      }
    }
  ]
}
```

---

### `GET /api/v1/macro/commodities`

Per-commodity data — global price, supply/demand balance, per-state prices, top producers, and price history.

```bash
curl https://statesofglory.com/api/v1/macro/commodities \
  -H "Authorization: Bearer alad_your_key_here"

# Filter to one commodity
curl "https://statesofglory.com/api/v1/macro/commodities?commodity=oil" \
  -H "Authorization: Bearer alad_your_key_here"
```

**Query params:** `?commodity=oil` — one of: `oil`, `coal`, `food`, `copper`, `iron`, `chemicals`, `plastics`, `electronics`, `energy`, `steel`

**Response:**
```json
{
  "snapshotId": 412,
  "count": 10,
  "commodities": [
    {
      "commodityType": "oil",
      "globalPrice": 84.20,
      "globalSupply": 12400000.0,
      "globalDemand": 11800000.0,
      "priceChange": 1.4,
      "statePrices": { "Texas": 82.10, "California": 85.40 },
      "topProducers": [ { "name": "OilCorp", "output": 840000 } ],
      "history": [ { "turn": 385, "price": 83.10, "supply": 12200000, "demand": 11600000 } ]
    }
  ]
}
```

---

## Field Reference

### Corp fields

| Field | Type | Description |
|-------|------|-------------|
| `corpId` | string | Unique game ID |
| `sequentialId` | int | Short numeric ID (used in game URLs) |
| `name` | string | Corporation name |
| `type` | string | Internal type key (e.g. `"corporation"`, `"natcorp"`) |
| `typeLabel` | string | Human-readable type (e.g. `"Corporation"`, `"National Corp"`) |
| `countryId` | string | Country code (`US`, `UK`, `DE`, `JP`, `IE`, `BR`, `CN`) |
| `homeState` | string | State/region the corp is headquartered in |
| `currencyCode` | string | Local currency (`USD`, `GBP`, `EUR`, `JPY`, etc.) |
| `sharePrice` | float | Current share price in local currency |
| `totalShares` | int | Total shares issued |
| `publicFloat` | int | Shares available for public trading (excludes locked/imperial shares) |
| `marketCap` | float | Market cap in local currency |
| `marketCapUsd` | float | Market cap converted to USD |
| `totalRevenue` | float | Total revenue in local currency |
| `income` | float | Net income in local currency |
| `incomeUsd` | float | Net income converted to USD |
| `liquidCapital` | float | Cash on hand in local currency |
| `liquidCapitalUsd` | float | Cash on hand converted to USD |
| `priceChange1h` | float | % price change over last 1 hour |
| `priceChange24h` | float | % price change over last 24 hours |
| `priceChange48h` | float | % price change over last 48 hours |
| `avgSectorGrowth` | float | Average growth rate across the corp's sectors |
| `dividendRate` | float | Dividend yield as a decimal (e.g. `0.04` = 4%) |
| `isNatcorp` | bool | `true` for government-owned national corps |
| `isNationalized` | bool | `true` for formerly-private corps that were nationalised |
| `ceoName` | string | Name of the current CEO; `null` if vacant |
| `ceoSequentialId` | int | Sequential ID of the CEO character; `null` if vacant |
| `ceoVacant` | bool | `true` if the CEO seat is empty |
| `shareholderCount` | int | Number of distinct shareholders |
| `sectorCount` | int | Number of sectors the corp operates in |
| `creditRating` | string | Letter rating (`AAA`, `AA`, `A`, `BBB`, `BB`, `B`, `CCC`, `D`); `null` if unrated |
| `creditScore` | float | Numeric credit composite score (0–100); `null` if unrated |
| `totalDebt` | float | Total outstanding debt in local currency |
| `netWorth` | float | Assets minus liabilities in local currency |

### Player fields

| Field | Type | Description |
|-------|------|-------------|
| `characterId` | string | Unique game ID |
| `sequentialId` | int | Short numeric ID (used in game URLs) |
| `name` | string | Player display name |
| `state` | string | Home state/region |
| `country` | string | Home country code |
| `corporation` | string | Name of the player's primary corporation; `null` if none |
| `rank` | int | Wealth rank (1 = richest) |
| `totalWealth` | float | Net worth in USD (`cash + stock + bonds`) |
| `cashValue` | float | Total cash in USD (`liquid + savings`) |
| `stockValue` | float | Total stock portfolio value in USD |
| `bondValue` | float | Total bond holdings value in USD |
| `portfolioValue` | float | `stockValue + bondValue` (excludes cash) |
| `wealthChange24h` | float | Absolute USD change in total wealth over 24 hours |
| `rankChange24h` | int | Rank positions gained (positive = moved up the leaderboard) |
| `stockHoldings` | int | Number of distinct corps the player holds shares in |
| `bondHoldings` | int | Number of distinct bond positions |
| `historyTurns` | int | Number of turns of wealth history recorded by ALADDIN |
| `borderKey` | string | Country code used for flag rendering — same as `country` for most players |
| `avatarUrl` | string | URL to the player's avatar image |

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
