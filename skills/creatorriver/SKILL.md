---
name: creatorriver
description: Find YouTube, TikTok and Instagram creators (influencers) for a brand or campaign, reveal their email, send outreach and track replies with Creator River. Use when the user wants to find creators or influencers, build a creator shortlist, contact creators, or check on influencer outreach.
---

# Creator River

Creator River is a database of 1.8M+ YouTube, TikTok and Instagram creators, every one with a contact email, with outreach built in. Its operations are available as MCP tools and as `creatorriver` CLI commands with the same inputs. MCP tools describe their own inputs; `--help` on any CLI command shows its flags and cost.

| MCP tool                | CLI command                          |
| ----------------------- | ------------------------------------ |
| `get_account`           | `creatorriver account`               |
| `search_creators`       | `creatorriver creators search`       |
| `browse_creators`       | `creatorriver creators browse`       |
| `find_creator`          | `creatorriver creators find`         |
| `find_similar_creators` | `creatorriver creators similar`      |
| `list_searches`         | `creatorriver searches list`         |
| `get_search`            | `creatorriver searches get`          |
| `get_search_filters`    | `creatorriver searches filters`      |
| `update_search`         | `creatorriver searches update`       |
| `export_search`         | `creatorriver searches export`       |
| `get_creator`           | `creatorriver creators get`          |
| `unlock_creator`        | `creatorriver creators unlock`       |
| `save_creator`          | `creatorriver creators save`         |
| `unsave_creator`        | `creatorriver creators unsave`       |
| `list_saved_creators`   | `creatorriver creators saved`        |
| `list_campaigns`        | `creatorriver campaigns list`        |
| `create_campaign`       | `creatorriver campaigns create`      |
| `update_campaign`       | `creatorriver campaigns update`      |
| `get_campaign_status`   | `creatorriver campaigns status`      |
| `add_creator`           | `creatorriver campaigns add`         |
| `exclude_creator`       | `creatorriver campaigns exclude`     |
| `reveal_contact`        | `creatorriver contacts reveal`       |
| `reveal_contacts`       | `creatorriver contacts reveal-batch` |
| `send_email`            | `creatorriver emails send`           |
| `get_conversation`      | `creatorriver conversations get`     |
| `record_reply`          | `creatorriver conversations record`  |
| `confirm_collaboration` | `creatorriver conversations confirm` |

## Connect

- **MCP tools present** (e.g. `get_account`): use them.
- **Otherwise use the CLI**: `npx -y creatorriver account`. Each command prints one JSON line; read `data` on success, or `error` (on stderr, non-zero exit code) on failure. List flags are comma-separated (`--platforms youtube,tiktok`). Account exclusions, batch reveals and follow-ups without a subject require CLI 0.5.0 or newer. Check `creatorriver --version` if using a locally installed copy.
- **`unauthorized`**: run `npx -y creatorriver auth login`. It prints a `verificationUrl` and a `userCode`: ask the user to open the link, check the code and approve, then run `npx -y creatorriver auth login --wait`. If the user would rather paste a key from https://www.creatorriver.com/settings/apikeys, save it with `printf '%s' "<key>" | npx -y creatorriver auth login --with-key`. To add the MCP server to Claude Code instead: `claude mcp add --transport http creatorriver https://www.creatorriver.com/api/mcp`, then sign in from `/mcp`.

## Workflow

