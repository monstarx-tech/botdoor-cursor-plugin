---
name: post-to-social
description: Post to social media (Instagram, TikTok, Facebook, LinkedIn, X, YouTube, Threads, Bluesky, Pinterest, Reddit) through Botdoor with human approval. Use when the user wants to publish, schedule, or draft a social post, add a first comment or thread, check whether a post went live, or set up Botdoor for the first time.
---

# Post to social media with Botdoor

Botdoor (https://botdoor.co) lets an agent create and schedule social posts while a human connects the accounts and approves what goes out. Full docs: https://docs.botdoor.co. Everything below uses the `botdoor` MCP server at `https://botdoor.co/api/mcp`.

## Tools

`signup` (only without a key), `list_accounts`, `get_connect_url`, `new_claim_link`, `upload_media`, `create_post`, `get_post`, `list_posts`, `cancel_post`, `reschedule_post`, `reject_post`, `retry_first_comment`, `approve_post` (people only, so it will fail for you).

## 1. First run: no key yet

If the only tool you see is `signup`, there is no API key yet.

1. Ask the user for **their** email address (the human who will own the workspace). Never invent one or use your own.
2. Call `signup` with `email` and, if known, `workspaceName` and `agentName`.
3. A `201` returns `apiKey` once. Tell the user to store it as the `BOTDOOR_API_KEY` environment variable (or in the MCP config as `Authorization: Bearer <key>`) and reload MCP servers. Never print the key in files you commit.
4. Botdoor emails them a **claim link**. They open it, set a password and choose whether your key keeps access. Until they claim, nothing can publish and no account can be connected (`403 claim_required`).
5. A `202` with `apiKey: null` means that email already has a workspace: the user signs in at https://botdoor.co/login and creates a key under **API keys**.
6. Lost email? Call `new_claim_link` (2 a day).

Details: https://docs.botdoor.co/bots/signup-and-claim/

## 2. Connect an account

1. `list_accounts`. If it is empty, call `get_connect_url` with `platform` (instagram, tiktok, facebook, linkedin, youtube, threads, pinterest, reddit, twitter).
2. Send the returned `url` to the user. Only they can log in to the network. Bluesky is connected in the app (https://botdoor.co/accounts) with an app password.
3. Call `list_accounts` again and use each account's `id` in `accountIds`.

## 3. Create a post

1. Media: `upload_media` (base64, up to 10 MB here; larger files go through `POST https://botdoor.co/api/v1/media`). Instagram, TikTok, YouTube and Pinterest need media. Several `mediaIds` make a carousel.
2. Always validate first: `create_post` with `dryRun: true`. Fix anything in the errors or `warnings`.
3. Then `create_post` for real with an `idempotencyKey` so a retry never posts twice.
4. Links go in `firstComment` (posted from the same account about 10 seconds after the post is live), not in the caption.
5. Threads (`thread` or `firstReply`) work on X, Threads and Bluesky. Polls on X and LinkedIn, without media.

## 4. Scheduling

- `scheduledFor` is ISO 8601 **with an offset**, for example `2026-10-12T09:00:00+11:00`. Ask the user's timezone if unsure. Past times and times over a year ahead are refused.
- Change it with `reschedule_post`; stop it with `cancel_post` (works while waiting or scheduled).

## 5. Approval

- Keys from signup **ask first**: every post stops at `needs_approval` until a signed-in person approves it on https://botdoor.co/posts or from the email. Tell the user it is waiting; do not claim it was posted.
- You cannot approve (`approval_requires_human`), not even with another key. Do not try to work around this.
- If the scheduled time passes while waiting, nothing is posted; the person approves with **Publish now** or a new time.

## 6. Check the result

Call `get_post` until `settled` is true. Each entry in `targets` has `status`, the live `url`, or an `error`. Report the URLs. `partial` means some targets failed: read each `error`. A failed first comment can be retried with `retry_first_comment`.

## Safety rules

- Only post what the user asked for, to the accounts they named. Confirm the text, accounts and time before a non-dry-run `create_post` unless the user already approved them.
- Never paste API keys into posts, comments or repository files.
- Errors have a stable `code` and a `hint`; follow the hint. Full list: https://docs.botdoor.co/help/errors/
