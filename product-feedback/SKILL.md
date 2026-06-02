---
name: product-feedback
description: |
  Draft and submit product feedback posts to Slack #product-feedback (C06VC8GC02V)
  in Archive's canonical format. Accepts Grain share links, Superhuman / email
  links, email screenshots, Slack thread links, raw call notes, or just the
  CSM's own idea. Extracts the feedback signal, fills the template, previews
  the draft inline, then posts to #product-feedback on user approval.

  Two modes auto-detected from input:
  - Client-originated (Mode A): full template with customer name, email,
    company, optional role/title, what they experienced + quote, why it
    matters, current workaround, proposed solution.
  - CSM-originated (Mode B): shorter, no client header — gap, why it
    matters, proposed solution.

  MANDATORY TRIGGERS: submit product feedback, draft product feedback,
  product feedback post, post to product feedback, share product feedback,
  send to product-feedback, feedback channel post.
---

# Product Feedback Skill

Replaces the multi-path classifier automation that was producing overstated, metadata-heavy posts. This skill stays tight, matches the channel's voice, and submits directly.

## When to use

User wants to publish customer feedback (or their own idea) to `#product-feedback` and provides any combination of:
- Grain share URL (call recording / notes)
- Superhuman link or thread (email)
- Email screenshot (image attachment)
- Slack thread URL
- Free-text from a call, conversation, or DM
- The CSM's own observation / idea

## Channel target

- Channel name: `#product-feedback`
- Channel ID: `C06VC8GC02V`
- Use `mcp__claude_ai_Slack__slack_send_message_draft` for a draft, or `slack_send_message` only on explicit "send it" / "post it" approval.

## Mode detection

Auto-detect based on inputs:

- *Mode A — Client-originated:* there is an identifiable external customer (Grain external participants, an email from a customer domain, a screenshot of a customer email, or explicit "feedback from {Name} at {Company}" in input).
- *Mode B — CSM-originated:* the only source is internal — CSM's own observation, internal Slack thread, internal pattern across multiple clients with no specific customer attribution. Channel pattern shows team members (Anne, Ethan M, Paul, Rumi, Andres, Emanuel) posting short observations directly without a customer header.

When in doubt, ask the CSM. Do not invent a customer.

## Required fields

### Mode A
- *Name* — required
- *Email* — *MANDATORY*. Do not post without one. Confirm with the CSM if not in the input.
- *Company* — required
- *Role / Title* — best-effort. See "Title lookup" below.

### Mode B
- No customer fields required. CSM is the source.

## Title lookup

When the email is provided in Mode A, attempt to find the contact's title:

1. Search HubSpot contacts by email using `mcp__claude_ai_HubSpot__search_crm_objects` with `objectType: contacts` and a filter on `email`.
2. If found and `jobtitle` is populated, use it.
3. If contact not found, or `jobtitle` empty, *leave the title out of the post*. Do not guess.
4. Optionally, surface this to the CSM in the draft preview: "Title not found in HubSpot — drop one in or leave empty?"

Never fabricate a title.

## Template — Mode A (client-originated)

```
Hi team! Sharing some product feedback from {Name}{, {Role}}{ at {Company}} ({email}).
{Optional 1-sentence customer context — only if non-obvious}

*1. {Bold title — the gap in one phrase}*
*What they experienced:* {2–3 sentences. Include verbatim quote in _italics_ when available.}
*Why it matters:* {2–3 sentences. Business impact, who else is affected.}
*Current workaround:* {What they do today. "None." if none.}
*Proposed solution:* {Logical solution from convo context, CSM input, or client suggestion. If multiple sources, label which (e.g., "Client's ask:" / "CSM thinking:").}

*2. {Next item, same structure}*
```

Field rules:
- `{Role}` and `{Company}` rendered with commas only if present
- Skip "Optional 1-sentence customer context" line if input doesn't support it
- Skip *Proposed solution* line if convo gives no signal AND CSM hasn't provided one — don't invent
- Each numbered item is one distinct gap. If input contains multiple gaps, number them sequentially.

## Template — Mode B (CSM-originated)

```
{Direct one-line setup with the gap or use case}

*Gap / opportunity:* {what's missing or what could be unlocked}
*Why it matters:* {business impact, use case context, who else hits this}
*Proposed solution:* {CSM's idea or logical next step}
```

