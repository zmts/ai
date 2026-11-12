# Modules and compatibility

- Establish the nearest package.json type, extensions, exports conditions and actual entrypoint. .mjs is ESM; .cjs is CommonJS; .js interpretation depends on environment. Bundlers and Node may resolve differently.
- Preserve module format; do not migrate a repository to ESM to fix one import.
- Native Node ESM relative imports generally need actual emitted extensions. Inspect output instead of copying source extensions blindly.
- Use public package exports; deep imports can be blocked by exports or change outside the supported contract.
- ESM/CommonJS interoperability depends on Node version and package shape. Do not assume named CJS exports or require support for an arbitrary ESM graph. Use the established bridge or supported dynamic import.
- Trace cycles and initialization before introducing barrels. Respect safe public module boundaries.
- Changing type/module/moduleResolution/target/exports can affect other consumers; keep it separate unless required for the task.
- Native TypeScript execution is version-dependent and is not typechecking. Verify syntax and resolution in the actual execution path.
- Validate actual startup/build/test entrypoints. For libraries, check emitted files and public entrypoints used downstream.

Sources: [Node packages](https://nodejs.org/api/packages.html), [Node ESM](https://nodejs.org/api/esm.html#interoperability-with-commonjs), [Node TypeScript](https://nodejs.org/api/typescript.html). Match docs to deployment versions.
