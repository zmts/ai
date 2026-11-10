# Types, precision and time

- Choose integer/bigint/UUID from domain range and existing contracts; bigint is not mandatory future-proofing. Check driver serialization: int8/numeric may arrive as strings; converting to JavaScript number can lose precision.
- Use exact numeric or established scaled-integer units for exact decimal quantities. Select numeric precision/scale from required ranges and rounding rules; do not assume every price has two decimals. Avoid integer division when a fractional result is required.
- text versus varchar(n) is primarily a domain constraint decision, not a universal speed rule. Preserve meaningful length limits and encoding/collation assumptions.
- timestamptz represents an instant and displays it in the session timezone; it does not retain the original zone name. timestamp without time zone is appropriate for deliberate local wall-clock semantics. Store a separate IANA zone when the domain needs it.
- For a local calendar day, derive start and end in the intended zone and compare instants with a half-open range. A DST day can be 23 or 25 hours; adding a fixed 24 hours is not always the next local midnight.
- Prefer predicates on raw indexed time values when possible; wrapping every row in date()/casts can prevent an ordinary index condition. Derive bounds with explicit types/timezone.
- now()/CURRENT_TIMESTAMP is transaction-start time. Use statement_timestamp()/clock_timestamp() only when their distinct timing semantics are intended.
- Preserve date_trunc buckets, inclusive/exclusive endpoints and timestamp precision. Do not mix seconds/milliseconds or assume the database session timezone matches the user's timezone.

Source: [PostgreSQL types](https://www.postgresql.org/docs/current/datatype.html), [date/time](https://www.postgresql.org/docs/current/datatype-datetime.html), [date functions](https://www.postgresql.org/docs/current/functions-datetime.html).
