
## JDK 7

使用了鏈地址法
- 負載系數（load factor）：決定擴容的臨界值，當數量超過 $capacity \times load\_factor$ 時，則進行擴容，並重新 Hash。

## JDK 8

使用了 數組 + 鏈表 + 紅黑樹
當鏈表中元素數組**到達8個時，鏈表會轉化成紅黑樹**，查找時時間複雜度降爲 $O(\log N)$

![[JDK8 HashMap.png]]

- *默認 `load_factor` ： 0.75*
- *初始容量：`1<<4` 16*
- **HashMap 所有的容量必須是2的冪**
- 每次擴容到原來的 **2倍**，**ArrayList 是原來的1.5倍**

