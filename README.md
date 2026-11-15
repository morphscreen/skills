# MorphScreen skills for coding agents

A skill that teaches Claude Code, Codex and other coding agents to add [MorphScreen](https://morphscreen.com) to an app: a model's answers drawn as UI (cards, charts, tables, forms, your own components) while they stream, with the key kept on the server.

It knows when MorphScreen fits and when it doesn't, runs `npx morphscreen init` and `doctor` where it can, has a hand-written recipe for Next.js and for React with Vite where it can't, covers your own components, and lists every mistake we've seen with its fix.

## Claude Code

```sh
claude plugin marketplace add morphscreen/skills
claude plugin install morphscreen@morphscreen
```

Or inside a session: `/plugin marketplace add morphscreen/skills`, then `/plugin install morphscreen@morphscreen`. Then ask for what you want, like "add a chat to this app whose answers come back as charts and tables".

## Codex and other agents

Copy the skill's folder into the agent's skills folder. For Codex:

```sh
git clone https://github.com/morphscreen/skills /tmp/morphscreen-skills
mkdir -p ~/.codex/skills
cp -r /tmp/morphscreen-skills/plugins/morphscreen/skills/morphscreen ~/.codex/skills/
```

An agent that reads `AGENTS.md` can also be pointed at the docs instead: https://morphscreen.com/docs/coding-agents

## What's in it

- `plugins/morphscreen/skills/morphscreen/SKILL.md`: when to use it, the pieces and the one rule, the fast path, how to check the result
- `references/nextjs.md` and `references/react-vite.md`: the setup by hand, the same shape as the [Next.js](https://github.com/morphscreen/starter-nextjs) and [React](https://github.com/morphscreen/starter-react) starters, which run end to end
- `references/custom-components.md`: your own components, from the pack to the drawer
- `references/troubleshooting.md`: each symptom, its cause and its fix

Apache-2.0. Issues and ideas: https://github.com/morphscreen/morphscreen/issues
