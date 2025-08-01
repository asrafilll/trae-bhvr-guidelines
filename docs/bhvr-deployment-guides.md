# Single Origin Deployment

import { Button } from "vocs/components";

Serve both your frontend and API from the same process, same port, and same origin—ideal for fullstack apps where simplicity matters.

This guide walks through configuring your bhvr project for **single origin** deployment, where your React frontend and Hono API run from the same Bun process.

Perfect for:

- VPS deployments
- Raspberry Pis
- Home servers
- Docker containers
- Projects where you want one URL to rule them all

## Prerequisites

This guide assumes you have a bhvr project set up. If not, start here:

```bash
bun create bhvr@latest my-app
cd my-app
```

<Button href='/getting-started'>Getting Started with bhvr</Button>

## What Is Single Origin?

Instead of running your client and server separately (the default bhvr setup), single origin serves everything from one process:

**Default bhvr setup:**

- Client runs on port 5173 (Vite dev server)
- Server runs on port 3000 (Hono API)
- Requires CORS for communication

**Single origin setup:**

- Everything runs on port 3000
- Hono serves both API routes and static React files
- No CORS needed

## Configuration

### 1. Update Your Hono Server

Modify `server/src/index.ts` to serve static files alongside your API:

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { serveStatic } from "hono/bun"; // [!code ++]
import type { ApiResponse } from "shared/dist";

const app = new Hono();

// CORS is optional for single origin deployment
// Keep for development flexibility, remove for production if desired
app.use(cors());

// Your existing API routes - keep the /api prefix for clarity
app.get("/api", (c) => {
  return c.text("Hello Hono!");
});

app.get("/api/hello", async (c) => {
  const data: ApiResponse = {
    message: "Hello BHVR!",
    success: true,
  };
  return c.json(data, { status: 200 });
});

// Add more API routes here with /api prefix
// app.get('/api/users', ...)
// app.post('/api/data', ...)

// Serve static files for everything else
app.use("*", serveStatic({ root: "./static" })); // [!code ++]

app.get("*", async (c, next) => {
  // [!code ++]
  return serveStatic({ root: "./static", path: "index.html" })(c, next); // [!code ++]
}); // [!code ++]

const port = parseInt(process.env.PORT || "3000");

export default {
  port,
  fetch: app.fetch,
};

console.log(`🦫 bhvr server running on port ${port}`);
```

### 2. Update Your React Client

Modify `client/src/App.tsx` to use relative API paths:

```typescript
import { useState } from "react";
import beaver from "./assets/beaver.svg";
import { ApiResponse } from "shared";
import "./App.css";

function App() {
  const [data, setData] = useState<ApiResponse | undefined>();

  async function sendRequest() {
    try {
      // Use relative path - works in both dev and production
      const req = await fetch("/api/hello"); // [!code hl]
      const res: ApiResponse = await req.json();
      setData(res);
    } catch (error) {
      console.log(error);
    }
  }

  return (
    <>
      <div>
        <a href="https://github.com/stevedylandev/bhvr" target="_blank">
          <img src={beaver} className="logo" alt="beaver logo" />
        </a>
      </div>
      <h1>bhvr</h1>
      <h2>Bun + Hono + Vite + React</h2>
      <p>A typesafe fullstack monorepo</p>
      <div className="card">
        <button onClick={sendRequest}>Call API</button>
        {data && (
          <pre className="response">
            <code>
              Message: {data.message} <br />
              Success: {data.success.toString()}
            </code>
          </pre>
        )}
      </div>
      <p className="read-the-docs">Click the beaver to learn more</p>
    </>
  );
}

export default App;
```

### 3. Configure Vite for Development

Update `client/vite.config.ts` to proxy API calls during development:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@client": path.resolve(__dirname, "./src"),
      "@server": path.resolve(__dirname, "../server/src"),
      "@shared": path.resolve(__dirname, "../shared/src"),
    },
  },
  server: {
    // [!code ++]
    proxy: {
      // [!code ++]
      "/api": {
        // [!code ++]
        target: "http://localhost:3000", // [!code ++]
        changeOrigin: true, // [!code ++]
      }, // [!code ++]
    }, // [!code ++]
  }, // [!code ++]
});
```

### 4. Add Single Origin Scripts

