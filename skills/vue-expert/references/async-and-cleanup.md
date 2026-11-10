# Watchers, async work and cleanup

- Watch the exact dependency, not the entire route/store by habit. watchEffect tracks only synchronously accessed dependencies before await. Choose flush timing from actual DOM needs; nextTick is not a race fix.
- Register cleanup before awaiting. onWatcherCleanup is Vue 3.5+ and must run synchronously; the watcher callback's onCleanup argument is another API with different constraints. Do not assume a cleanup registered after a request finishes can cancel it.
- Abort cooperative IO and also prevent stale success/error/finally from overwriting current state. A loader may ignore cancellation or complete before cancellation takes effect.
- Watchers created synchronously in component setup are normally stopped on unmount. Watchers started later in timers/async callbacks need an explicit owner/stop. Use effectScope/onScopeDispose for owned composable effects where appropriate.
- Remove listeners, intervals, chart/socket subscriptions and pending work through their actual lifecycle owner. KeepAlive deactivation is not unmount; decide whether effects should pause there.
- SSR cannot rely on browser globals or mounted hooks during server render. Keep request-specific state separate; validate hydration in a browser when it is affected.

Illustrative latest-result composable; call synchronously in setup or an owned effect scope. The loader must return the already validated domain value; input errors and cancellation policy remain application concerns.

```javascript
import { shallowRef, ref, watch } from 'vue';

function useLatest(source, load) {
  const data = shallowRef(null);
  const error = shallowRef(null);
  const pending = ref(false);
  const stopWatch = watch(source, async (key, _previous, onCleanup) => {
    let active = true;
    const controller = new AbortController();
    onCleanup(() => { active = false; controller.abort(); });
    pending.value = true;
    error.value = null;
    try {
      const result = await load(key, { signal: controller.signal });
      if (active) data.value = result;
    } catch (cause) {
      if (active) error.value = cause;
    } finally {
      if (active) pending.value = false;
    }
  }, { immediate: true });
  const stop = () => { stopWatch(); pending.value = false; };
  return { data, error, pending, stop };
}
```

This retains previous data during a new request; clear it deliberately if the UI contract requires that. It is not a cache, retry manager or replacement for existing server-state tooling.

Source: [Vue watchers](https://vuejs.org/guide/essentials/watchers.html), [composables](https://vuejs.org/guide/reusability/composables.html).
