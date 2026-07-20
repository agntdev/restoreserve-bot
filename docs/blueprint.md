# RestoReserve — Bot specification

**Archetype:** booking

**Voice:** professional and warm — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for restaurant table reservations with availability checks, booking codes, and owner management. Guests reserve via date/time/party size flow with real-time availability. Owners get compact views of reservations, remaining seats, and no-show controls. All with inline buttons and reminders.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant owners/staff

## Success criteria

- 100% accurate availability checks with no double-bookings
- owners can view daily seat availability at a glance
- guests receive timely reminders before reservations

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main reservation menu with party size buttons
- **Reschedule** (button, actor: user, callback: reschedule:booking_code) — Initiate rescheduling flow for existing reservation
  - inputs: booking_code
  - outputs: new_time_slots
- **Cancel** (button, actor: user, callback: cancel:booking_code) — Cancel existing reservation with confirmation
  - inputs: booking_code
  - outputs: cancellation_confirmation
- **/owner** (command, actor: owner, command: /owner) — Open owner dashboard with today's summary and reservation list

## Flows

### Guest reservation
_Trigger:_ /start

1. Select party size
2. Choose date
3. Select time slot
4. Enter optional contact info
5. Confirm booking with code

_Data touched:_ reservation, restaurant_config

### Owner management
_Trigger:_ /owner

1. View today's summary
2. List upcoming reservations
3. Mark no-show/cancel
4. Receive real-time updates

_Data touched:_ reservation, restaurant_config

### Rescheduling
_Trigger:_ reschedule:booking_code

1. Validate booking code
2. Show available time slots
3. Confirm reschedule

_Data touched:_ reservation

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **restaurant_config** _(retention: persistent)_ — Restaurant operating parameters
  - fields: opening_hours, slot_duration, reminder_lead_time, table_types, booking_window
- **reservation** _(retention: persistent)_ — Confirmed or pending table booking
  - fields: booking_code, party_size, date, time, table_type, status, guest_name, guest_phone
- **owner_account** _(retention: persistent)_ — Private owner access credentials
  - fields: telegram_chat_id, restaurant_id

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View all reservations
- Mark no-shows
- Cancel bookings
- See daily seat availability
- Configure restaurant settings

## Notifications

- Guest pre-visit reminders
- Owner new booking alerts
- Owner cancellation alerts
- Owner daily seat summary

## Permissions & privacy

- Guest contact info visible only in owner private chat
- Booking codes unique within retention period
- No guest data shared with third parties

## Edge cases

- Party size exceeding table capacity
- Invalid date/time inputs
- Conflicting reservations during rescheduling
- Multiple owners/staff access (future extension)

## Required tests

- End-to-end reservation flow with availability checks
- Owner no-show marking triggers guest notification
- Rescheduling doesn't create overlapping bookings

## Assumptions

- Default 30-day booking window used if not configured
- Reminder lead time defaults to 2 hours
- Table splitting disabled by default
- Single owner account initially
