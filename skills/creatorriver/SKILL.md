---
name: creatorriver
description: Find YouTube, TikTok and Instagram creators (influencers) for a brand or campaign, reveal their email, send outreach and track replies with Creator River. Use when the user wants to find creators or influencers, build a creator shortlist, contact creators, or check on influencer outreach.
---

# Creator River

Creator River is a creator database with outreach built in. Its ten operations are available as MCP tools and as `creatorriver` CLI commands with the same inputs. MCP tools describe their own inputs; `--help` on any CLI command shows its flags and cost.

| MCP tool              | CLI command                      |
| --------------------- | -------------------------------- |
| `get_account`         | `creatorriver account`           |
| `search_creators`     | `creatorriver creators search`   |
| `get_creator`         | `creatorriver creators get`      |
| `unlock_creator`      | `creatorriver creators unlock`   |
| `list_campaigns`      | `creatorriver campaigns list`    |
| `create_campaign`     | `creatorriver campaigns create`  |
| `get_campaign_status` | `creatorriver campaigns status`  |
| `reveal_contact`      | `creatorriver contacts reveal`   |
| `send_email`          | `creatorriver emails send`       |
| `get_conversation`    | `creatorriver conversations get` |

## Connect

- **MCP tools present** (e.g. `get_account`): use them.
- **Otherwise use the CLI**: `npx -y creatorriver account`. Each command prints one JSON line; read `data` on success, or `error` (on stderr, non-zero exit code) on failure. List flags are comma-separated (`--platforms youtube,tiktok`).
- **`unauthorized`**: run `npx -y creatorriver auth login`. It prints a `verificationUrl` and a `userCode`: ask the user to open the link, check the code and approve, then run `npx -y creatorriver auth login --wait`. If the user would rather paste a key from https://www.creatorriver.com/settings/apikeys, save it with `printf '%s' "<key>" | npx -y creatorriver auth login --with-key`. To add the MCP server to Claude Code instead: `claude mcp add --transport http creatorriver https://www.creatorriver.com/api/mcp`, then sign in from `/mcp`.

## Workflow

1. **Check the account.** `get_account` shows the balance, prices and today's limits. Do it before anything that costs credits.
2. **Pick a campaign.** Reuse a fitting one from `list_campaigns`, or `create_campaign` with a brief: the product, the goal and the creators wanted. Every account has a default "General Outreach" campaign.
3. **Search.** Pass `campaignId`. Write `query` as a description of the creators, not keywords: niche, what their content is like, who watches it ("US moms reviewing budget kitchen gadgets in short videos"). Put only hard constraints in filters: platform, country, language, follower and view ranges. If results are off, rephrase the query rather than piling on filters; each search counts against the daily limit.
4. **Shortlist.** Show the best matches as a table: platform, country, followers, average views, engagement, topics, estimated price. Names stay masked until a creator is unlocked; that is expected. Use `get_creator` (free) for a closer look at the promising ones.
5. **Spend with consent.** `reveal_contact` returns the email and adds the creator to the campaign. The first time for each creator it costs the unlock plus the reveal (see `get_account` for prices); repeats are free. Before revealing more than a couple of creators, tell the user the total and get a go-ahead, unless they already gave you a budget.
6. **Write and send.** Draft each email as [Writing outreach](#writing-outreach) describes. Emails go to real people and cannot be unsent, so show the drafts to the user and send only what they approve. `send_email` is safe to retry: the same message to the same creator is never sent twice.
7. **Follow up.** `get_conversation` shows replies and what the thread is waiting on; `get_campaign_status` shows every creator in the campaign by stage, with the next action.

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
| `rate_limited`              | A daily limit is used up until `details.resetAt`; tell the user rather than retrying.                           |
| `unauthorized`              | See Connect.                                                                                                    |
| `login_pending`             | The user has not approved the login yet; ask them to, then run `auth login --wait` again.                       |
| `internal`, `network_error` | Retry once; if it fails again, tell the user.                                                                   |
