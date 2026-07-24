# Channel Gate Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that requires users to join a specific Telegram channel before receiving a simple text reply. The bot sends the channel invite, provides a confirmation button, verifies the join, and then delivers the configured message.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram channel owners
- users seeking gated content

## Success criteria

- User receives configured reply after successfully joining the required channel

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initiates the channel join verification process
- **I've joined** (button, actor: user, callback: verify_join) — Triggers channel membership verification

## Flows

### join_verification
_Trigger:_ start

1. Send channel invite link and 'I've joined' button
2. User clicks 'I've joined' button
3. Verify channel membership
4. If verified, send configured reply; if not, re-prompt with error

_Data touched:_ channel_id, reply_text, user_verification_attempts

### admin_controls
_Trigger:_ /admin

1. Verify admin identity
2. Display channel and reply configuration options

_Data touched:_ channel_id, reply_text

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **channel_id** _(retention: persistent)_ — Telegram ID of the required channel for membership verification
  - fields: id
- **reply_text** _(retention: persistent)_ — Configurable text message sent after successful channel verification
  - fields: text
- **user_verification_attempts** _(retention: session)_ — Temporary tracking of verification attempts per user
  - fields: user_id, attempts, last_attempt_time

## Integrations

- **Telegram** (required) — Bot API messaging and channel membership verification
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set required channel ID
- Update reply text message

## Permissions & privacy

- The bot requires access to verify if users are members of the specified channel
- User IDs are only stored temporarily for verification attempts tracking

## Edge cases

- User joins channel but clicks 'I've joined' before verification can run
- User joins multiple channels with the same name
- Channel ID is invalid or inaccessible to the bot

## Required tests

- Verify that users receive the configured reply only after successful channel verification
- Test admin commands to update channel ID and reply text

## Assumptions

- Owner will provide a valid Telegram channel ID during setup
- Owner will provide the reply text during setup
- Users will have access to Telegram and the required channel
