---
layout: ../../layouts/Plugin.astro
name: "Prometheus Client"
description: "Plugin for adding prom-client to Fastify and collecting default metrics."
tags:
  - Monitoring
  - Decorator
  - Hooks
---

Decorates the Fastify instance with a Prometheus client and registry using the prom-client library. Collects default metrics and exposes a <code class="il">/metrics</code> endpoint.

## Plugin definition
```ts
import { FastifyPluginCallback } from "fastify";
import fp from "fastify-plugin";
import prom from "prom-client";

const promPlugin: FastifyPluginCallback = (instance, opts, done) => {
  // See prom-client library documentation for more details.
  const client = prom;
  const collectDefaultMetrics = client.collectDefaultMetrics;
  const appRegistry = new client.Registry();

  // Initialize default metric collection using app lifecycle hook
  instance.addHook("onReady", function (done) {
    collectDefaultMetrics({ register: appRegistry });
    done();
  });

  // Expose metrics endpoint for Prometheus to scrape
  instance.get("/metrics", async (req, reply) => {
    reply.header("Content-Type", appRegistry.contentType);
    return await appRegistry.metrics();
  });

  // Decorate instance with client and registry for creating custom metrics.
  instance.decorate("prom", client);
  instance.decorate("promRegister", appRegistry);

  done();
};

export default fp(promPlugin);
```

## Type Definition

```ts
import prom, { Registry } from "prom-client";

declare module "fastify" {
  interface FastifyInstance {
    prom: typeof prom;
    promRegister: Registry;
  }
}
```