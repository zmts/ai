# Async ordering, cancellation and resources

- Promise.all rejects on a rejection; it does not cancel other operations or make effects atomic. Bound concurrency for large input sets.
- Promise.allSettled supports partial success; inspect both statuses and define how rejection affects the result.
- Promise.race settles on the first settlement; Promise.any on the first fulfillment. Neither cancels losers. A timer race alone only stops waiting.
- forEach does not await async callbacks. Use for...of for ordering or bounded concurrency for independent work.
- Async listeners/timer callbacks need rejection handling when the owner does not await them. Avoid async Promise executors.

## Cooperative deadlines

For fetch-compatible runtimes, pass cancellation to the operation and clear owned timers in finally. Check response.ok: HTTP error statuses do not reject fetch. Body reading belongs inside the deadline.

```javascript
async function fetchJsonWithTimeout(url, timeoutMs) {
  if (!Number.isFinite(timeoutMs) || timeoutMs <= 0 || timeoutMs > 2_147_483_647) {
    throw new RangeError('timeoutMs must be a supported positive timer duration');
  }
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);
  try {
    const response = await fetch(url, { signal: controller.signal });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } finally {
    clearTimeout(timer);
  }
}
```

This helper owns its signal. Preserve caller cancellation using supported signal composition or a listener removed during cleanup; do not replace an existing caller signal. Aborting is cooperative; verify client support. Do not promise cancellation for arbitrary promises.

- Release owned handles/locks in finally; remove listeners with the same function reference used to register them.
- Retry classified transient failures within bounded attempt/time budgets. Writes need established idempotency; do not retry every failure.
- for await...of provides ordered consumption, not parallelism. Early exit can invoke iterator return/finally; resource release depends on the iterator implementation.
- Keep framework subscription cleanup with its lifecycle owner.

Sources: [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController), [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch).
