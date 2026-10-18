# Creator River for AI agents

[Creator River](https://www.creatorriver.com) finds YouTube, TikTok and Instagram creators for a brand, reveals their email, sends outreach and tracks replies. This repository holds the Creator River skill and Claude plugin, so your agent can run that workflow for you: describe the creators you want, review a shortlist, approve the emails, and follow the replies.

## Install

**Claude Code**

```
/plugin marketplace add creatorriver/skills
/plugin install creatorriver@creatorriver
```

The plugin adds the skill and connects the Creator River MCP server. The first time you use it, sign in from `/mcp` and approve access in the browser.

**Codex, Cursor, OpenCode and other agents**

```
npx skills add creatorriver/skills
```

Without the MCP server, the skill uses the [`creatorriver`](https://www.npmjs.com/package/creatorriver) CLI (`npx -y creatorriver`), which asks you to approve it in the browser the first time.

**claude.ai**: add `https://www.creatorriver.com/api/mcp` as a custom connector.

## What it does

- Searches the creator database from a plain description: niche, content, audience, plus filters such as platform, country, language, followers and views.
- Shows creator profiles with audience, recent content, engagement and an estimated price.
- Reveals a creator's email, sends plain-text outreach from your Creator River account, and shows the replies and what each conversation is waiting on.

## Data and costs

- The plugin runs no local code. The skill and the MCP server talk only to Creator River at `https://www.creatorriver.com` (`/api/mcp`, `/api/v1`).
- Signing in creates an API key on your Creator River account. You can see and revoke it at https://www.creatorriver.com/settings/apikeys.
- Searching and reading are free within daily limits. Unlocking a creator, revealing an email and sending beyond the daily free emails cost credits. The skill tells the agent to check prices first, ask before spending, and send only the emails you approve.
- [Privacy policy](https://www.creatorriver.com/privacy-policy) · [Terms of service](https://www.creatorriver.com/terms-of-service)

## Support

support@creatorriver.com

## License

[MIT](LICENSE)
