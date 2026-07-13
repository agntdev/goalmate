# Goal Planner — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for planning and tracking goals with day/week/month scopes. Users can create single or recurring goals, receive reminders, mark completions, and view progress summaries. The bot supports scheduling, habit tracking, and optional CSV exports.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individuals
- personal productivity users
- Telegram users

## Success criteria

- Users can create and track goals with various scheduling options
- Users receive timely reminders and summaries
- Users can view progress and analytics for their goals

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and onboarding flow
- **New Goal** (button, actor: user, callback: goal:new) — Start the goal creation flow
- **/today** (command, actor: user, command: /today) — Show today's goals and instances
- **/week** (command, actor: user, command: /week) — Show this week's goals and instances
- **/month** (command, actor: user, command: /month) — Show this month's goals and instances
- **/habits** (command, actor: user, command: /habits) — Show all habit tracking goals
- **/export** (command, actor: user, command: /export) — Request CSV export of goals and instances
- **/delete_account** (command, actor: user, command: /delete_account) — Request deletion of user account and data

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Request timezone
3. Offer quick examples
4. Show main menu

_Data touched:_ User

### Create Goal
_Trigger:_ goal:new

1. Request title
2. Optional description
3. Choose schedule type
4. Configure recurrence (if applicable)
5. Set reminder time
6. Confirm and show summary

_Data touched:_ Goal, Schedule, Reminder

### View Goals
_Trigger:_ /today

1. Show today's goals and instances
2. Display action buttons (Done/Skip/Postpone/Details)

_Data touched:_ Instance

### View Goals
_Trigger:_ /week

1. Show this week's goals and instances
2. Display action buttons (Done/Skip/Postpone/Details)

_Data touched:_ Instance

### View Goals
_Trigger:_ /month

1. Show this month's goals and instances
2. Display action buttons (Done/Skip/Postpone/Details)

_Data touched:_ Instance

### View Habits
_Trigger:_ /habits

1. Show all habit tracking goals
2. Display progress and analytics

_Data touched:_ Habit

### Mark Done
_Trigger:_ instance:mark_done

1. Mark instance as done
2. Record timestamp
3. Optionally add note

_Data touched:_ Instance

### Postpone
_Trigger:_ instance:postpone

1. Offer postponement options (+1 day, next occurrence, pick date)
2. Update instance schedule

_Data touched:_ Instance

### Reminders
_Trigger:_ reminder:trigger

1. Send reminder messages for pending instances
2. Include daily/weekly/monthly summaries

_Data touched:_ Reminder, Instance

### Export
_Trigger:_ /export

1. Generate CSV file
2. Send file via Telegram

_Data touched:_ Goal, Instance

### Delete Account
_Trigger:_ /delete_account

1. Confirm deletion
2. Delete user data

_Data touched:_ User, Goal, Instance

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account information
  - fields: Telegram account identifier, timezone, language preference, notification preferences
- **Goal** _(retention: persistent)_ — User-defined goal with metadata
  - fields: title, description, scope, visibility, created_at
- **Schedule** _(retention: persistent)_ — Goal scheduling information
  - fields: single deadline date/time, date range (start_date, end_date), recurrence rule (daily, weekly, monthly)
- **Instance** _(retention: persistent)_ — Concrete occurrence of a Goal on a calendar date
  - fields: status, completion timestamp, note
- **Habit** _(retention: persistent)_ — Recurring goal treated as a habit
  - fields: streak, success ratio
- **Reminder** _(retention: persistent)_ — Notification settings for a Goal or Instance
  - fields: time of day, enabled/disabled

## Integrations

- **Telegram** (required) — Bot API messaging
- **Telegram Notifications** (required) — Push notifications for reminders and summaries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure default reminder times
- Enable/disable CSV export feature
- Set data retention policies
- Manage user account deletion

## Notifications

- Reminder notifications for pending instances
- Daily summary at configurable time
- Weekly summary on Monday morning
- Monthly summary on first day of month

## Permissions & privacy

- Access to user's Telegram account identifier
- Storage of user preferences (timezone, language, notification settings)
- Storage of goal data and completion history

## Edge cases

- User changes timezone after creating goals
- Conflicting schedule rules for recurring goals
- Handling of leap years and calendar edge cases
- Large number of goals causing performance issues

## Required tests

- End-to-end test of goal creation and tracking flow
- Test of reminder and summary notifications at different times
- Test of postponement and rescheduling functionality
- Test of CSV export and data persistence

## Assumptions

- Users will interact exclusively via Telegram messages and buttons
- Timezone will be set during onboarding and used for all scheduling
- Default reminder settings will be sufficient for most users
- Recurrence rules will cover common user needs
