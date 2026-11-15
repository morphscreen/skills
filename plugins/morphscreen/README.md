# MorphScreen for coding agents

This plugin adds one skill, `morphscreen`, that teaches Claude to add [MorphScreen](https://morphscreen.com) to a web app. MorphScreen lets a model answer with UI: cards, charts, tables, forms, quizzes and the app's own components, checked line by line against the components the app allows and drawn while the answer streams.

With the skill, Claude knows when MorphScreen fits a request and when it doesn't, sets it up the right way (a server route that keeps the MorphScreen key out of the browser, the web SDK on the page, the stylesheet, the WebAssembly core), adds the app's own components, and fixes the mistakes people hit most, each with its cause.

## What it runs, sends and fetches

The plugin has no hooks, no MCP servers and no scripts of its own. Its skill is instructions and recipes. When you ask Claude to set MorphScreen up, the skill tells it to:

- run `npx morphscreen@0.1.0 init` in your app, which downloads the `morphscreen` command-line tool from npm, installs `@morphscreen/web` from npm with your package manager, and writes a few files into your app (a server route, a chat page, a script that copies the core, a `MORPHSCREEN_KEY=` line in `.env.local`). It never overwrites a file.
- run `npx morphscreen@0.1.0 doctor`, which reads your app's files and environment and, unless you pass `--offline`, asks registry.npmjs.org for the SDK's latest version, checks that api.morphscreen.com is up, and sends your `MORPHSCREEN_KEY` to api.morphscreen.com once, in a one-line render, to confirm the key is valid. It sends nothing else.
- read MorphScreen's docs at morphscreen.com (llms.txt and the pages as Markdown) when it needs a detail.

The source of the command-line tool is at [github.com/morphscreen/morphscreen/tree/main/sdk/cli](https://github.com/morphscreen/morphscreen/tree/main/sdk/cli). Using MorphScreen's API needs a MorphScreen key, free at [app.morphscreen.com](https://app.morphscreen.com) with no card; the skill asks you for it and never makes one up.

## Use it

Ask for what you want, for example "add a chat to this app whose answers come back as charts and tables", or "make the model's answers in this page use our ProductCard component".

Apache-2.0. Issues: [github.com/morphscreen/morphscreen/issues](https://github.com/morphscreen/morphscreen/issues).
