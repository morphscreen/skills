# React with Vite, and a small server for the key

A Vite app has no server routes, so the key lives in a small Express server beside it; in development Vite proxies `/api` to it. The same shape as https://github.com/morphscreen/starter-react, which is tested end to end.

```sh
npm install @morphscreen/web express
```

## The server

```js
// server.mjs: run with `node --env-file=.env server.mjs` (Node 20+)
import express from "express";
import { Readable } from "node:stream";

const API = process.env.MORPHSCREEN_API ?? "https://api.morphscreen.com/v1";
const MODEL = process.env.MORPHSCREEN_MODEL ?? "deepseek/deepseek-v4.1-flash";

const app = express();
app.use(express.json({ limit: "1mb" }));

app.post("/api/morphscreen", async (req, res) => {
  const stop = new AbortController();
  res.on("close", () => stop.abort());
  const upstream = await fetch(`${API}/ui`, {
    method: "POST",
    headers: { "Content-Type": "application/json", "X-MorphScreen-Key": process.env.MORPHSCREEN_KEY },
    body: JSON.stringify({ ...req.body, forwardedProps: { ...req.body.forwardedProps, model: MODEL } }),
    signal: stop.signal,
  });
  res.status(upstream.status).type(upstream.headers.get("content-type") ?? "text/event-stream");
  Readable.fromWeb(upstream.body).pipe(res);
});

app.listen(process.env.PORT ?? 3000);
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: { proxy: { "/api": "http://localhost:3000" } },
});
```

In production, serve the built page from the same server (`app.use(express.static("dist"))`) or put both behind one host.

## The chat

Vite serves the core straight from the package with `?url`:

```tsx
import { useEffect, useRef, useState } from "react";
import { connectAgent, loadCore, type Conversation } from "@morphscreen/web";
import coreUrl from "@morphscreen/web/morphscreen-agent.wasm?url";
import "@morphscreen/web/morphscreen.css";

export function Chat() {
  const element = useRef<HTMLDivElement>(null);
  const [chat, setChat] = useState<Conversation | null>(null);

  useEffect(() => {
    let live: Conversation | undefined;
    let gone = false;
    loadCore(coreUrl).then((core) => {
      if (gone) return;
      live = connectAgent(element.current!, { core, url: "/api/morphscreen" });
      setChat(live);
    });
    return () => {
      gone = true;
      live?.destroy();
    };
  }, []);

  // A form that calls chat.send(text), as in the Next.js recipe.
  return <div ref={element} />;
}
```

The key goes in `.env` as `MORPHSCREEN_KEY`, read by the server only. Never `VITE_MORPHSCREEN_KEY`: Vite bundles `VITE_` variables into the page.
