import { Button } from "vocs/components";

# `server`

This package lets us build an API or connect databases and other tools that require a server environment. bhvr uses [Hono](https://hono.dev) to power the API as it's simple, easy to use, and has a lot of ecosystem plugins and adoption. It's similar to express but lighter and more modern.

:::tip
Check out the [official Hono documentation](https://hono.dev) for a full reference
:::

## Basics

You can declare routes in your API like so

```typescript
import { Hono } from "hono";

const app = new Hono();

app.get("/", (c) => {
  return c.text("Hello Hono!");
});

export default app;
```

Hono also makes it easy to add in path parameters

```typescript
app.get("/user/:name", async (c) => {
  const name = c.req.param("name");
  // ...
});
```

or multiple parameters

```typescript
app.get("/posts/:id/comment/:comment_id", async (c) => {
  const { id, comment_id } = c.req.param();
  // ...
});
```

The `(c)` in Hono is the `Context` which has loads of primary features of your API.

**Access a Request**

```typescript
app.get("/hello", (c) => {
  const userAgent = c.req.header("User-Agent");
  // ...
});
```

**Return JSON or HTML**

```typescript
app.get("/api", (c) => {
  return c.json({ message: "Hello!" });
});
```

```typescript
app.get("/", (c) => {
  return c.html("<h1>Hello! Hono!</h1>");
});
```

**Access an ENV**

```typescript
// Type definition to make type inference
type Bindings = {
  MY_KV: KVNamespace;
};

const app = new Hono<{ Bindings: Bindings }>();

// Environment object for Cloudflare Workers
app.get("/", async (c) => {
  c.env.MY_KV.get("my-key");
  // ...
});
```

## RPC

One of the unique built in features of Hono is it's RPC. With this enabled you can create a Hono client in your frontend and get automatic type safety without needing to import or export types from `shared`. This is not enabled by default to help keep an unbiased template starter, but when creating a new bhvr project it is easy to enable.

```bash [terminal]
bun create bhvr@latest --rpc
```

This will setup your API with the following code, and the key being the use of `const routes` and exporting the `AppType`

```typescript src/index.ts
import { Hono } from "hono";
import { cors } from "hono/cors";
import type { ApiResponse } from "shared/dist";

const app = new Hono();

app.use(cors());

const routes = app
  .get("/", (c) => {
    //[!code focus]
    return c.text("Hello Hono!");
  })

  .get("/hello", async (c) => {
    const data: ApiResponse = {
      message: "Hello BHVR!",
      success: true,
    };

    return c.json(data, { status: 200 });
  });

export type AppType = typeof routes; // [!code focus]
export default app;
```

In your `client` code Hono is installed as a dependency, and the `hc` client is imported and initialized. The `AppType` is also used so we can build types with it.

```typescript src/App.tsx
import { useState } from "react";
import beaver from "./assets/beaver.svg";
import type { AppType } from "server"; // [!code focus]
import { hc } from "hono/client"; // [!code focus]
import "./App.css";

const SERVER_URL = import.meta.env.VITE_SERVER_URL || "http://localhost:3000";

const client = hc<AppType>(SERVER_URL); // [!code focus]

type ResponseType = Awaited<ReturnType<typeof client.hello.$get>>; // [!code focus]

function App() {
  const [data, setData] = useState<
    Awaited<ReturnType<ResponseType["json"]>> | undefined
  >();

  async function sendRequest() {
    try {
      const res = await client.hello.$get();
      if (!res.ok) {
        console.log("Error fetching data");
        return;
      }
      const data = await res.json();
      setData(data);
    } catch (error) {
      console.log(error);
    }
  }

  return <>{/* JSX markup...*/}</>;
}

export default App;
```

## DB Connections

There are lots of options out there which can be simple as installing some packages and setting up API keys like [Supabase](https://supabase.com). You can also install ORM clients to handle raw connections like [Drizzle](https://orm.drizzle.team) or [Prisma](https://prisma.io/orm). Since Hono works great with Cloudflare Workers I would also highly recommend checking out [D1](https://developers.cloudflare.com/d1) as it can be easily accessed through the Hono context.

```typescript
import { Hono } from "hono";

// This ensures c.env.DB is correctly typed
type Bindings = {
  DB: D1Database;
};

const app = new Hono<{ Bindings: Bindings }>();

// Accessing D1 is via the c.env.YOUR_BINDING property
app.get("/query/users/:id", async (c) => {
  const userId = c.req.param("id");
  try {
    let { results } = await c.env.DB.prepare(
      "SELECT * FROM users WHERE user_id = ?"
    )
      .bind(userId)
      .all();
    return c.json(results);
  } catch (e) {
    return c.json({ err: e.message }, 500);
  }
});

// Export our Hono app: Hono automatically exports a
// Workers 'fetch' handler for you
export default app;
```

## Cloudflare Workers

One of the best ways to use Hono is with Cloudflare Workers. They're cheap, pretty easy to use, and can link to other Cloudflare services like KVs or Databases. However there are some differences with how you use Hono if you take the Worker route, so we'll point out some tips here.

### Environment Variables

Cloudlfare has both public and private variables, and can only be accessed through the Hono Context. This means if you have a function in another file or folder using something like `process.env.MY_SECRET` it's not going to work. Instead you need to put your environemnt names as Bindings to make everything typesafe.

```typescript
type Bindings = {
  MY_BUCKET: R2Bucket;
  USERNAME: string;
  PASSWORD: string;
};

const app = new Hono<{ Bindings: Bindings }>();

// Access to environment values
app.put("/upload/:key", async (c, next) => {
  const key = c.req.param("key");
  await c.env.MY_BUCKET.put(key, c.req.body);
  return c.text(`Put ${key} successfully!`);
});
```

To set these variables, public ones can be put inside the `wrangler.jsonc` or `wrangler.toml` file.

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "server",
  "main": "src/index.ts",
  "compatibility_date": "2025-05-07",
  // "compatibility_flags": [
  //   "nodejs_compat"
  // ],
  "vars": {
    "MY_VAR": "my-variable"
  },
  "kv_namespaces": [
    {
      "binding": "MY_KV_NAMESPACE",
      "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
  ]
  // "r2_buckets": [
  //   {
  //     "binding": "MY_BUCKET",
  //     "bucket_name": "my-bucket"
  //   }
  // ],
  // "d1_databases": [
  //   {
  //     "binding": "MY_DB",
  //     "database_name": "my-database",
  //     "database_id": ""
  //   }
  // ],
  // "ai": {
  //   "binding": "AI"
  // },
  // "observability": {
  //   "enabled": true,
  //   "head_sampling_rate": 1
  // }
}
```

Secret variables can be used in a test env by storing them in a `.dev.vars` file

```
MY_SECRET = "SOME_SECRET"
```

To use secret variables for deployment you can use the Cloudflare dashboard or use the Wrangler CLI

```bash [terminal]
bunx wrangler secret put MY_SECRET

# Prompt: Enter your secret to have it encrypted on Cloudflare
```

:::tip
Read the [Hono documentation](https://hono.dev/docs/getting-started/cloudflare-workers) on Workers for more info
:::

## Deployment

<Button href="/deployment/server/cloudflare-workers">Deployments Section</Button>
