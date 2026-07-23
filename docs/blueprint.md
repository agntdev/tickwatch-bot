# Crypto Price Alert Bot — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot that lets users track cryptocurrency prices and set custom alerts for threshold crossings or percentage movements. Users manage watchlists with inline buttons and text input, receive on-demand price checks, and configure daily summaries, quiet hours, and per-coin cooldowns. The bot owner receives aggregate metrics about usage and top-fired alerts without exposing user identities.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual Telegram users interested in crypto tracking
- bot owner for analytics

## Success criteria

- users can manage private watchlists with alerts
- alerts are delivered accurately with cooldown suppression
- owner receives daily aggregate metrics without PII
- price checks show current and last-seen price deltas

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with watchlist management options
- **Add Coin** (button, actor: user, callback: watchlist:add) — Add a new coin to the watchlist with pre-seeded options or custom ticker
- **View Watchlist** (button, actor: user, callback: watchlist:view) — Display current watchlist items and alert rules
- **Set Morning Summary** (button, actor: user, callback: summary:configure) — Configure optional daily price summary time
- **Configure Quiet Hours** (button, actor: user, callback: quiet:configure) — Set start/end times for alert suppression
- **/price** (command, actor: user, command: /price) — Request current price for a specific ticker or full watchlist

## Flows

### Add Watchlist Item
_Trigger:_ watchlist:add

1. Show pre-seeded coin list with buttons
2. Handle button selection or free-form ticker input
3. Prompt to set alert rules (threshold/percentage)

_Data touched:_ User profile, Watchlist item

### Price Check
_Trigger:_ /price

1. Parse ticker parameter or default to full watchlist
2. Fetch current price from feed
3. Compare with last-seen price and format response

_Data touched:_ Watchlist item, User profile

### Alert Trigger
_Trigger:_ price_threshold_crossed

1. Validate against user's active rules
2. Check cooldown status for the coin
3. Send alert message if not in quiet hours
4. Update last-seen price and set cooldown

_Data touched:_ Watchlist item, Alert event

### Daily Summary
_Trigger:_ morning_summary_time

1. Gather all watchlist prices
2. Highlight coins with significant movements
3. Format summary with user's local time

_Data touched:_ User profile, Watchlist item

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — User-specific settings and preferences
  - fields: telegram_id, timezone, quiet_hours_start, quiet_hours_end, default_cooldown_hours, summary_time
- **Watchlist item** _(retention: persistent)_ — Tracked cryptocurrency and associated rules
  - fields: ticker, display_name, last_seen_price, alert_rules
- **Alert event** _(retention: session)_ — Record of triggered alerts for analytics
  - fields: timestamp, user_id, ticker, rule_type, old_price, new_price, percent_change
- **Alert rule** _(retention: persistent)_ — User-defined conditions for price alerts
  - fields: type, threshold_value, percent_value, timeframe_minutes

## Integrations

- **Crypto Price Feed** (required) — Fetch real-time price data for alerts and summaries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View aggregate metrics (active users, top-fired alerts)
- Configure system-wide defaults (e.g. default cooldown)

## Notifications

- Price alerts with absolute and percent changes
- Daily morning summaries with watchlist updates
- Aggregate metrics sent to owner's admin chat

## Permissions & privacy

- Store user profiles and watchlists securely with private access
- Aggregate metrics exclude user identifiers and PII
- Respect user-configured quiet hours and cooldown periods

## Edge cases

- Unrecognized ticker symbol entered during watchlist addition
- Price feed API failure during alert check
- Multiple alert rules triggering during active cooldown

## Required tests

- User adds Bitcoin with threshold rule and receives alert when price crosses it
- Price check for 'all' returns full watchlist with deltas
- Alert is queued during quiet hours and delivered after end time
- Owner receives top-fired alerts summary without user identifiers

## Assumptions

- Default currency is USD for price thresholds and displays
- 6-hour cooldown is the system default for all users
- Percentage rule timeframe defaults to 1 hour
- Morning summaries are optional and require explicit configuration
