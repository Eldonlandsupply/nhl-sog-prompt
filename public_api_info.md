# Public NHL Stats API Endpoints for Shots on Goal (SOG) & Analytics

This document summarises the publicly accessible endpoints in the official **NHL Statistics API** (`https://statsapi.web.nhl.com/api/v1`) that provide data needed to model players’ **shots on goal (SOG)** and related metrics.  The NHL API is open and does not require an API key.  These endpoints can be queried via HTTP GET and return JSON.

## Game-Level Endpoints

- **Game Feed (play by play)** – `GET /game/{gameId}/feed/live`.  This endpoint returns all available data for a particular game, including boxscore summary, play‑by‑play data and on‑ice coordinate information【686229575100768†L7534-L7543】.  The `liveData.plays.allPlays` array contains every event (shots, missed shots, goals, blocked shots, penalties, etc.), with timestamps and player/team identifiers.  Filtering events where `result.eventTypeId` is `"SHOT"` or `"GOAL"` yields shots on goal.  Each play also contains `coordinates.x` and `coordinates.y` for rink location, and `players` entries identifying the shooter and goalie.  Use this endpoint to build detailed shot logs and derive features such as shot distance, angle, rebound shots and shooter identity.

- **Game Boxscore** – `GET /game/{gameId}/boxscore`.  Returns the aggregated boxscore for a game.  It includes team‑level `teamSkaterStats` (goals, penalty minutes, shots, blocked shots, takeaways, giveaways, hits, power‑play statistics) as well as per‑player stats within each team【686229575100768†L7462-L7493】.  While less granular than the live feed, it is useful for quickly obtaining total shots on goal per team and per player in a single request.

- **Game Content** – `GET /game/{gameId}/content`.  Provides media associated with a game, such as highlights, recaps, and play clips【686229575100768†L7499-L7529】.  Although not directly used for SOG modelling, these videos may provide qualitative context.

- **Game Feed Differential** – `GET /game/{gameId}/feed/live/diffPatch?startTimeCode=YYYYMMDD_HHMM`.  Returns only the events occurring after a specified timestamp, allowing you to poll for updates during a live game【686229575100768†L7571-L7619】.

## Player & Team Endpoints

- **Player Summary** – `GET /people/{playerId}` returns a player’s basic demographic information and position【686229575100768†L7624-L7654】.

- **Player Statistics** – `GET /people/{playerId}/stats?stats=statsSingleSeason&season=YYYYYYYY`.  Retrieves a player’s season‑level statistics, including `shots`, `goals`, `assists`, `hits`, `penaltyMinutes`, time on ice and faceoff metrics【686229575100768†L7658-L7704】.  Changing the `stats` query parameter provides splits such as `homeAndAway`, `byMonth`, `vsTeam`, etc., enabling the calculation of splits by opponent【686229575100768†L7686-L7707】.  Use this endpoint to compute a player’s baseline shot rate, shooting percentage and usage trends.

- **Team Statistics** – `GET /teams/{teamId}/stats?stats=statsSingleSeason&season=YYYYYYYY`.  Returns team‑level metrics such as `shotsPerGame`, `shotsAllowed`, `powerPlayPercentage`, `penaltyKillPercentage`, and `faceOffsTaken`.  These metrics summarise a team’s offensive and defensive tendencies (pace, shot suppression) and can inform matchup adjustments.

- **Roster & Lineups** – `GET /teams/{teamId}/roster`.  Lists players currently on the roster, including their IDs.  Combined with `people/{playerId}` data, this endpoint can be used to build expected line combinations, though actual line assignments require external sources or beat reports.

- **Schedule** – `GET /schedule?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD`.  Provides game schedules and returns game IDs for the specified date range【686229575100768†L7758-L7772】.  Use this to identify upcoming games for daily slates and to fetch game feeds.

## Example Workflow

1. **Get Upcoming Games**: Query `/schedule?date=2025-10-22` to list today’s games and extract each `gamePk` (game ID).
2. **Fetch Live Data**: For each game ID, call `/game/{gamePk}/feed/live` and iterate through `liveData.plays.allPlays` to record events where `result.eventTypeId` is `"SHOT"` or `"GOAL"`.  Collect shooter, goalie, period, time, strength, shot location and outcome.
3. **Merge Player & Team Context**: Use `/people/{playerId}/stats` (season splits) to compute each shooter’s baseline shot rates by situation (5v5, PP).  Use `/teams/{teamId}/stats` to obtain opponent shot‑allowance metrics (shots allowed per game, penalty kill efficiency).
4. **Aggregate & Feature Engineer**: Combine shot events with player usage (time on ice), team tendencies, rest/travel context (via schedule), and rink coordinates to build features such as shot volume, shooting location distribution, expected power-play time, score effects and opponent suppression.

## Notes

- The NHL API is unofficially documented and may occasionally change without notice.  Always validate responses and handle missing fields gracefully.
- The API is rate‑limited but does not require authentication for basic calls.  For large‑scale ingestion, implement polite request spacing (e.g., 1–2 requests per second).
- For advanced metrics (e.g., Corsi, Fenwick, expected goals), you may need to derive them from the play‑by‑play data or use third‑party sources like **Natural Stat Trick** or **MoneyPuck**.  Those services do not offer official public APIs and often require scraping or paid access.

## Citation

Information about these endpoints (summaries and descriptions) is derived from the open‑source NHL API specification file【686229575100768†L7534-L7543】【686229575100768†L7658-L7704】 and related documentation【686229575100768†L7462-L7493】【686229575100768†L7758-L7772】.
