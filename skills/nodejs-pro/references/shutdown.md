# Startup, readiness and graceful shutdown

1. Map resource ownership and existing hooks/signals before editing. Framework hooks and custom process handlers can otherwise race or clean up twice.
2. Fail readiness and stop admitting new requests/jobs; initiate HTTP server closure without blocking the start of worker draining. Keep shared dependencies alive for in-flight work.
3. Drain/cancel active work according to its contract, then close owned queues, subscriptions, Redis and database pools in dependency order. Independent cleanup may run together only if ownership permits it.
4. Bound the overall drain with the deployment termination budget; a request socket timeout is not a shutdown deadline. On timeout, report incomplete work and use an explicit force-close policy with failure status.
5. Make repeated signal handling idempotent (share one shutdown promise/state). Await cleanup and handle rejections; async EventEmitter listeners are not automatically awaited by the process.

- Do not call process.exit(0) while work/logs are still pending. process.exitCode does not close live handles; use it together with real cleanup. Forced exit may be justified by the explicit deadline policy.
- Node server.close behavior for idle HTTP connections varies by version. Inspect minimum runtime before adding closeIdleConnections/closeAllConnections. Upgraded WebSocket connections require their own cleanup; do not assume the HTTP close callback covers them.
- Readiness indicates ability to serve; liveness indicates the process is alive. A dependency outage should not automatically trigger restart loops via liveness. Graceful shutdown alone does not guarantee zero downtime.
- Test in a child process with an ephemeral port: begin a request, trigger shutdown, verify completion/rejection policy, send another signal, assert finite termination and check exit status. Include a stuck-work deadline scenario when implementing that policy.

Sources: [Node server.close](https://nodejs.org/api/http.html#serverclosecallback), [process signals](https://nodejs.org/api/process.html#signal-events), [Hono Node](https://hono.dev/docs/getting-started/nodejs#graceful-shutdown).
