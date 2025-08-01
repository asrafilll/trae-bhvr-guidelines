# Get started

bhvr is a framework used to build apps that are not tied down to a single provider, and each piece can be deployed in multiple places.

## Quickstart

::::steps

### Create a New Project

To start using bhvr make sure you have [Bun](https://bun.sh) installed first, then run the following command.

```bash [terminal]
bun create bhvr@latest
```

The stack is composed of the following:

```
.
├── client/               # React frontend
├── server/               # Hono backend
├── shared/               # Shared TypeScript definitions
│   └── src/types/        # Type definitions used by both client and server
└── package.json          # Root package.json with workspaces
```

### Start Up Dev Server

Once you have created your project you can `cd` into it and run the dev server

```bash [terminal]
bun run dev
```

This will spin up dev servers for the `server`, `client`, and `shared` packages

Try updating the API endpoints in the `server` package

```typescript [server/src/index.ts]
import { Hono } from "hono";
import { cors } from "hono/cors";
import type { ApiResponse } from "shared/dist";

const app = new Hono();

app.use(cors());

app.get("/", (c) => {
  return c.text("Hello Hono!");
});

app.get("/hello", async (c) => {
  const data: ApiResponse = {
    message: "Hello BHVR!",
    success: true,
  };

  return c.json(data, { status: 200 });
});

export default app;
```

Also try editing the React app in `client`

```tsx [client/src/App.tsx]
import { useState } from "react";
import beaver from "./assets/beaver.svg";
import { ApiResponse } from "shared";
import "./App.css";

const SERVER_URL = import.meta.env.VITE_SERVER_URL || "http://localhost:3000";

function App() {
  const [data, setData] = useState<ApiResponse | undefined>();

  async function sendRequest() {
    try {
      const req = await fetch(`${SERVER_URL}/hello`);
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

### Build Project

Once you have your project ready to go you can use the build command to build the `client` and `shared` packages

```bash [terminal]
bun run build
```

From there you can select multiple [deployment options](/deployment/client/orbiter)

::::

## Manual Setup

There are few other ways you can get started with bhvr outside of the CLI

### Clone or Use GitHub Template

If you visit the [bhvr repo](https://github.com/stevedylandev/bhvr) you can click the "Use This Template" button in the top right.

![screenshot](https://cdn.stevedylan.dev/ipfs/bafybeicf2phwxwkqwl7uhr4awdrd5h7a37mqd7eay3czbpaldozjze3noa)

Alternatively you can clone the template

```bash [terminal]
git clone https://github.com/stevedylandev/bhvr
```

### Create from Scratch

To recreate bhvr from scratch you can do the following

::::steps

#### Install Bun

```bash [terminal]
curl -fsSL https://bun.sh/install | bash
```

#### Setup Project

```bash [terminal]
mkdir bhvr
cd bhvr
bun init -y
rm index.ts
mkdir shared
```

#### Update Root Files

Update `tsconfig.json` with the following settings

```json
{
  "compilerOptions": {
    // Environment setup & latest features
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "target": "ESNext",
    "module": "ESNext",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,

    // Path resolution
    "baseUrl": "./",
    "paths": {
      "@server/*": ["./server/src/*"],
      "@client/*": ["./client/src/*"],
      "@shared/*": ["./shared/src/*"]
    },

    // Module resolution
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "verbatimModuleSyntax": true,

    // Strictness and best practices
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "experimentalDecorators": true,

    // Output control
    "skipLibCheck": true,

    // Optional strict flags (disabled by default)
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noPropertyAccessFromIndexSignature": false
  }
}
```

Update the `package.json` with the following contents

```json
{
  "name": "bhvr",
  "version": "0.3.0",
  "description": "A monorepo template built with Bun, Hono, Vite, and React",
  "license": "MIT",
  "workspaces": ["./server", "./client", "./shared"],
  "scripts": {
    "dev:client": "cd client && bun run dev",
    "dev:server": "cd server && bun run dev",
    "dev:shared": "cd shared && bun run dev",
    "dev": "concurrently \"bun run dev:shared\" \"bun run dev:server\" \"bun run dev:client\"",
    "build:client": "cd client && bun run build",
    "build:shared": "cd shared && bun run build",
    "build:server": "cd server && bun run build",
    "build": "bun run build:shared && bun run build:client",
    "postinstall": "bun run build:shared && bun run build:server"
  },
  "keywords": ["bun", "hono", "react", "vite", "monorepo"],
  "devDependencies": {
    "bun-types": "latest",
    "concurrently": "^9.1.2"
  },
  "peerDependencies": {
    "typescript": "^5.0.0"
  }
}
```

#### Setup Client

Setup Vite with your preferences

```bash [terminal]
bun create vite@latest client
cd client
```

Update the `package.json` file

```json
{
  "name": "client",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "shared": "workspace:*", // [!code focus]
    "server": "workspace:*" // [!code focus]
  },
  "devDependencies": {
    "@eslint/js": "^9.22.0",
    "@types/node": "^22.15.2", // [!code focus]
    "@types/react": "^19.0.10",
    "@types/react-dom": "^19.0.4",
    "@vitejs/plugin-react": "^4.3.4",
    "eslint": "^9.22.0",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-react-refresh": "^0.4.19",
    "globals": "^16.0.0",
    "typescript": "~5.7.2",
    "typescript-eslint": "^8.26.1",
    "vite": "^6.3.1"
  }
}
```

Update the `tsconfig.json` file

```json
{
  "extends": "../tsconfig.json", // [!code focus]
  "files": [],
  "references": [
    { "path": "./tsconfig.node.json" },
    { "path": "./tsconfig.app.json" }
  ]
}
```

Update the `vite.config.ts` file

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path"; // [!code focus]

export default defineConfig({
  plugins: [react()],
  resolve: {
    // [!code focus]
    alias: {
      // [!code focus]
      "@client": path.resolve(__dirname, "./src"), // [!code focus]
      "@server": path.resolve(__dirname, "../server/src"), // [!code focus]
      "@shared": path.resolve(__dirname, "../shared/src"), // [!code focus]
    },
  },
});
```

