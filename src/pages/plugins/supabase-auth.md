---
layout: ../../layouts/Plugin.astro
name: "Supabase Auth"
description: "Authorization plugin for checking a Supabase token from a Bearer token."
tags:
  - Authorization
  - Decorator
  - Hooks
usedPlugins:
  - fastify-sensible
---

Uses a Supabase admin client decorators and an onRequest hook to authorize requests that include supabase bearer tokens. Adds a Supabase user instance to request objects.

## Supabase Decorator Definition

```ts
// Adding Supabase client as decorator
import fp from "fastify-plugin";
import { FastifyPluginCallback } from "fastify";
import { createClient } from "@supabase/supabase-js";

const supabasePlugin: FastifyPluginCallback = (fastify, opts, done) => {
  const supabaseUrl = process.env.SUPABASE_URL;
  const supabaseKey = process.env.SUPABASE_SERVICE_KEY;
  if (!supabaseUrl || !supabaseKey) {
    throw new Error("SUPABASE_URL and SUPABASE_SERVICE_KEY must be provided");
  }
  const client = createClient(supabaseUrl, supabaseKey, {});
  fastify.decorate("supabase", client);
  done();
};

export default fp(supabasePlugin, { name: "supabase" });
```

## Authorization Plugin Definition

```ts
const authorization: FastifyPluginCallback = (fastify, opts, done) => {
  // Decorate requests with a user instance, defaults to null
  fastify.decorateRequest("user", null);

  // Add an authorize method to the fastify instance
  fastify.decorate("authorize", authorize);

  async function authorize(req: FastifyRequest, res: FastifyReply) {
    const token = extractToken(req);
    if (!token) {
      // Using fastify-sensible response helpers
      return res.unauthorized("Must be logged in to access resource");
    }
    const { allowed, err, user } = await userAllowed(token);
    if (!allowed) {
      return res.unauthorized(err ?? "Bad authorization token");
    } else if (user) {
      req.user = user;
    }
  }

  // Takes a token value from Authorization: Bearer headers returns null if not found
  function extractToken(req: FastifyRequest) {
    const header = req.headers.authorization;
    if (!header || !header.startsWith("Bearer")) {
      return null;
    }
    const [_, token] = header.split(" ");
    return token;
  }

    // Retrieves a Supabase user instance using the server client and an access token
  async function userAllowed(
    token: string
  ): Promise<{ err?: string; user?: User; allowed: boolean }> {
    const user = await fastify.supabase.auth.getUser(token);
    if (user.error) {
      return { err: user.error.message, allowed: false };
    }
    return { user: user.data.user, allowed: true };
  }
}
```

## Type Definition

```ts
import type { SupabaseClient, User } from "@supabase/supabase-js";

declare module "fastify" {
  interface FastifyInstance {
    supabase: SupabaseClient;
    authorize: (req: FastifyRequest, res: FastifyReply) => Promise<void>;
  }

  interface FastifyRequest {
    user: null | User;
  }
}
```

## Usage
```ts
const app = fastify();

// Register plugins
app.register(supabase);
app.register(authorization);

// All requests will check for a valid bearer token and reject any without or invalid tokens.
app.addHook("onRequest", app.authorize);
```