1. **Check the account.** `get_account` shows the balance, prices and today's limits. Do it before anything that costs credits.
2. **Pick a campaign.** Reuse a fitting one from `list_campaigns`, or `create_campaign` with a brief: the product, the goal and the creators wanted. Every account has a default "General Outreach" campaign. When the brief changes, `update_campaign` keeps later searches ranked against the new one.
3. **Search.** Pass `campaignId`, describe the content and audience you want, and put hard requirements in filters. For new prospects use `excludeContacted` across campaigns. Apply `excludeExported` or `excludeUploaded` when the user also wants to skip past exports or the list they uploaded on the website. Pass handles or profile links as `exampleCreators` for lookalikes and read `warnings` for unresolved examples. Use `find_creator` for an exact handle, `find_similar_creators` for more like one result, or `browse_creators` for filters alone. New searches, similar searches and filtered lists share the allowance in `get_account`. The default unfiltered list and reopening a successful search that day use no extra allowance. `list_searches` and `get_search` reopen saved results.
4. **Shortlist.** Show the strongest matches with evidence from their profiles. Names are visible before unlocking. Read `get_creator` for a closer look. `add_creator` puts a creator in the campaign, and `save_creator` keeps an account bookmark. Either costs the first unlock. `exclude_creator` with `searchId` removes a poor fit from the campaign and that search. Exporting a search as CSV records the returned creators for `excludeExported`.
5. **Reveal selected emails.** Use `reveal_contact` for one creator or `reveal_contacts` for up to 10 per request. Both add the creator to the campaign. For a larger selection, quote the maximum for the whole selection using `get_account` and get approval unless the user already provided a sufficient budget. An initial unlock and reveal cost 0.7 credits together. Reuse across campaigns is free. Read each batch item's status, keep successes and pause further batches if credits or today's allowance run out.
6. **Write and send.** Draft each email as [Writing outreach](#writing-outreach) describes. Show the recipient, subject and text for each email and send only the drafts the user approves. The first email needs a subject. The subject and body support `{name}`, `{handle}`, `{platform}`, `{followers}` and `{country}`. If a value is unavailable, revise the draft. On a follow-up in the same campaign, omit the subject to reply in the existing thread. Creator River copies the last person who replied on the creator’s behalf. Retrying the same send reuses its result.
7. **Keep the pipeline current.** When the user tells you about a reply that came some other way (a DM, a call), `record_reply` with how it went; replies to your emails arrive on their own, so never record those. Once a creator has clearly agreed to terms, `confirm_collaboration` with what was agreed. When the user rules a creator out, `exclude_creator`. None of these messages the creator.
8. **Follow up.** `get_conversation` shows replies and who the thread is waiting on. An `auto_reply` such as an out-of-office message leaves it waiting on the creator. The account owner receives a notification for a human email reply. `get_campaign_status` shows how many creators are at each stage and the most recently active ones, with the next action. To go through all creators at one stage, pass `stage` and read `page` 1, 2, … until you have seen `count` of them.

## Writing outreach

Creators get many pitches and answer the ones written for them. Before drafting, read the creator with `get_creator`: recent video titles, topics, language and audience.

- **Subject:** short and specific, naming the brand and the collaboration, e.g. "Paid collab: Acme x your desk setup videos". No clickbait, no all caps.
- **Opening:** one concrete detail from a recent video, in a sentence. Not generic praise.
- **Who and what:** who you are and what the product does, in one sentence.
- **Offer:** the deliverable (a dedicated video, a 60–90 second integration, a Reel) and the pay (a fixed fee, free product, affiliate). If the user has not set a budget, ask for the creator's rate instead of naming a number.
- **One ask:** one easy next step, e.g. reply with a rate and availability.
- **Form:** 80 to 150 words, plain text, at most one link, no attachments, in the creator's language. Sign with the user's real name and company.
- **One creator per email:** never send the same text to many creators. Identical mail gets filtered as spam and harms the sender's reputation.
- **Follow up once,** after four or five days without a reply, in a few lines. Stop after that, and stop at once if the creator says no.

## Errors

| Code                        | What to do                                                                                                      |
| --------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `invalid_input`             | Fix the fields listed in `details.issues` and retry.                                                            |
| `not_found`                 | The id is wrong or belongs to another account; get ids again from `search_creators` or `list_campaigns`.        |
| `conflict`                  | A step is missing, e.g. `reveal_contact` before `send_email`; the message says which.                           |
| `insufficient_credits`      | Tell the user the balance and the cost (`details`); credits are bought at https://www.creatorriver.com/pricing. |
| `rate_limited`              | Give the reset time in `details.resetAt` and the purchase link if the response includes one.                    |
| `unauthorized`              | See Connect.                                                                                                    |
| `login_pending`             | The user has not approved the login yet; ask them to, then run `auth login --wait` again.                       |
| `internal`, `network_error` | Retry once; if it fails again, tell the user.                                                                   |
