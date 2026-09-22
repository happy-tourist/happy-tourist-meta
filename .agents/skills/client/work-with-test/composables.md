# Work With Lib / Composable Tests

Use with the [core test skill](SKILL.md) when the unit under test is a pure
helper under `src/lib/` or a small composable — not a `.vue` page.

Examples in this client: `passwordPolicy.ts`, `passwordStrength.ts`, and any
future board math helpers that stay client-side.

## How to invoke

Pick **one** pattern. Prefer the simplest that still exercises the contract.

### A. Call directly

Use when the module does not need a component instance or Vue lifecycle.

```typescript
import { describe, expect, it } from 'vitest';
import { passwordPolicyRule } from '@/lib/passwordPolicy';

describe('passwordPolicy', () => {
  it.each`
    password        | ok
    ${'short'}      | ${false}
    ${'Abcd1234!'}  | ${true}
  `('$password → $ok', ({ password, ok }) => {
    const result = passwordPolicyRule(password);
    expect(result === true).toBe(ok);
  });
});
```

Adapt to the real export shape (`passwordPolicyRule` may return `true | string`).

### B. Thin Vue wrapper

Use only when you need a mountable surface (slots, lifecycle). Add a tiny
`defineComponent` / `.vue` under `__tests__`, `shallowMount` via `getWrapper()`,
drive clicks / props. Do **not** invent a wrapper when pattern A is enough.

## What to mock

- External modules the helper imports (rarely needed for pure policy).
- Colyseus / stores only if the composable actually calls them →
  [colyseus.md](colyseus.md), [stores.md](stores.md).

Do **not** mock the SUT module itself.

## Mocking one export from a barrel

Prefer `vi.spyOn` on the real module so other exports stay real. If you
`vi.mock`, list an explicit factory (stub substituted API; import real helpers
from their source file). Avoid `importOriginal` + `…actual` spreads unless
necessary — see [SKILL.md](SKILL.md) anti-patterns (CSM lesson).

## What to verify

- Early-return / invalid inputs.
- Boundary cases via `it.each`.
- Return contract (`true` vs error string; strength score bands).

Do not assert implementation details or unconditional pass-through with no
behavior.

## Checklist

- Pure libs: direct call (pattern A).
- One export stub: `vi.spyOn` or explicit factory.
- No Vue mount for pure functions.
