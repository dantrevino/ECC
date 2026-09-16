---
name: appwrite-patterns
description: Build and review Appwrite functions and backend data access using shared clients, explicit authentication boundaries, TablesDB, and permission-aware tests.
metadata:
  origin: ECC
---

# Appwrite Patterns

Adapt Appwrite changes to the project's installed SDK, deployment, and shared client layer.

## When to Activate

- Implementing Appwrite Functions or a backend that calls Appwrite.
- Changing row access, authentication, execution scopes, or a shared Appwrite wrapper.
- Reviewing whether a function enforces ownership when it uses server credentials.

## Workflow

1. Read the nearest project instructions, package manifest, lockfile, shared client implementation, and function configuration. Identify SDK/server versions and whether requests arrive through authenticated executions, a public function domain, a proxy, or a separate HTTP server.
2. Reuse the existing workspace wrapper for endpoint configuration, credentials, and SDK exports. If a required operation is missing, extend that shared boundary within the task's scope rather than duplicating clients in handlers. Respect project instructions requiring discussion before a workaround.
3. Choose the credential model explicitly: user JWT for user-permission access; narrowly scoped server credentials for privileged operations with application authorization checks.
4. Implement validation and route dispatch at the handler boundary; keep resource logic separate and inject the data client for tests. Match the project's response envelope and module/build configuration.
5. Verify behavior and document any changed permissions, environment variables, or execution configuration in existing project docs.

Use `documentation-lookup` to resolve and query Appwrite documentation when available; otherwise consult official docs. Match examples to installed SDK types. Server and SDK version numbers are separate: do not infer TablesDB support from a phrase such as “v2.0 SDK.”

## Authentication and Client Lifetime

- Appwrite Functions expose a dynamic server API key through execution headers. Use it only within a verified Appwrite execution boundary and configure minimal function scopes. An arbitrary HTTP header received by another server is not a trusted platform credential. See [function development](https://appwrite.io/docs/products/functions/develop).
- A helper that extracts `x-appwrite-user-id` or checks its presence does not authenticate a caller. For public HTTP entry points, validate the supplied user credential through Appwrite and derive identity from the authenticated account; do not authorize using a caller-supplied ID. Review the entry point against [function execution](https://appwrite.io/docs/products/functions/execute).
- Keep JWT clients separate from API-key clients. JWT authentication applies the user's permissions; API-key authentication can access resources beyond that user's permissions. Server-key operations therefore need explicit ownership or role checks. See [JWT authentication](https://appwrite.io/docs/products/auth/jwt).
- Create credential-bearing clients per request. Never cache a user's JWT or dynamic key in a shared mutable client. Keep keys and JWTs out of browser bundles, responses, and logs.
- Outside Appwrite Functions, use the deployment's approved secret source or authenticated user flow. Do not assume dynamic execution headers exist on a separate backend.

## Data Access and Routing

- For projects already using TablesDB, preserve table/row terminology and object-parameter calls through the shared wrapper. Inspect the [TablesDB API reference](https://appwrite.io/docs/references/cloud/server-nodejs/tablesDB) and installed types for exact methods. Do not migrate existing Databases callers as an unrelated cleanup.
- Keep database/table identifiers in existing configuration. Validate writable fields and derive ownership fields from verified identity, never from the request body.
- Make list pagination explicit and validate bounds against the installed API and project policy. Do not encode a universal “maximum 200” assumption. Use a stable ordering and cursor when the selected API supports it.
- Dispatch on the runtime's parsed path and method with exact route matching. Avoid prefix matching that accidentally accepts neighboring resource names. Parse bodies using the installed runtime's documented request shape.
- Keep functions stateless across executions. Stateless execution alone does not make multiple writes atomic; use supported transactions or a documented recovery strategy when a workflow needs all-or-nothing behavior.
- Catch failures at the response boundary, including client/config initialization. Return safe messages and stable error codes; log sanitized diagnostic context separately. Never send raw SDK exception messages to callers.

## Practical Scenario: Update an Owned Row

For `PATCH /items/:id` in an existing Appwrite function:

1. Match the method/path and validate the ID and allowed patch fields.
2. Authenticate the caller at the actual entry point and obtain the verified user ID.
3. Obtain the existing wrapper's request-scoped client. If it uses a server key, load the row and explicitly check ownership or the required role before writing; if using a JWT client, preserve Appwrite permissions and enforce additional business rules.
4. Apply only the allowed patch fields. Do not permit an ownership change through a general update route. If authorization-relevant state can change concurrently, use the project's transaction/concurrency strategy.
5. Return the project's success envelope. Map missing authentication, forbidden access, validation failures, and internal errors to its established HTTP/error conventions.

This scenario intentionally leaves imports, table names, and response types to the consuming project.

## Testing

- Unit-test route matching, invalid bodies, rejected ownership-field changes, and failure-to-response mapping using the shared client boundary.
- Test anonymous requests, forged user-ID headers, expired/invalid credentials, another user's row, and authorized access. Assert rejected requests perform no write.
- Use integration tests against a disposable Appwrite project to verify JWT row permissions and configured server-key scopes; mocked SDK calls cannot establish these permissions.
- Exercise pagination across multiple pages and ensure sequential requests from different users cannot share credentials.
- Run the consuming package's type check, build, and relevant tests. Exercise the critical user flow through its real entry point when authentication or deployment behavior changes.

## Related Skills

- [api-design](../api-design/SKILL.md) — endpoint and response conventions.
- [security-review](../security-review/SKILL.md) — authentication and authorization review.
- [documentation-lookup](../documentation-lookup/SKILL.md) — SDK documentation lookup.
- [tdd-workflow](../tdd-workflow/SKILL.md) — implementation tests.

## Attribution

Adapted from Boom source paths (relative to the Boom repository, not ECC): `services/AGENTS.md`, `services/functions/AGENTS.md`, and `packages/appwrite-client/src/{client,index}.ts`. The wrapper's `package.json` was checked to distinguish its actual SDK dependency from historical version labels in prose. Boom package names and deployment-specific assumptions remain project-local; this skill generalizes the shared-client workflow and clarifies credential trust boundaries.
