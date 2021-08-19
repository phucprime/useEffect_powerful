# Effect Hook cho phép thực hiện side effect bên trong các function component

Dùng `useRef` để lưu trữ các trạng thái `mount` | `update` | `unmount`.

Trong `useEffect`, trả về một destructure function để truy xuất trạng thái `unmount`.

<div align="middle">
  <img src="https://i.imgur.com/NomR1lZ.jpg" />
</div>

### Giải thích các parameters

* `effect` Imperative function với khả năng trả về một cleanup function.

* `deps` Nếu effect hiện diện, nó sẽ chỉ được kích hoạt nếu các giá trị trong danh sách có sự thay đổi.

* `returns` Trả về trạng thái của vòng đời, được sử dụng bên ngoài hook này.

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

  // Gọi hàm callback.
  useEffect(() => {
    if (!effect) { 
        return; 
    }
    return effect(status);
  }, deps);

  return { status };
}
```
### Sử dụng
```typescript

useStatusEffect((status) => {

    // Bỏ qua lần render ban đầu vì tôi chỉ muốn sử dụng effect này khi `variable` có sự thay đổi.
    if (status.current === 'mount') {
      return;
    }

    // Khi `variable` có sự thay đổi, lấy dữ liệu từ API.
    myFunction(variable).then((result) => {
      if (status.current !== 'unmount') {
        setResults(result);
      }
    }).catch((error) => {
      if (status.current !== 'unmount') {
        setError(error);
      }
    });

    // Destructure không cần thiết lúc này.
    // Vì status đã được bao gồm status hiện tại và được đặt khi hook bị hủy.
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
