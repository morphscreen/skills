# The app's own components

Full page: https://morphscreen.com/docs/format/custom-components.md

## 1. Describe each one in a pack

```json
{
  "name": "shop",
  "version": "1.0.0",
  "components": {
    "ProductCard": {
      "about": "a product with its price",
      "attrs": {
        "name": { "type": "string", "required": true },
        "price": { "type": "string", "required": true },
        "stock": ["in", "low", "out"]
      },
      "content": "none"
    }
  }
}
```

- Attribute types: `string`, `number`, `boolean`, `url`, `list` (values split by ` | `), or a list of allowed values whose first is the default.
- `content`: `none`, `text`, `table`, `list`, `images` or `any`.
- A `text` component may take `"actions": true`: the model can then put `Button`s inside it (`ask` or `href`).
- One attribute, one meaning: `stock="low"`, never a style string.
- A name can't be defined twice: an app's pack can't replace a built-in component by accident (except when `packs` is left out entirely, where the app's `Product` or `Map` replaces the pack's).

## 2. Send it to the API with every call

From the server route, beside the model:

```ts
run.forwardedProps = { ...run.forwardedProps, model: MODEL, components: [shop] };
```

For `/v1/chat/completions`: `"morphscreen": { "components": [shop] }` in the body.

## 3. Draw it on the page

The page gets the same pack and a drawer per component. `connectAgent`, `createAnswer` and the React `<Answer>` all take `custom` and `components`:

```ts
connectAgent(el, {
  core,
  url: "/api/morphscreen",
  custom: [shop],
  components: {
    ProductCard: (el, { attrs }) => {
      el.textContent = `${attrs.name}: ${attrs.price}`;
    },
  },
});
```

```tsx
import { Answer, type CustomProps } from "@morphscreen/web/react";

function ProductCard({ attrs }: CustomProps) {
  return (
    <div className="product">
      <strong>{attrs.name}</strong> {attrs.price}
    </div>
  );
}

<Answer core={core} text={text} custom={[shop]} components={{ ProductCard }} />;
```

A drawer is called as soon as the component opens (`streaming: true`, attributes only), then again when it's complete, into the same element emptied, with `html` (its words, then its buttons) and `text`. It may return a cleanup. Its element is `<div class="ms-custom">`, which MorphScreen's stylesheet leaves to the app's own styles.

A component the page doesn't draw still shows, as its attributes and content: nothing is dropped silently.
