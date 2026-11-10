# Reactivity and component contracts

- Find the owner of state before changing ref/reactive/computed/watch. Keep derived values computed where appropriate; use watchers for effects, not redundant mirrors of the same state.
- Destructuring a reactive object's primitive property captures a value. Use a getter/toRef/toRefs where a reactive connection is needed. Object-valued properties can retain a reactive object reference but not follow replacement of the parent property.
- Reactive props destructure is a compiler feature of Vue 3.5+ in script setup, not a universal property of JavaScript destructuring. Watch a getter such as () => title; passing the current primitive value does not make it a reactive source. Check matching compiler/runtime versions.
- Deep refs/reactive are appropriate when nested mutation is intentional. shallowRef is useful for externally owned objects or replace-as-a-whole data, not a universal optimization. Its nested mutation does not trigger deep updates by itself.
- Props are parent-owned; do not mutate them or nested objects as a hidden child-to-parent channel. Use explicit events or an agreed v-model contract. Component events do not bubble like DOM events.
- defineModel requires Vue 3.4+. Its default can desynchronize parent and child when the parent supplies undefined; choose a deliberate initial-value contract. Preserve established modelValue/update:modelValue code without mandatory migration.
- Define props/emits/runtime validation consistently; TypeScript declarations are not general validation for API responses. Keep unit/nullability semantics explicit.
- Stable list keys preserve identity; forcing a new key can destroy local state and subscriptions. Understand v-if/v-show/KeepAlive lifetime before fixing a rendering issue with remounting.
- Keep templates/composables aligned with project style. Split by real responsibility/reuse, not a fixed component count or mandatory feature-folder template. Do not migrate Options API or script ordering incidentally.
- Diagnose runtime warnings from the concrete stack/render branch, then trace source ownership and updates. Avoid deep watches, forceUpdate, nextTick chains or global state as symptom patches.

Sources: [Reactivity](https://vuejs.org/guide/essentials/reactivity-fundamentals.html), [script setup](https://vuejs.org/api/sfc-script-setup.html), [component v-model](https://vuejs.org/guide/components/v-model.html).