#### Setup Server

Create `server` repo with Hono

```bash [terminal]
bun create hono@latest server
cd server
```

Update the `tsconfig.json` file

```json
{
  "extends": "../tsconfig.json", // [!code focus]
  "compilerOptions": {
    // Environment settings
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "ESNext",
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx",

    // Types
    "types": ["bun-types"],

    // Output settings
    "declaration": true, // [!code focus]
    "outDir": "dist", // [!code focus]
    "noEmit": false, // [!code focus]
    "emitDecoratorMetadata": true, // [!code focus]

    // Module resolution
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": false
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Update the `package.json` file

```json
{
  "name": "server",
  "version": "0.0.1",
  "main": "dist/index.js", // [!code focus]
  "types": "dist/index.d.ts", // [!code focus]
  "scripts": {
    "build": "tsc",
    "dev": "bun run --hot src/index.ts"
  },
  "dependencies": {
    "hono": "^4.7.7",
    "shared": "workspace:*" // [!code focus]
  },
  "devDependencies": {
    "@types/bun": "latest"
  }
}
```

#### Setup Shared

Initialize the directory

```bash [terminal]
cd shared
bun init -y
mkdir src
mv index.ts src/index.ts
```

Update the `tsconfig.json` file

```json
{
  "extends": "../tsconfig.json", // [!code focus]
  "compilerOptions": {
    // Environment setup
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "ESNext",

    // Output configuration
    "declaration": true, // [!code focus]
    "outDir": "./dist", // [!code focus]
    "noEmit": false, // [!code focus]

    // Type checking
    "strict": true,
    "skipLibCheck": true,

    // Additional checks
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  },
  "include": ["src/**/*"], // [!code focus]
  "exclude": ["node_modules", "**/*.test.ts", "dist"] // [!code focus]
}
```

Update the `package.json` file

```json
{
  "name": "shared",
  "version": "0.0.1",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch"
  },
  "devDependencies": {
    "typescript": "^5.2.2"
  }
}
```

#### Start Up bhvr 🦫

```bash [terminal]
bun install
bun dev
```

::::
