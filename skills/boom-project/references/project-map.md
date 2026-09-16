# Boom Source Map

All paths below are relative to the target Boom repository, not the ECC skill
directory. Discover the workspace first; never assume a personal checkout path.
Read existing files and current manifests before relying on this snapshot.

| Work | Source to inspect |
| --- | --- |
| Root conventions and package commands | `AGENTS.md`, `package.json`, `pnpm-workspace.yaml` |
| Wallet app conventions | `apps/boom/AGENTS.md`, `apps/boom/package.json` |
| App visual direction | `apps/boom/DESIGN.md` |
| Token wiring and component styling | `.skills/design-system.md`, `packages/design-tokens/package.json`, `packages/design-tokens/src/tokens.ts` |
| External brand and marketing | `.skills/boom-brand.md` |
| Shared backend policy | `services/AGENTS.md` |
| Appwrite functions | `services/functions/AGENTS.md`, affected function manifest and entrypoint |
| Appwrite client exports and identity handling | `packages/appwrite-client/src/`, `packages/appwrite-client/package.json` |
| Hiro requests and rate limiting | `packages/hiro-client/src/`, `packages/hiro-client/package.json` |
| Fastify API and storage ownership | `services/api/boom-api/package.json`, its routes, services, and migrations |
| Contract workflow and requirements | `packages/clarity/AGENTS.md`, `packages/clarity/Clarinet.toml`, `packages/clarity/package.json` |
| Contract behavior and SDK integration | `packages/clarity/contracts/`, `packages/clarity/tests/`, `packages/clarity/src/sdk/` |

## Corrections Made During Adaptation

- Older service instructions mention `services/appwrite/` and
  `services/railway/`; the inspected checkout uses `services/functions/` and
  `services/api/`. Do not relocate services based on the old examples.
- Older contract instructions use `pnpm --dir blockchain test`. The inspected
  manifest identifies `@boom/blockchain` under `packages/clarity`, so select the
  package through the workspace instead.
- The source's Appwrite "v2.0" label and blanket query limit are not treated as
  universal SDK facts. Confirm the installed SDK, server, wrapper, and endpoint.
- Identity extraction is not authentication on an arbitrary public endpoint.
  Verify the trusted execution boundary and authorization checks.
- Appwrite sample error handlers expose raw messages; preserve the API contract
  while returning safe public errors and redacting diagnostic logs.
- Source BTC examples use floating-point multiplication. Use existing validated
  integer amount handling for financial operations.
- The token package currently exports generated `dist` assets. Its manifest and
  build scripts take precedence over older generated-file maps.

These corrections scope the imported helper; they do not edit Boom's originals.
