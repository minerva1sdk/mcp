# Minerva Connect

Backtest, optimize and build stock trading strategies and signals on your own Minerva account. Sign in with your Minerva account at https://connect.minerva1.com.

## Claude Code

```bash
/plugin marketplace add minerva1sdk/mcp
```

```bash
/plugin install minerva-connect@minerva-connect
```

Starter commands: `/minerva-connect:<name>` with the names below.

## Gemini command line

```bash
gemini extensions install https://github.com/minerva1sdk/mcp
```

Then sign in with `/mcp auth minerva`. Starter commands: `/minerva:<name>` with the names below.

## Codex

```bash
codex mcp add minerva --url https://connect.minerva1.com/mcp
```

```bash
codex mcp login minerva
```

## Any other host

Add the MCP server `https://connect.minerva1.com/mcp` and sign in. Paste this block into the host's AGENTS.md or GEMINI.md:

```markdown
## Minerva
The `minerva` MCP server (Minerva Connect) backtests and optimizes stock trading strategies and signals on my Minerva account. Use it whenever I ask to build, test, backtest, optimize or study a trading strategy or signal, or ask about my Minerva runs, reports, strategies or credits. When I start without asking for something specific, call get_started first. Full guide: https://github.com/minerva1sdk/mcp/blob/main/skills/minerva-connect/SKILL.md
```

## Starter prompts

- `describe-a-strategy`: I want to describe a trading strategy in my own words and test it on Minerva.
- `start-from-a-paper-or-link`: I want Minerva to build a trading strategy from a paper, web page or post.
- `build-one-step-by-step`: I want to build a trading strategy on Minerva together, step by step.

## License

The files in this repository are under the MIT License (LICENSE). Minerva's service is under Minerva's [Terms of Service](https://minerva1.com/legal/terms).
