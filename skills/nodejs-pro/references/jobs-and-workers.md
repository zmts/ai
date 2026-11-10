# Jobs, workers and delivery semantics

- Identify producer, queue, processor, retry policy, acknowledgement/lock behavior and persisted result. Verify installed library/version and existing integration before recommending queue APIs.
- Assume duplicate delivery is possible where the queue permits retries/stalled recovery. Deduplication/job IDs alone do not prove exactly-once business side effects.
- Make retryable operations idempotent at the durable side-effect boundary. Retrying a payment/order/write needs a real key/state transition/constraint, not an in-memory processed set.
- Classify retryable dependency failures versus permanent validation/business failures. Bound attempts, backoff and concurrency; account for global rate limits across replicas.
- Avoid blocking the event loop and starving queue locks. Use existing CPU-worker facilities for expensive calculations when warranted, rather than adding worker infrastructure to every job.
- Cancellation, failure and worker shutdown are distinct states. Persist the state according to the established contract; a timeout or lost HTTP client does not establish cancellation of an already accepted job.
- Stop job intake before releasing DB/Redis dependencies. BullMQ worker.close waits for active work and has no built-in completion timeout; the application needs a bounded shutdown policy. Do not close a shared Redis connection while another owner still uses it.
- Test duplicate delivery, retry exhaustion and shutdown during active work when those semantics change. Use task-owned fake/local resources, not a real queue by default.

Sources: [BullMQ shutdown](https://docs.bullmq.io/guide/workers/graceful-shutdown), [idempotent jobs](https://docs.bullmq.io/patterns/idempotent-jobs). Check installed versions before applying current documentation.