Field rules:
- Keep it short. Channel pattern shows 3–6 sentences total for Mode B posts.
- No customer header.
- No cc.

## Hard constraints

- *Do not auto-tag people.* No `@Dorothy`, `@Denis`, `@Paul`, etc. unless the CSM explicitly says to add a cc. The aa-bug-reports skill auto-tags by domain; this skill does not.
- *Do not include a metadata block.* No "Customer Tier", "Segment", "Date", "Contact" header. The customer info goes in the first sentence only.
- *Do not include classifier reasoning.* No "Path: feature (0.95 confidence)", no "Voice checks", no "Considered" lines.
- *Do not include an "Internal notes" block at the bottom.* Posts are read by product/eng — they don't need workflow commentary.
- *Do not include KB / help-article links* unless the CSM explicitly attaches them as context for the product team.
- *Do not include "Proposed solution"* when the input genuinely has no signal — leave it out cleanly.

## Workflow

1. *Read all inputs.* Fetch Grain transcripts/notes if a share URL is provided. Fetch Superhuman threads if a link is provided. Read email screenshots directly. Pull Slack threads if a URL is provided.
2. *Detect mode.* Mode A vs Mode B.
3. *Extract feedback items.* Identify each distinct product gap. Do not pad. If the input describes one gap, post one item.
4. *Fill required fields* for Mode A. Look up title in HubSpot if email is available. Confirm with CSM if email is missing.
5. *Apply the template.* Use *italics* for verbatim customer quotes. Use *bold* for section labels.
6. *Show draft inline.* Render the full post the way it will appear in Slack.
7. *Wait for explicit approval.* "Looks good", "post it", "send it", "go" — then post via `slack_send_message`. Anything ambiguous → keep as a draft via `slack_send_message_draft`.
8. *Confirm the post.* Return the Slack message link.

## Inputs the skill accepts

| Input | Handling |
|---|---|
| Grain share URL | Extract meeting_id from URL, call `mcp__claude_ai_Grain__fetch_meeting_notes` (preferred) or `fetch_meeting_transcript` if notes are sparse |
| Superhuman link | Resolve via `mcp__claude_ai_Superhuman__query_email_and_calendar` to find the thread, then `get_thread` for full content |
| Email screenshot | Read directly as an image, transcribe relevant content + extract sender details |
| Slack thread URL | Parse channel ID + message ts from URL, call `mcp__claude_ai_Slack__slack_read_thread` |
| Raw text | Use as-is |
| Just an idea from CSM | Mode B, no fetching needed |

## Voice rules (channel native style)

- "Sharing some product feedback from..." opener (Mode A)
- Direct, professional, conversational. Not corporate-stiff.
- Customer quotes in italics, framed by "_As they put it:_ ..." or just "_..._"
- Why-it-matters always connects to business impact (other affected customers, retention risk, scalability, competitive gap)
- Workaround field is honest — "None." is a complete answer
- Proposed solution should be modest — describe the shape of the fix, not a full spec

## Examples of good posts (from channel)

- *Mode A:* Andres on Snif (Madeline) re: YoY date comparison gap
- *Mode A:* Andres on SkinSpirit (Jackie) re: combined engagement sort
- *Mode A:* Emanuel on DoorDash (Megan) re: Competitive Insights API export
- *Mode B:* Paul on "we need to be able to filter magic fields in reports this cycle"
- *Mode B:* Rumi on Gymshark unlock — 175 of 275 paid posts not in compliance via magic fields
- *Mode B:* Ethan M on self-serve cancellation UX

## Error states

- *Email missing in Mode A* → stop, ask CSM
- *Two distinct customers in one input* → ask CSM if it should be one post or two
- *Input is too vague to extract a gap* → ask CSM to clarify the feedback signal before drafting
- *Grain link returns no notes* → fall back to transcript, summarize relevant section
- *HubSpot title lookup fails* → leave title blank, optionally surface to CSM

## What this skill replaces

The previous multi-path classifier automation that:
- Added a Customer Tier / Segment / Date / Contact metadata block
- Output classification reasoning ("Path: feature (0.95 confidence)")
- Padded posts with KB citations, Voice checks, Considered paths
- Force-appended `<@Nate> <@Paul>` tags
- Generated an "Internal notes" block at the bottom

This skill does none of that. It produces what Andres, Valentina, Paul, Rumi, Anne, Ethan M, and Emanuel actually post.
