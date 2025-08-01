import { Button } from 'vocs/components'

# `client`

bhvr uses Vite + React as its default client side template, as React has become one of the industry defaults with a vast amount of ecosystem support. With that said you can absolutely replace it with your client of choice by [following the guide](). We highly recommend using [Vite](https://vite.dev) as your bundler since it too has lots of ecosystem suppport and is very lightweight.

## Basics

Just in case you haven't used Vite + React much here are some basics you may want to keep in mind.

### Client Side Only

Unlike Next.js, Vite + React is client side only. This means any kind of environment variable used here will be publicly accessible, which is why we have a [`server`](/packages/server) package to keep those secure. This can take some getting used to when building an application and how you might fetch data. A classic way to achieve an "on-load" API call to your server is through a `useEffect`.

```tsx App.tsx
import { useState, useEffect } from "react";

function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Define an async function inside useEffect
    async function fetchUser() {
      try {
        setLoading(true);
        // Make the API call to your server
        const response = await fetch(`${import.meta.env.SERVER_URL}`);

        // Handle non-200 responses
        if (!response.ok) {
          throw new Error(`Error: ${response.status}`);
        }

        const data = await response.json();
        setUser(data);
        setError(null);
      } catch (err) {
        setError(err.message);
        setUser(null);
      } finally {
        setLoading(false);
      }
    }

    // Call the function
    fetchUser();

    // If needed, you can return a cleanup function
    return () => {
      // Any cleanup code (if needed)
    };
  }, []); // Empty dependency array means this runs once on mount

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user data found</div>;

  return (
    <div>
      <h2>User Profile</h2>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

### Environment Variables

Despite environment variables being public in this setup, they still come in handy for things like working with your local server URL vs your deployed instance. It's best practice to keep your variables in a `.env.local` file with the following format, taking special note that they need to start with `VITE_`.

```
// [!code word:VITE]
VITE_MY_VAR=value
```

To use them inside your app be sure to use this Vite formatting

```typescript
const variable = import.meta.env.VITE_MY_VAR;
```

### Check the Config

Vite's config file `vite.config.ts` is worth exploring as it can provide some extra options and plugins.

## Styles

When creating a new bhvr project you can use the CLI to specify the CSS template you want

```bash [terminal]
bun create bhvr@latest --template default # Classic CSS
bun create bhvr@latest --template tailwind # Tailwind installed and setup
bun create bhvr@latest --template shadcn # Tailwind + Shadcn/ui component setup
```

## Routing

There are serveral ways to handle routing in your client app, but few come close to [React Router](http://reactrouter.com). Setting it up is quite simple and intuitive.

::::steps

### Install `react-router`

Make sure you're inside the `client` directory and then install `react-router`

```bash [terminal]
bun add react-router
```

### Setup Router

You can do this inside `main.tsx` or `App.tsx`, I prefer the latter. All you have to do is import the `BrowserRouter`, `Routes`, then declare your routes with the components they go to inside.

```tsx App.tsx
import { BrowserRouter, Routes, Route } from "react-router";
import Home from "./components/Home";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### Use Dynamic Routes

If you want to have a dynamic route with a path param you can set it up in your `Routes` like so

```tsx
import { BrowserRouter, Routes, Route } from "react-router";
import Home from "./components/Home";
import Post from "./components/Post";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/post/:slug" element={<Post />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

Then inside the component you can access those params with `useParams`

```typescript
import { useParams } from "react-router";

function Post() {
  const { slug } = useParams();

  return (
    <>
      <h1>Post {slug}</h1>
    </>
  );
}

export default Post;
```

::::

## Deployment

<Button href="/deployment/client/orbiter">Deployments Section</Button>
