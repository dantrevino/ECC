---
name: stacks-clarity
description: Build and review Stacks Clarity contracts and their transaction clients, with Clarinet checks, principal-aware authorization, escrow accounting, and post-condition verification.
metadata:
  origin: ECC
---

# Stacks and Clarity

Apply contract-specific reasoning to authorization, asset custody, state transitions, and client transactions. Preserve the target project's Clarity version and test harness.

## When to Activate

- Editing or reviewing `.clar` contracts or Clarinet configuration.
- Changing Stacks transaction construction, escrow, settlement, or wallet post-conditions.
- Debugging contract errors, caller identity, asset mismatches, or execution costs.

## Discover the Project First

1. Read applicable project instructions, `Clarinet.toml`, the target contract, package scripts, and existing tests. Record the manifest contract name, source path, epoch, language version, and required traits. Different contracts can use different versions.
2. Find the installed Clarinet SDK and Stacks.js versions before adopting API examples. Use [documentation-lookup](../documentation-lookup/SKILL.md) when available, or official documentation matching those versions.
3. Use the owning package's test script. A workspace can use Vitest with the Clarinet SDK rather than `clarinet test`; neither a directory named `blockchain` nor a package name proves where tests live.
4. Inspect existing transaction clients when changing a public function: argument encoding, error codes, asset identifiers, network, and post-conditions are part of its interface.

## Authorization and Custody

For each changed entrypoint, write down the intended authorized principal, permitted call paths, asset owner before and after execution, and external callees.

- `tx-sender` follows the originating sender until a context switch; `contract-caller` identifies the immediate caller. `as-contract` changes both to the contract principal within its expression. Trace these values across each call boundary. [Keyword reference](https://docs.stacks.co/reference/clarity/keywords)
- Choose guards from the trust model. Checking the transaction sender can admit calls forwarded by an untrusted contract; checking the immediate caller can intentionally prohibit otherwise legitimate proxy calls. Neither “all transfers use tx-sender” nor “all admin functions use contract-caller” is a universal rule.
- Test direct calls, authorized forwarding where supported, and unauthorized intermediary calls. Include state-only attacks: wallet asset post-conditions do not establish authorization for configuration changes.
- For escrow, identify who sends, holds, and releases each asset. Check the called token contract's authorization behavior inside any context switch.
- Keep native STX, SIP-010 fungible tokens, and SIP-009 NFTs as explicit transfer paths. Validate contract principals and asset identity; enforce allowlists when the product supports a restricted asset set. Implementing a trait alone does not establish trust.
- Match transaction post-conditions to expected outflows, principals, asset identifiers, and amounts across the complete call. Test deny-mode compatibility; do not remove protection just to make a transaction succeed. [Post-condition examples](https://docs.stacks.co/post-conditions/examples)
- Newer Clarity contract allowances such as `as-contract?` require version support. Do not substitute them into an older contract without a deliberate migration. [Contract post-conditions](https://docs.stacks.co/cookbook/clarity/cryptography-and-security/contract-post-conditions)

## State and Arithmetic

- Reject impossible states when writing them: unsupported assets, unusable recipients, invalid expiry, and settlement parameters that cannot be fulfilled.
- Use `asserts!` for invariants, `unwrap!` for required optionals, and `try!` for responses whose failure must propagate. Preserve the project's stable public error codes and response types.
- Check integer rounding, overflow, underflow, fee bounds, and conservation of assets across settlement and refunds. Use explicit base units in client code; do not pass large token amounts through lossy JavaScript numbers.
- Bound batch size and work per item. Inspect costs for batch operations, loops, and multiple transfers after correctness is established.
- Identify whether expiry uses Stacks height, burn height, tenure, or time. Use the matching version-supported keyword and test helper; do not assume advancing one clock advances another equivalently.

## Verification Workflow

Run `clarinet check` from the manifest directory, before edits when a baseline is needed and after edits. Separate target failures, unrelated workspace failures, and checker warnings; an unchecked-data warning is a review lead, not proof of an exploit.

From the owning package directory, after confirming these commands exist:

```sh
clarinet check
pnpm run test
```

Use the project's package manager in place of `pnpm` where appropriate. Add a failing test for the changed state transition, implement the change, then run the affected tests and relevant suite. Verify:

- Success, unauthorized caller, invalid asset, missing state, and repeated settlement/refund.
- Balances and custody as well as returned values; failure must not leave partial settlement.
- Expiry just before, at, and after the boundary using the configured chain model.
- Fee rounding and maximum supported input sizes.
- Client argument encoding and post-conditions for changed transfers. A successful simnet call alone does not prove wallet transaction post-condition behavior; exercise it in a supported transaction harness or devnet when needed.

Use installed console/SDK cost tooling for expensive paths; verify command availability against that version. Report missing tooling or unrelated failures accurately. Routine development stays in simulation/local testing; broadcasting or deploying contracts requires authorization for the specific network and action.

## Example: Add an Escrow Refund

Request: “Let the buyer refund an expired purchase.”

Read the order map, expiry clock, and payment asset path. Define who may request a refund, whether proxy calls are supported, and which principal currently holds the funds. Add tests for early refund, exact expiry, wrong caller, successful balance restoration, and a second refund. Check settlement cannot also succeed afterward. Update the client's expected contract outflow and test its post-conditions. Run the manifest check and package tests; report cost impact if the refund adds external calls or batch work.

## Related Skills

- [security-review](../security-review/SKILL.md) for authorization and trust-boundary review.
- [tdd-workflow](../tdd-workflow/SKILL.md) for regression-driven implementation.
- [documentation-lookup](../documentation-lookup/SKILL.md) for installed-version API verification.

## Source Provenance

Adapted from the user's Boom project contract instructions (`packages/clarity/AGENTS.md`), checked against its `Clarinet.toml`, package scripts, marketplace contract, and Clarinet SDK collection tests. Project-specific names, deployment principals, and stale `blockchain/` test paths were removed. The original caller-selection heuristic was replaced with explicit principal reasoning verified against the official Stacks references linked above. These source artifacts explain the adaptation; using this skill does not require the original project.
