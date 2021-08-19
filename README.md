# The Effect Hook lets you perform side effects in function components

Use `useRef` to store the `mount` | `update` | `unmount` status.

In `useEffect`, return a destructure function to retrieve the state of `unmount`.

<div align="middle">
  <img src="https://i.imgur.com/NomR1lZ.jpg" />
</div>

### Explanation of Parameters

* `effect` Imperative function with the ability to return a cleanup function.

* `deps` If the effect is present, it will only be activated if the values in the list change.

* `returns` The lifecycle status, which is used outside of this hook.

```typescript
import { useRef, useEffect, DependencyList, MutableRefObject } from 'react';

type LifecycleStatus = 'mount' | 'update' | 'unmount';

type StatusEffectCallback = (status: MutableRefObject<LifecycleStatus>) => void | (() => void);

export function useStatusEffect(effect?: StatusEffectCallback, deps?: DependencyList) {

  const status = useRef<LifecycleStatus>('mount');
  const mounted = useRef(false);

  useEffect(() => {
  
    if (!mounted.current) {
      mounted.current = true;
      return;
    }
    
    status.current = 'update';
    
    return () => { 
        status.current = 'unmount'; 
    };
    
  }, deps);

  // Call the callback.
  useEffect(() => {
    if (!effect) { 
        return; 
    }
    return effect(status);
  }, deps);

  return { status };
}
```
### Usage
```typescript

useStatusEffect((status) => {

    // We'll skip the initial render since we only want to evaluate this effect when the variable changes.
    if (status.current === 'mount') {
      return;
    }

    // When `variable` changes, get data from the API.
    myFunction(variable).then((result) => {
      if (status.current !== 'unmount') {
        setResults(result);
      }
    }).catch((error) => {
      if (status.current !== 'unmount') {
        setError(error);
      }
    });

    // Destructure is not required.
    // Since status includes the current status and is set when the hook is destructured.
    /* 
    return () => {
      status.current = 'unmount';
    }
    */

  }, [variable]);
```


<div align="middle">
  <img src="https://i.imgur.com/D9QRY5W.png" />
</div>
