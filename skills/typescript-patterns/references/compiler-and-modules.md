# Compiler and modules

- Read installed TypeScript and effective config including extends/project references. Use project scripts; bare tsc in the wrong directory can check a different project.
- Match module resolution to the host. NodeNext/Node16 model Node package rules; bundler resolution can accept imports native Node rejects.
- Inspect emitted extensions and public exports for native ESM. Typechecking success does not establish runtime resolution.
- Use import type only for type-only dependencies. Preserve runtime value imports required by instantiation/reflection/framework metadata.
- paths guides compiler resolution; tsc does not generally rewrite emitted aliases. Verify bundler/runtime support.
- target/lib/@types influence syntax/declarations, not runtime polyfills. A declaration for fetch does not establish deployment support.
- Native Node type stripping is version-dependent, does not typecheck, and does not implement every tsconfig transform. Verify actual runtime restrictions.
- Do not weaken strictness to hide an error or enable new flags across a repository incidentally.
- Check emitted declarations for libraries when relevant. Do not require declaration generation for ordinary apps.

Sources: [Module reference](https://www.typescriptlang.org/docs/handbook/modules/reference.html), [TSConfig](https://www.typescriptlang.org/tsconfig/), [Node TypeScript](https://nodejs.org/api/typescript.html).
