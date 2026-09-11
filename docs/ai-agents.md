# Use with an AI agent

Drop our Claude Agent Skill into your agent, or feed it Markdown.

The Booking Pro API is built to be driven by LLMs and AI agents.

## Claude Agent Skill

We publish a ready-made **Agent Skill** (authored to the open
[agentskills.io](https://agentskills.io) standard, so it also works in Cursor, Codex,
and GitHub Copilot). It teaches an agent how to authenticate, the conventions, common
recipes, and the full endpoint catalog, all kept in sync with this reference.

- **Get the skill from this repo**: the [`skill/`](../skill/) folder (`SKILL.md` plus
  `reference/endpoints.md`). Copy it into your agent's skills directory.
- **Hosted copy**: `https://developers.bookingpro.ai/skill/booking-pro-api/SKILL.md`,
  and a zipped bundle at `https://developers.bookingpro.ai/skill/booking-pro-api.zip`.

Give the agent a key (as an environment variable, never inline) and it can check
availability, book, and manage contacts on your behalf.

## Every page is Markdown

- Click **Copy Markdown** at the top of any page on the docs site.
- Or append `.md` to any page's URL to fetch its Markdown.
- `https://developers.bookingpro.ai/llms.txt` is a compact index of the whole site.
- `https://developers.bookingpro.ai/llms-full.txt` is every page concatenated, ready to
  paste into a context window.

## A note on keys

An AI agent calling the API needs a real key with real scopes. Mint a dedicated key for
the agent under **Settings -> Developers**, grant only the scopes it needs, and revoke
it if anything looks wrong.
