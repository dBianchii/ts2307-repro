## Repro: default import compiles even though source has no default export

This repo shows `tsc` succeeding when a package default-imports from another package that only has named exports.

### What happens

- In `@ts2307-repro/b`, we do a default import from `@ts2307-repro/a/src/Utils` and call `Test.test()`.
- In `@ts2307-repro/a/src/Utils.ts`, there is only a named export `test`; there is no default export.
- Running `pnpm -F @ts2307-repro/b tsc` exits 0 (no error), which is unexpected.

### Exact steps to reproduce

1. Install deps

```sh
pnpm i
```

2. Build just package `@ts2307-repro/b`

```sh
pnpm -F @ts2307-repro/b typecheck
```

Observed: no diagnostics. Expected: an error like “Module '@ts2307-repro/a/src/Utils' has no default export.”

### Minimal code involved

`packages/a/src/Utils.ts`

```ts
export const test = () => {
  return "test";
};
```

`packages/b/src/index.ts`

```ts
import Test from "@ts2307-repro/a/src/Utils";

const main = () => {
  console.log(Test.test());
};

main();
```

### Configuration notes

All packages extend a shared config at `tools/tsconfig/tsconfig.json` with (abbreviated):

```json
{
  "compilerOptions": {
    "composite": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": false,
    "isolatedModules": true,
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "verbatimModuleSyntax": true
  }
}
```

TypeScript version: `5.8.3` (from root `package.json`). Package manager: `pnpm`.

### Expected vs actual

- Expected: Type error for a default import when the target module only has named exports.
- Actual: No error; build succeeds.
