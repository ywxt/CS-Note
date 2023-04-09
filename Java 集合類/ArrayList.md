## 1. 構造過程

1. 如果無參 —— 使用 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}` 空的數組，並在第一次 `add` 時擴容
2. 如果指定容量，則使用給定的值
3. 擴容：在 `add` 調用 `ensureCapacityInternal` 方法
4. `ensureCapacityInternal`： 計算需要的容量，然後擴容
5. 計算容量（`calculateCapacity`）：如果是`DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 是 $\min (10, size+1)$，否則就 $size$
	**初始容量： 10**
6. 調用 `grow` 方法擴容，**新容量：oldCapacity + (oldCapacity >> 1)**，即**每次擴容到原來的 1.5 倍**

## 2. 攷點

1. **默認大小： 10**
2. **擴容大小：原來的 1.5 倍**
3. `grow` 中 `if` 的目的 —— 攷慮溢出
```java
int newCapacity = oldCapacity + (oldCapacity >> 1);  
if (newCapacity - minCapacity < 0)  
newCapacity = minCapacity;  
if (newCapacity - MAX_ARRAY_SIZE > 0)  
newCapacity = hugeCapacity(minCapacity);  
// minCapacity is usually close to size, so this is a win:
```

