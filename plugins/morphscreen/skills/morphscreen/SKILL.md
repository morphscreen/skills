---
name: morphscreen
description: Add MorphScreen to an app so a model's answers are drawn as UI (cards, charts, tables, forms, quizzes, the app's own components) while they stream, checked line by line against the components the app allows. Use when the user wants generative UI, a chat or assistant that answers with charts or forms instead of plain text, to render LLM output as UI, to use @morphscreen/web, the MorphScreen API or `npx morphscreen`, to add their own components to model answers, or to fix a MorphScreen integration (unstyled answers, a core that won't load, 401s, a stream that arrives all at once).
---

# MorphScreen

MorphScreen lets a model answer with UI. The model writes Markdown with a few component tags; a small Rust core (WebAssembly in the browser) checks every finished line against the components the app allows; the API asks the same model to rewrite only the lines that break a rule; and the web SDK, `@morphscreen/web`, draws the answer as it arrives.

Docs, written for agents: https://morphscreen.com/llms.txt (every page also as `.md`, the whole set at https://morphscreen.com/llms-full.txt). Read the page you need instead of guessing an API.

## When it fits, and when it doesn't

Fits: a chat, assistant or agent whose answers have shape (comparisons, plans, budgets, dashboards, forms to fill, quizzes, steps), streamed into a web page; an app with its own components it wants the model to use; small or cheap models (the format is Markdown, which they already write well).

Doesn't fit: generating whole apps or arbitrary code (use a coding model); answers that are short plain text (Markdown alone is enough); a page that must run no WebAssembly. Native iOS and Android SDKs are in the repository but not released.

## The pieces, and the one rule

1. **A server route** of the app's own, which adds the MorphScreen key (`X-MorphScreen-Key`) and the model, and streams the response straight back. The key never reaches the browser: never `NEXT_PUBLIC_`, `VITE_` or any browser code.
2. **The page**: `loadCore` loads the core (`morphscreen-agent.wasm` for `connectAgent`), `connectAgent(el, { core, url })` runs the conversation against the app's route, and the stylesheet `@morphscreen/web/morphscreen.css` is imported once.
3. **The API**: `POST https://api.morphscreen.com/v1/ui` (AG-UI runs, what `connectAgent` speaks) or `/v1/chat/completions` (OpenAI's format, for an app already on it).

Keys: free at https://app.morphscreen.com (no card). The free plan runs `deepseek/deepseek-v4.1-flash` on MorphScreen's key, so no model key is needed to start. Any other OpenRouter model needs the app's own OpenRouter key as `Authorization: Bearer sk-or-...` from the server route.

## Fast path: Next.js with the App Router

```sh
npx morphscreen@0.1.0 init      # installs the SDK, writes the route, a chat page at /morphscreen, the core copy step, MORPHSCREEN_KEY= in .env.local
npx morphscreen@0.1.0 doctor    # every check with its fix; --json for you to read, --offline to skip the network
```

`init` never overwrites a file and is safe to run twice. Then the key goes in `.env.local` (ask the user for it, or tell them where to get one; never invent one), and the app's dev server shows the chat at `/morphscreen`. Move `<MorphScreenChat />` wherever the user wants it.

If `npx morphscreen` isn't available, or the app isn't Next.js with the App Router, follow the recipe for the stack:

- Next.js by hand: [references/nextjs.md](references/nextjs.md)
- React with Vite (and Express for the route): [references/react-vite.md](references/react-vite.md)
- Vue, Svelte, Angular, plain HTML: https://morphscreen.com/docs/integrations.md lists a page for each

## The app's own components

Describe each in a pack (JSON), send the pack to the API with every call, give the page the same pack and a function or React component that draws it. Recipe: [references/custom-components.md](references/custom-components.md).

## Verify

1. `npx morphscreen@0.1.0 doctor --json`: `ok: true`.
2. The app builds (`next build`, `vite build`).
3. In the browser: ask "compare three phones under $500" and watch for a heading, a table or cards drawing while it streams, and no console errors.

## When something's wrong

Read [references/troubleshooting.md](references/troubleshooting.md): each symptom with its cause and fix (unstyled answers, "this core has no AG-UI client", 401s, `model_needs_your_key`, a stream that arrives all at once, two chats after a reload, a content security policy that blocks WebAssembly).
