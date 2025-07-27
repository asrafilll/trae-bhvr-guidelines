Of course. Here is the information formatted in Markdown.

# The `shared` Package

The `shared` package is primarily used for types that you may want to pass between the server and the client, but it can also be used for functions or libraries.

---

## Project Structure

When you create a project with `bun create bhvr@latest`, the structure of the `shared` package is as follows:

```
shared/
├── package.json
├── src/
│   ├── index.ts
│   └── types/
│       └── index.ts
└── tsconfig.json
```

---

## Barrel Export Pattern

The package uses a barrel export pattern. The contents of `src/index.ts` are:

```typescript
export * from "./types";
```

---

## Building the Package

By running either **`bun run build`** or **`bun run dev`**, the types will be compiled and exported from the `dist` build folder and can be used in the client or server.
