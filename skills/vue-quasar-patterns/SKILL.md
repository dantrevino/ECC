---
name: vue-quasar-patterns
description: Build and review Vue 3 applications using Quasar components, Composition API, shared design tokens, and mobile-first forms. Use for Quasar UI work; plain Vue and Nuxt work belongs to vue-patterns.
metadata:
  origin: ECC
---

# Vue and Quasar Application Patterns

Adapted from Boom's Vue/Quasar project instructions. Apply the reusable patterns
below within the target project's own conventions. Boom-specific formatting,
package names, and design direction belong to `boom-project`.

## When to Activate

- Editing Vue single-file components in a project that depends on Quasar.
- Connecting Quasar forms to composables, stores, or existing service clients.
- Applying a shared design-token system to mobile-first Quasar screens.

## How It Works

1. Read the target package manifest, Quasar configuration, nearest project
   instructions, and an adjacent component. Confirm the active app mode and
   scripts; preserve existing PWA, boot-file, router, and plugin wiring.
2. Use existing Quasar components before introducing custom replacements.
   Keep domain calculations in pure utilities and reusable stateful behavior in
   composables. Components own presentation and user interaction.
3. Use Composition API and `<script setup lang="ts">` where that is the local
   convention. Follow the target's SFC section order and formatter rather than
   reformatting unrelated components to match an example.
4. Use existing service clients for requests. Keep credentials and signing
   material out of component state, persisted stores, and browser logs.
5. Verify the changed interaction with the project's component tests and, for
   a critical flow, its browser tests. Run its type-check and build scripts.

## Forms and State

- Give fields labels and useful validation messages. Quasar `QForm` coordinates
  child components' internal `rules`; native attributes alone do not define all
  the checks performed by `QForm.validate()`. See the [QForm documentation](https://quasar.dev/vue-components/form/).
- Render pending, success, and failure states explicitly. Prevent duplicate
  submission while a request is in flight; retain recoverable input on failure.
- Treat frontend validation as feedback. Enforce authorization and validation at
  the backend boundary as well.
- Keep derived values computed rather than synchronizing duplicate state with
  watchers. Dispose subscriptions, timers, and event listeners when their owner
  ends; see [Vue composables](https://vuejs.org/guide/reusability/composables.html).
- Follow existing Pinia/composable boundaries. Do not introduce a new global
  store for state used only inside one form.

## Design Integration

- Read the app's design specification and token package exports before styling.
  Reuse semantic colors, surfaces, spacing, and typography.
- Edit token source files and run the existing generator when changing shared
  values. Do not patch generated CSS or hardcode replacement colors locally.
- Use the existing theme-aware composable for dynamic colors and the established
  CSS variables for static styles. Verify both supported color modes.
- Check narrow screens, keyboard navigation, visible focus, and text wrapping.
  Preserve the project's sizing conventions and existing Quasar overrides.

## Example: Add a Recipient Form

Inspect the neighboring payment screen and existing address-validation utility.
Build the form with the project's Quasar fields and buttons, route submission
through its existing service/composable, and show a pending state. Test invalid
input, a failed request followed by retry, duplicate clicks, and success. Use
test doubles or the configured test environment rather than making a real payment.

## Avoid

- Replacing Quasar controls with a second UI framework for one screen.
- Copying React/Next.js state or routing patterns into Vue components.
- Changing app modes, package managers, or the design system as incidental cleanup.

## Sources and Related Skills

Derived from the Boom repository's `AGENTS.md`, `apps/boom/AGENTS.md`, and
`.skills/design-system.md`. These are provenance paths in Boom, not ECC files.
Check installed package versions and official Vue/Quasar docs for API changes.

- `vue-patterns` — general Vue reactivity and component guidance.
- `e2e-testing` — critical browser interactions.
- `boom-project` — Boom's scoped conventions and source map.