Add these scripts to your root `package.json` (alongside the existing bhvr scripts):

```json
{
  "scripts": {
    "dev:client": "cd client && bun run dev",
    "dev:server": "cd server && bun run dev",
    "dev:shared": "cd shared && bun run dev",
    "dev": "concurrently \"bun run dev:shared\" \"bun run dev:server\" \"bun run dev:client\"",
    "build:client": "cd client && bun run build",
    "build:shared": "cd shared && bun run build",
    "build:server": "cd server && bun run build",
    "build": "bun run build:shared && bun run build:client",
    "build:single": "bun run build && bun run copy:static && bun run build:server", // [!code ++]
    "copy:static": "rm -rf server/static && cp -r client/dist server/static", // [!code ++]
    "start:single": "cd server && bun run dist/index.js", // [!code ++]
    "postinstall": "bun run build:shared && bun run build:server"
  }
}
```

## Development vs Production

### Development (Default bhvr)

Use the standard bhvr development workflow:

```bash
bun run dev
```

This runs:

- Client on `http://localhost:5173` (Vite dev server)
- Server on `http://localhost:3000` (Hono API)
- Vite proxy forwards `/api` calls to the server

### Production (Single Origin)

Build and run from single origin:

```bash
# Build everything and prepare for single origin
bun run build:single

# Start the single origin server
bun run start:single
```

Your app now runs entirely on `http://localhost:3000`.

## Deployment

### Docker

```dockerfile
FROM oven/bun:latest
WORKDIR /app

# Copy package files
COPY package.json bun.lock ./
COPY client/package.json ./client/
COPY server/package.json ./server/
COPY shared/package.json ./shared/

# Copy source code
COPY . .

# Install dependencies
RUN bun install

# Build for single origin
RUN bun run build:single

EXPOSE 3000
CMD ["bun", "run", "start:single"]
```

### VPS / Bare Metal

```bash
# Clone your bhvr project
git clone <your-repo> my-app && cd my-app

# Install and build
bun install
bun run build:single

# Run (consider using PM2 or systemd for production)
bun run start:single
```

### Environment Variables

Configure the port and other settings:

```bash
PORT=8080 bun run start:single
```

## File Structure

After building for single origin, your bhvr project structure looks like:

```
.
├── client/
│   ├── dist/           # Built React app
│   └── src/
├── server/
│   ├── dist/
│   │   └── index.js    # Built Hono server
│   ├── static/         # Copied from client/dist
│   └── src/
├── shared/
│   ├── dist/           # Built shared types
│   └── src/
└── package.json
```

## CORS Configuration

Since single origin serves everything from the same origin, **CORS is not required** for production. However, you might want to keep it for development flexibility:

**Production (Single Origin):**

- React app and API served from same origin (e.g., `https://yourapp.com`)
- All requests are same-origin
- CORS not needed

**Development:**

- Vite proxy handles cross-origin requests automatically
- CORS still optional due to proxy, but useful for:
  - Testing API directly in browser/tools
  - Alternative development setups
  - Third-party integrations during development

**To remove CORS for production**, you can conditionally apply it:

```typescript
// Only use CORS in development
if (process.env.NODE_ENV !== "production") {
  app.use(cors());
}
```

Or remove the `app.use(cors())` line entirely if you don't need development flexibility.

## Key Benefits

- **Simplified deployment**: One process, one port, one URL
- **No CORS complexity**: Frontend and API share the same origin
- **Maintains bhvr workflow**: Still use `bun run dev` for development
- **Type safety preserved**: All bhvr type sharing continues to work
- **Resource efficient**: Perfect for small VPS, Raspberry Pi, or containers

## Troubleshooting

**API calls fail in development?**

- Ensure Vite proxy is configured in `client/vite.config.ts`
- Check that your server is running on port 3000

**404 errors on page refresh?**

- The `serveStatic` catchall should handle SPA routing automatically
- Verify client files are copied to `server/static/`

**Build fails?**

- Run `bun run build` first to ensure shared types are available
- Check that all bhvr workspaces install correctly

## Summary

Single origin deployment transforms your bhvr project from a multi-port development setup into a production-ready single process application, while preserving all the type safety and development experience that makes bhvr great.

## More Resources

<Button href='/getting-started'>Getting Started with bhvr</Button>
