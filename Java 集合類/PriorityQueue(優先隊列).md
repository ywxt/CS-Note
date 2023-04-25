## 1. 存儲

![[二叉堆.png]]

優先隊列**邏輯上是完全二叉樹實現的小頂堆**，近似於二叉樹。**實際存儲是數組。**

### 父子節點關係

1. *若父節點在數組中位置爲 $n$，則左孩子位置爲 $2n + 1$， 右孩子位置 $2n + 2$*
2. 同理，*若孩子節點位置爲 $n$，則父節點爲 $\frac{n-1}{2}$ 或者 `(n-1)>>>1`*

### 節點間關係

1. 父節點*小於等於***所有**子節點
2. 同一級的左右節點不需要有序，而是在輸出的時候判斷

### 判斷葉子節點與非葉子節點

- 葉子節點: **index >= size >>> 1** 下標大於等於一半即是葉子節點

### 時間複雜度

- `peek` 和 `element` 是常數複雜度
- `add`、`offer`、無參 `remove` 和 `poll` 的複雜度是 $\log (N)$

### 類方法

#### 1. add 和 offer
`add` 和 `offer` 語義相同，區別在於 `add` 失敗時會拋異常，`offer` 失敗時返回 `false`。
 ![[優先隊列offer.png]]
過程：
1. 首先把新的元素加入尾部
2. 然後，依次比較他與其父節點的大小 `element <= queue[(index-1) >>> 1]`
3. 若不滿足，則交換，接着進行第二步，直到 `element <= queue[(index-1) >>> 1]`

實現：
```java
public void add(int num) {  
// grow  
	if (size - queue.length >= 0) {  
		queue = Arrays.copyOf(queue, queue.length + (queue.length >>> 1));  
	}  
	int i = size;  
	size++;  
	queue[i] = num;  
	shiftUp(i);  
  
}  
  
private void shiftUp(int index) {  
	if (index == 0) return;  
	int parent = (index - 1) >>> 1;  
	if (queue[parent] <= queue[index]) {  
		return;  
	}  
	int tmp = queue[index];  
	queue[index] = queue[parent];  
	queue[parent] = tmp;  
	shiftUp(parent);  
}
```

#### 2. element 和 peek

取堆頂的元素，兩者作用相同，不同在於 `element` 爲空時會拋異常。

```java
public int peek() {
	// 應該判斷一下範圍
	return queue[0];
}
```

#### 3. remove 和 poll
![[優先隊列remove 和 peek.png]]
刪除堆頂元素，時間複雜度 $\log(N)$

方法：
1. 把最後一個元素放到堆頂
2. 比較父節點和左右節點中較小的一個
3. 如果大於，則交換
4. 重複 2-3

```java
public int remove() {  
	size--;  
	int e = queue[0];  
	queue[0] = queue[size];  
	shiftDown(0);  
	return e;  
}  
  
private void shiftDown(int index) {  
	int left = index * 2 + 1;  
	if (left >= size) {  
		return;  
	}  
	int right = index * 2 + 2;  
	int min;  
	if (right >= size) {  
		min = left;  
	} else {  
		min = queue[left] < queue[right] ? left : right;  
	}  
	if (queue[min] < queue[index]) {  
		int tmp = queue[index];  
		queue[index] = queue[min];  
		queue[min] = tmp;  
		shiftDown(min);  
	}  
}
```

#### 4. `remove(o)`
![[PriorityQueue remove(o).png]]
1. 刪除最後一個節點：直接刪除
2. 刪除中間節點：和 `remove` 類似，但是把最後一個放到刪除的位置

```java
public boolean remove(int num) {  
	int index = -1;  
	for (int i = 0; i < size; i++) {  
		if (num == queue[i]) {  
			index = i;  
		}  
	}  
	if (index == -1) return false;  
	size--;  
	queue[index] = queue[size];  
	shiftDown(index);  
	return true;  
}
```

