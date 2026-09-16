---
name: boom-project
description: Apply Boom's project conventions for its pnpm wallet monorepo, Vue/Quasar design system, shared Appwrite/Hiro clients, and Stacks/Clarity packages. Use only in a confirmed Boom workspace or when explicitly adapting Boom's conventions.
metadata:
  origin: ECC
---

# Boom Project Helpers

This is an on-demand adaptation of Boom's project instructions. Confirm the
workspace using its package metadata (for example `@boom/monorepo`) and source
layout before applying Boom-specific names or policies. A Vue or Appwrite
dependency alone is not evidence that a project is Boom.

## When to Activate

- Implementing or reviewing a change in the Boom wallet monorepo.
- Locating Boom's shared clients, design tokens, or contract test workflow.
- Explicitly adapting Boom's conventions to another project; carry over only
  the conventions requested by the user.

## How It Works

1. Read the current root and nearest `AGENTS.md` files and affected package
   manifests. Local instructions and actual package exports are authoritative;
   this skill is a portable starting point, not a frozen replacement.
2. Find the relevant source in [the project map](references/project-map.md).
   Read only the design, service, or contract material needed for the task.
3. Reuse existing workspace packages and patterns. Keep the change scoped and
   verify it through the affected package's real scripts.
4. Report stale guidance when it affects the task. Do not silently migrate the
   repository to fit an older path, SDK label, or sample snippet.
   Reconcile missing executable paths and commands against current manifests;
   report the mismatch and use the verified equivalent for the same task.

## Application Conventions

- Prefer functional composition, pure domain helpers, and reusable composables.
  Preserve existing architecture while making focused changes.
- Use Vue Composition API with `<script setup lang="ts">` and existing Quasar
  components. Boom's root guidance places template before script before style;
  follow more specific local instructions where present.
- Prefer interfaces and `const` objects over enums. Use PascalCase component
  names, camelCase functions, and named constants for repeated domain values.
- Follow the local formatter: single quotes, two-space indentation, no
  semicolons, and `rem` sizing are the recorded Boom preferences. Do not
  reformat unrelated files. Follow the nearest instruction for JSDoc scope.
- Read `apps/boom/DESIGN.md` for app UI and `.skills/design-system.md` for its
  implementation. Marketing brand guidance has a different scope from app UI.
  Prefer the current app-specific design specification over older root font or
  branding examples; flag any unresolved conflict with the implementation.
- Change `packages/design-tokens/src/tokens.ts` for shared token updates, then
  run the package's build script. Check current exports and generated output
  locations instead of assuming older `src/css` paths.

## Service and Wallet Boundaries

- Use `@boom/appwrite-client` and `@boom/hiro-client` in backend services where
  the local instructions require them. Inspect exports before calling a helper.
  If they cannot support the task, follow the local extension policy rather
  than bypassing the wrapper with a new SDK client or raw Hiro request.
- Distinguish permission-scoped browser reads, authenticated server writes,
  Appwrite function execution, and Fastify endpoints. Do not infer permission
  to access data merely from knowing a user ID or receiving a header.
- Respect the actual storage owner. Some services use PostgreSQL/Drizzle and
  Redis; the presence of Appwrite does not mean every operation belongs there.
- Keep wallet signing material out of logs and persistent UI state. Use the
  existing amount parser and integer base-unit representation; do not copy
  floating-point BTC conversion snippets into settlement or signing code.
- For contract work, apply `stacks-clarity` and inspect the current Clarinet
  configuration, caller model, asset ownership, and tests.

## Commands and Verification

Run commands from the confirmed Boom workspace. These package names were
verified during adaptation; re-read manifests if the layout changes.

```bash
pnpm --filter @boom/app type-check
pnpm --filter @boom/app e2e
pnpm --filter @boom/api test
pnpm --filter @boom/blockchain test
pnpm --filter @boom/design-tokens build
```

Choose checks appropriate to the changed package. The app's `test` script runs
Vitest in watch mode; use its supported one-shot option for automated checks.
Run `clarinet check` from the directory containing the relevant `Clarinet.toml`.
Inspect package scripts before execution; test/build work does not imply a
deployment, real transaction, or commit. Boom requires an explicit commit request.

## Example: Change Marketplace Settlement

Trace the app call through the API/shared client to the affected Clarity
entrypoint. Confirm amount units and asset ownership at each boundary. Add
coverage for authorization, failure, and settlement accounting in the relevant
existing test suites, then run the changed packages' checks. Update the existing
API/contract documentation when interfaces change.

## Related Skills

- `vue-quasar-patterns` — reusable Quasar UI guidance.
- `appwrite-patterns` — Appwrite identity, permissions, and function boundaries.
- `stacks-clarity` — contract and Stacks integration review.

## Provenance

Adapted from user-provided Boom project instructions and checked against its
local source at revision `5043760ebbbc96e0df478b1765a18ea43cf45805`.
See [the source map](references/project-map.md) for paths and known corrections.
