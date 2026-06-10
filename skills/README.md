# Skills

A collection of Claude Code skills for autonomous workflows, project planning, and development tooling.

## Install

Using [skills](https://github.com/vercel-labs/skills):

```bash
# Install all skills globally
npx skills add glittercowboy/taches-cc-resources

# Install specific skills (e.g. create-prompt and run-prompt)
npx skills add glittercowboy/taches-cc-resources --skill create-prompt --skill run-prompt -g

# List available skills without installing
npx skills add glittercowboy/taches-cc-resources --list
```

Or manually copy to `~/.claude/skills/`.

## If You Create a Fork

Replace `glittercowboy` with your GitHub username. For example, for `behagoras`:

```bash
npx skills add behagoras/taches-cc-resources --skill create-prompt --skill run-prompt
```

To install from a local clone:

```bash
npx skills add ./path/to/taches-cc-resources
```

## Available Skills

| Skill | Description |
|---|---|
| [`create-plans`](./create-plans/) | Hierarchical project planning for solo agentic development |
| [`create-agent-skills`](./create-agent-skills/) | Create, write, and refine Claude Code skills |
| [`create-meta-prompts`](./create-meta-prompts/) | Build Claude-to-Claude pipelines with research/plan/implement stages |
| [`create-slash-commands`](./create-slash-commands/) | Create custom Claude Code slash commands |
| [`create-subagents`](./create-subagents/) | Create and configure Claude Code subagents |
| [`create-hooks`](./create-hooks/) | Create event-driven automation with Claude Code hooks |
| [`create-mcp-servers`](./create-mcp-servers/) | Build MCP servers (TypeScript or Python) |
| [`create-prompt`](./create-prompt/) | Generate optimized XML-structured prompts for Claude Code |
| [`run-prompt`](./run-prompt/) | Execute saved prompts in isolated sub-agent contexts |
| [`debug-like-expert`](./debug-like-expert/) | Systematic debugging with evidence gathering and hypothesis testing |
| [`setup-ralph`](./setup-ralph/) | Set up Geoffrey Huntley's Ralph Wiggum autonomous coding loop |
| [`the-pirate-bay`](./the-pirate-bay/) | Search torrents via apibay.org API |
