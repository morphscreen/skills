# When something's wrong

`npx morphscreen doctor` finds most of these by itself.

| Symptom | Cause | Fix |
|---|---|---|
| The answer shows as bare HTML: huge icons, no cards, unstyled tables | the stylesheet isn't loaded | `import "@morphscreen/web/morphscreen.css";` once (Next.js: `app/layout.tsx` or the chat's page) |
| `this core has no AG-UI client: load morphscreen-agent.wasm instead of morphscreen.wasm` | `connectAgent` was given the render-only core | `loadCore("/morphscreen-agent.wasm")` |
| `loadCore` fails with a 404 | the core isn't served | Next.js: copy it into `public/` before dev and build; Vite: `import coreUrl from "@morphscreen/web/morphscreen-agent.wasm?url"` |
| The core won't start: `CompileError` or a Content-Security-Policy error naming WebAssembly | the app's CSP blocks WebAssembly | add `'wasm-unsafe-eval'` to `script-src` |
| 401 `morphscreen_key_required` | the route didn't send `X-MorphScreen-Key` | read `MORPHSCREEN_KEY` on the server and send it in that header |
| 401 `morphscreen_key_invalid` | a wrong, mistyped or revoked key | a new one at https://app.morphscreen.com |
| 400 `model_needs_your_key` | a model other than the included one, without an OpenRouter key | use `deepseek/deepseek-v4.1-flash`, or send `Authorization: Bearer sk-or-...` from the route |
| 402 `included_calls_used` | the free plan's month of calls on MorphScreen's model is used | the app's own OpenRouter key, or a paid plan |
| The answer appears all at once at the end | something buffers the stream: the route read the whole body, or a proxy holds it | return `upstream.body` as it is; behind nginx send `X-Accel-Buffering: no` or turn buffering off |
| Two chats in one element after a reload in development | React's dev mode mounts effects twice and the first core finished late | the `gone` flag in the effect, and `live?.destroy()` in its cleanup |
| The key shows in the browser's network tab or bundle | it's in browser code or a public variable | never `NEXT_PUBLIC_`, `VITE_` or `PUBLIC_`; only the server route reads it |
| A component the app made shows as a list of its attributes | the page has the pack but no drawer for it | add it to `components` on the page |
| `hallucinated-component` in the verdict | the model used a component the app doesn't allow | nothing to do: the API repairs it when repair is on (the default) |

Still stuck: https://morphscreen.com/docs/faq.md, then https://github.com/morphscreen/morphscreen/issues.
