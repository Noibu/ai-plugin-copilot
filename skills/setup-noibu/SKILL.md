---
name: setup-noibu
description: "Connect to Noibu and confirm your site is sending data. Interactive onboarding guide for Noibu customers in Cowork — checks the Noibu MCP connection, then verifies the site is actively sending session data, and tells the user what to do next either way."
---
<!-- overlay: add — derived-from: none -->

You're running the Noibu onboarding flow: confirm the Noibu connection, then confirm the site is actually sending data.

Run this on every invocation — don't assume prior state. Always check fresh so the status you show the user is accurate right now.
 
---

**Call metadata:** on every `noibu_*` call this skill drives, pass `skill: "setup-noibu"`, `skillVersion: "cowork:fbf39d7"`, and a short `skillStep` (see the `querying-noibu-data` skill).

## Connect Noibu

Call `noibu_get_current_user` to check if Noibu is connected:

```
noibu_get_current_user()
```

**If it succeeds:** Greet the user warmly and confirm Noibu is active. For example:

> "Great news — Noibu is already connected! That means I can help you dig into your checkout error data, analyse session impact, and surface revenue leaks. Let's make sure your site is actually sending data."

Then move on to confirming data is flowing, below.

**If it fails or errors** (not yet authenticated): Explain what connecting Noibu unlocks in plain, customer-friendly terms — e.g., the ability to ask natural-language questions about checkout errors, session replays, and revenue impact. Keep it to 2-3 sentences. Then tell the user, in plain text, that they'll need to authenticate:

> "You'll just need to connect Noibu first — head to **Customize → Plugins**, find Noibu, and click Connect. If Connect isn't there yet, refresh the page and it should show up. Once that's done, start a **new chat** with me and I'll pick up right where we left off."

Cowork loads a connector's tools once per chat, at the start — connecting mid-conversation doesn't add them to this chat, so re-checking here will keep failing even after the user connects. Don't ask the user to reply "done" and retry in this same chat; a fresh chat is the only thing that picks up a newly-connected tool. End the turn after the instruction above.
 
---

## Confirm data is flowing

Resolve the domain: call `noibu_list_domains` (skip if you already have a UUID from earlier in this chat). Exactly one domain → use it. More than one → ask in plain text which site to check. Keep the `companyId` from that result alongside the UUID — this check needs both.

Then call `noibu_check_data_connection`:

```
noibu_check_data_connection({
  domainId: "<resolved-domain-uuid>",
  companyId: <companyId from the same noibu_list_domains row>,
  rationale: "Onboarding — confirming the customer's site is sending session data to Noibu",
  skill: "setup-noibu",
  skillStep: "check-data-connection"
})
```

**If `isReceivingData` is true:** Confirm it plainly and point the user at what to ask next. For example:

> "And your site is actively sending data — sessions are coming in right now. Try asking me something like 'What's my conversion rate this week?', or ask for a store health check and I'll walk your last 24 hours."

**If `isReceivingData` is false:** Say plainly that the connection works but the site isn't sending data, and what to check:

> "Noibu is connected, but I'm not seeing recent sessions from [domain] yet. That usually means the tracking snippet isn't installed on the live site, or it was added recently and hasn't caught up. Check the install with your dev team, then ask me again."

Keep this step short — it's a connectivity check, not a diagnosis. Don't chase the root cause with other tools.
 
---

## Tone Guidelines

- This skill is for Noibu's customers, not internal users. Write as if you're a friendly product expert from Noibu helping someone get set up for the first time.
- Be encouraging and specific, not generic. Reference their actual domain name.
- Keep each step moving — don't over-explain. One clear action at a time.
- Avoid jargon like "OAuth", "MCP", "registry" in messages to the user. Say "connect" and "sign in" instead.
