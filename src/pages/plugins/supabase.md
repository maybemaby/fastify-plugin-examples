---
layout: ../../layouts/Plugin.astro
name: "Supabase Client"
description: "Add a Supabase admin client to the Fastify instance."
tags:
  - Decorator
  - Supabase
---

Adds a Supabase client instance to the Fastify instance.

```ts
// Plugin definition
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

## Type definition
```ts
import type { SupabaseClient } from "@supabase/supabase-js";

declare module "fastify" {
  interface FastifyInstance {
    supabase: SupabaseClient;
  }
}
```