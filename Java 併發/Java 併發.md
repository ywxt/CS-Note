
併發問題的三大根源

1. 可見性：由於 Cache 的原因，值被修改後並不會立即寫入內存，而是在 Cache 中，其他線程讀取時會讀到舊數據
2. 原子性：由於分時復用，多步指令中途被打斷。
3. 有序性：指令重排
	1. 第一級：編譯器重排序
	2. 第二級：指令級並行重排序
	3. 第三級：內存系統重排序

### CAS (Compare And Swap)

CAS，比較和交換需要原子性，由硬件完成

三個值，*比較的地址* *預期的舊值* *新值*

### ABA

指 CAS 中，如果一個值由 `A` 改到 `B` 再改到 `A` ，CAS類會認爲沒有變化，可以使用 `AtomicStampedReference` 來使用版本解決問題

## 同步的實現方法

1. 非阻塞同步
	1. CAS
	2. AtomicInteger
2. 互斥同步
	1. synchronized
	2. ReentrantLock
3. 無同步
	1. 棧封閉：只使用本地變量
	2. ThreadLocal
	3. 可重入代码(Reentrant Code)：以在代码执行的任何时刻中断它，转而去执行另外一段代码(包括递归调用它本身)，而在控制权返回后，原来的程序不会出现任何错误。
