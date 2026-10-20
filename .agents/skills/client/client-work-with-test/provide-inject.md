# Work With Provide / Inject Tests

Use with the [core test skill](SKILL.md) when the component or composable under
test uses `provide` / `inject`.

This client rarely uses injection keys (prefer Pinia). When you add provide /
inject, follow this file.

## Export The Key

Export the injection key and its type from a `types.ts` (or adjacent module)
next to the feature. Do not keep the key private inside a composable if tests
or children need it.

```typescript
import type { InjectionKey } from 'vue';

export interface FeatureInjection {
  loadData: () => Promise<void>;
}

export const featureInjectionKey = Symbol(
  'featureInjectionKey',
) as InjectionKey<FeatureInjection>;
```

`provide` is available only to **descendant** components, not to the component
that called `provide`.

## Composable Error Path

Use `inject(key, null)` when a missing provider is an expected error path —
avoids Vue warnings in strict test mode:

```typescript
const injection = inject(featureInjectionKey, null);

if (!injection) {
  throw new Error('need provider at the top level');
}
```

## Component Tests

Pass the provided value through `global.provide` with the exported key:

```typescript
import { featureInjectionKey } from '../types';

const getWrapper = () =>
  shallowMount(ChildComponent, {
    global: {
      provide: {
        [featureInjectionKey as symbol]: {
          loadData: vi.fn(),
        },
      },
    },
  });
```

## Provider / Consumer Split

When the provider creates reactive state itself, mount a parent that calls the
provider setup and a child that calls the consumer:

```typescript
const Parent = defineComponent({
  setup() {
    useFeature().initFeature();
  },
  render: () => h(Child),
});

wrapper = mount(Parent);
```

Do not call provider and consumer setup in the same component unless the test
passes the injection through `global.provide`.

## Checklist

- Injection keys are exported for tests.
- Tests use `global.provide` or a parent/child mount when split.

## Anti-Patterns

```typescript
// Calling provider and consumer setup in the same component
setup() {
  useFeature().initFeature();
  return useFeature().useFeatureConsumer();
}
```
