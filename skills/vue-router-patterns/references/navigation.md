# Navigation and component reuse

- Parameter changes can reuse the same component instance, so mounted hooks need not run again. Watch the exact parameter or use the appropriate route-update hook; a new component key is not a default fix.
- Route params/query are parsed URL values, not validated domain types. Normalize optional/array/null cases and validate numeric/enum meaning at the boundary. Do not assert a scalar just to silence typing.
- Use named routes/params and query according to actual route definitions. Supplying a path with params does not generally perform named-route interpolation. Preserve required params and URL encoding.
- Choose return/throw/redirect guard behavior supported by the installed version. If a legacy next callback is used, ensure each logical path settles once. Avoid redirect loops and unvalidated external return URLs.
- beforeEnter is not a universal observer for every params/query change on the same route; select the hook/watch matching the actual transition.
- Await router.push/replace when completion matters and inspect navigation failures; a resolved promise does not always mean the desired route became active. Rejections/errors are distinct from expected cancellation/duplication; use official failure APIs.
- Guards can access stores only with a correctly initialized/provided Pinia. Do not add module-scope store reads that run before app installation.
- Preserve public deep links, aliases, base paths and not-found behavior. Router 5 typed/file-based routing features require checking the actual plugin/configuration rather than assuming they are enabled.

Sources: [Dynamic params](https://router.vuejs.org/guide/essentials/dynamic-matching.html), [guards](https://router.vuejs.org/guide/advanced/navigation-guards.html), [navigation failures](https://router.vuejs.org/guide/advanced/navigation-failures.html).
