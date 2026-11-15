# Next.js (App Router) by hand

What `npx morphscreen init` writes, for when it can't run. The same shape as https://github.com/morphscreen/starter-nextjs, which is tested end to end.

```sh
npm install @morphscreen/web
```

## The core, copied where the browser can fetch it

Next.js doesn't serve files out of `node_modules`.

```js
// scripts/copy-morphscreen-core.mjs
import { copyFileSync, mkdirSync } from "node:fs";
import { createRequire } from "node:module";

const require = createRequire(import.meta.url);
mkdirSync("public", { recursive: true });
copyFileSync(require.resolve("@morphscreen/web/morphscreen-agent.wasm"), "public/morphscreen-agent.wasm");
```

In `package.json`, run it before `dev` and `build` (in the script itself: pnpm doesn't run `pre` scripts by default):

```json
"dev": "node scripts/copy-morphscreen-core.mjs && next dev",
"build": "node scripts/copy-morphscreen-core.mjs && next build"
```

Add `public/morphscreen-agent.wasm` to `.gitignore`.

## The route

```ts
// app/api/morphscreen/route.ts
const API = process.env.MORPHSCREEN_API ?? "https://api.morphscreen.com/v1";
const MODEL = process.env.MORPHSCREEN_MODEL ?? "deepseek/deepseek-v4.1-flash";

export async function POST(req: Request) {
  const run = await req.json();
  run.forwardedProps = { ...run.forwardedProps, model: MODEL };
  const upstream = await fetch(`${API}/ui`, {
    method: "POST",
    headers: { "Content-Type": "application/json", "X-MorphScreen-Key": process.env.MORPHSCREEN_KEY ?? "" },
    body: JSON.stringify(run),
    signal: req.signal,
  });
  return new Response(upstream.body, {
    status: upstream.status,
    headers: { "Content-Type": upstream.headers.get("content-type") ?? "text/event-stream", "X-Accel-Buffering": "no" },
  });
}
```

Return `upstream.body` as it is (no `await upstream.text()`, no `NextResponse.json`), or the reader waits for the whole answer. `signal: req.signal` stops the model when the reader leaves.

## The chat

```tsx
// app/morphscreen/chat.tsx
"use client";

import { useEffect, useRef, useState } from "react";
import { connectAgent, loadCore, type Conversation } from "@morphscreen/web";

export function MorphScreenChat() {
  const element = useRef<HTMLDivElement>(null);
  const [chat, setChat] = useState<Conversation | null>(null);
  const [busy, setBusy] = useState(false);

  useEffect(() => {
    let live: Conversation | undefined;
    let gone = false;
    loadCore("/morphscreen-agent.wasm").then((core) => {
      if (gone) return;
      live = connectAgent(element.current!, { core, url: "/api/morphscreen" });
      setChat(live);
    });
    return () => {
      gone = true;
      live?.destroy();
    };
  }, []);

  async function send(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const input = e.currentTarget.elements.namedItem("message") as HTMLInputElement;
    const text = input.value;
    input.value = "";
    setBusy(true);
    try {
      await chat?.send(text);
    } finally {
      setBusy(false);
    }
  }

  return (
    <div>
      <div ref={element} />
      <form onSubmit={send}>
        <input name="message" placeholder="Ask anything" aria-label="Your message" required />
        <button disabled={!chat || busy}>Send</button>
      </form>
    </div>
  );
}
```

```tsx
// app/morphscreen/page.tsx (or app/layout.tsx for the stylesheet)
import "@morphscreen/web/morphscreen.css";
import { MorphScreenChat } from "./chat";

export default function Page() {
  return <MorphScreenChat />;
}
```

## The key

`.env.local`: `MORPHSCREEN_KEY=ms_live_...` (48 hex digits after the prefix). Server only.
