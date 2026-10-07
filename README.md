# Skills

The agent skills I use every day to build and grow small web products.

Each skill is a folder with a `SKILL.md`: plain instructions your agent loads when a task matches. They work with Claude Code, Codex and any agent that reads skills. Read them, change them, make them yours.

## Installation

Two ways in. **The Claude Code plugin** installs the whole set as a managed bundle that updates with this repo. **The skills installer** copies editable skill files into your project or home folder. Pick one: installing both gives you every skill twice.

<details>
<summary><strong>Claude Code</strong></summary>

```bash
claude plugin marketplace add znerol74/skills
claude plugin install znerol74-skills@znerol74
```

Or, from inside a session:

```
/plugin marketplace add znerol74/skills
/plugin install znerol74-skills@znerol74
```

To get updates automatically, turn on auto-update for the `znerol74` marketplace under `/plugin` → Marketplaces. It is off by default for marketplaces outside Anthropic's.

</details>

<details>
<summary><strong>Codex, Cursor and other agents</strong></summary>

```bash
npx skills@latest add znerol74/skills
```

Pick the skills you want and the agents to install them on. The files land in your repo or home folder as ordinary files you own. Pull later changes with `npx skills update`.

</details>

<details>
<summary><strong>By hand</strong></summary>

Copy a skill folder into `~/.claude/skills/` (for all projects) or `.claude/skills/` (for one project):

```bash
git clone https://github.com/znerol74/skills.git
cp -R skills/skills/growth/checkout-optimizer ~/.claude/skills/
```

</details>

## The skills

### Growth

| Skill | What it does |
|---|---|
| [`checkout-optimizer`](./skills/growth/checkout-optimizer/SKILL.md) | Audits the path from upgrade offer to paid customer in a web product that sells with Stripe: when the paywall shows up, how it is built, the Stripe Checkout session and the funnel from "offer shown" to "paid". Ranks changes by impact, puts business and legal choices in front of you, then builds what you approve. |
| [`seo-scale`](./skills/growth/seo-scale/SKILL.md) | Keyword research and SEO potential with real data (Ubersuggest MCP): volume, difficulty, intent, competitor structure, and an honest call on which keywords and which page architecture are worth it. Written in German. |

`seo-scale` needs an Ubersuggest MCP connection in your agent.

## Repo layout

```
skills/<category>/<skill>/SKILL.md   the skills
.claude-plugin/plugin.json           the Claude Code plugin (lists every skill)
.claude-plugin/marketplace.json      the marketplace that serves the plugin
```

Adding a skill: put its folder under `skills/<category>/`, add its path to `skills` in `.claude-plugin/plugin.json`, add a row to the table above, and bump the plugin `version`.

## License

MIT
