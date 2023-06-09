
## 字節碼

- `monitorenter`: 進入鎖
- `monitorexit`：釋放鎖

![[synchronized 過程.webp]]

**如果第一次沒有成功得到鎖，就會加入同步隊列，線程Blocked，其他線程釋放鎖後，通知隊列，再出隊列**

## 可重入鎖原理：記錄加鎖次數

- 可重入：（来源于维基百科）若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant或re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

- 可重入锁：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。


**这就是Synchronized的重入性，即在同一锁程中，每个对象拥有一个monitor计数器，当线程获取该对象锁后，monitor计数器就会加一，释放锁后就会将monitor计数器减一，线程不需要再次获取同一把锁。


## 保證可見性原理：內存模型和 happens-before 規則

 
![[happens-before.webp]]

```java
public class MonitorDemo { 
	private int a = 0; 
	public synchronized void writer() { // 1 
		a++; // 2 
	} // 3 
	public synchronized void reader() { // 4 
		int i = a; // 5 
	} // 6 
}

```

在图中每一个箭头连接的两个节点就代表之间的happens-before关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程A释放锁happens-before线程B加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来happens-befor关系，通过传递性规则进一步推导的happens-before关系。现在我们来重点关注2 happens-before 5，通过这个关系我们可以得出什么?

根据happens-before的定义中的一条:如果A happens-before B，则A的执行结果对B可见，并且A的执行顺序先于B。线程A先对共享变量A进行加一，由2 happens-before 5关系可知线程A的执行结果对线程B可见即线程B所读取到的a的值为1。

**happens-before 保證 第一個 `synchronized` 會在第二個 `synchronized`  之前完成，且保證 `a` 修改後的值在第二個 `synchronized` 可見**


## JVM 鎖優化

- `锁粗化(Lock Coarsening)`：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。
    
- `锁消除(Lock Elimination)`：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本的Stack上进行对象空间的分配(同时还可以减少Heap上的垃圾收集开销)。
    
- `轻量级锁(Lightweight Locking)`：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒(具体处理步骤下面详细讨论)。
    
- `偏向锁(Biased Locking)`：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟。
    
- `适应性自旋(Adaptive Spinning)`：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态。
    

### 自旋鎖與自適應自旋鎖

#### 自旋鎖

> 引入背景：大家都知道，在没有加入锁优化时，Synchronized是一个非常“胖大”的家伙。在多线程竞争锁时，当一个线程获取锁时，它会阻塞所有正在竞争的线程，这样对性能带来了极大的影响。在挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作对系统的并发性能带来了很大的压力。同时HotSpot团队注意到在很多情况下，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和回复阻塞线程并不值得。在如今多处理器环境下，完全可以让另一个没有获取到锁的线程在门外等待一会(自旋)，但不放弃CPU的执行时间。等待持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需要让线程执行一个忙循环(自旋)，这便是自旋锁由来的原因。

- 如果鎖佔用的時間很短，自旋鎖性能很好。如果同步代碼執行時間較長，則會佔用高 CPU

#### 自適應自旋鎖

**自適應鎖的自旋時間不固定，而是由前一次同一個鎖的自旋時間以及鎖的擁有者的狀態來決定**

- 如果同一個鎖對象上，剛剛成功獲取過鎖，並且持有鎖的線程正在運行，那麼 JVM 會認爲獲取到鎖的可能性很大，會增加等待時間。
- 如果前面幾次很少獲取到鎖，那麼就會減少等待時間。儘早進入 Blocked



### 鎖消除

JVM 通過逃逸分析，分析出線程獨佔的變量，即可消除對應對象的鎖

### 鎖粗化

如果在相近的代碼中反覆加鎖會降低性能，所以把多個鎖優化爲一個鎖。

```java
StringBuffer sb = new StringBuffer()
sb.append(s1);
sb.append(s2);
sb.append(s3);
return sb.toString()
```

`append` 是加鎖的，JVM 可以在第一次 `append` 加鎖，最後一個 `append` 釋放鎖

## 鎖升級

對於 Synchronized ，JVM 會根據競爭激烈程度逐漸升級，四種狀態分別爲 *無鎖*、 *偏向鎖*、*輕量級鎖*、*重量級鎖*

**鎖升級的過程是不可逆的**

### 輕量級鎖

在线程执行同步块之前，JVM会先在当前线程的栈帧中创建一个名为锁记录( `Lock Record` )的空间，用于存储锁对象目前的 `Mark Word` 的拷贝(JVM会将对象头中的 `Mark Word` 拷贝到锁记录中，官方称为 `Displaced Mark Ward` )这个时候线程堆栈与对象头的状态如图：

![[輕量級鎖CAS操作之前的棧.webp]]

對象頭：**無鎖標誌爲 `01` 狀態。**

#### 加鎖

首先創建 Lock Record 空間，用於放入 `Mark Word` 的拷貝。JVM 使用 CAS 操作把對象的 `Mark Word` 放到線程 `Lock Record` 中，然後將 `Mark Word` 更新爲指向 `Lock Record` 的指針（放在 HashCode），表示該線程獲得了此對象的鎖。然後將 **對象`Mark Word ` 的標誌位更新爲 `00`， 表示對象處於輕量級鎖狀態**。
![[加輕量級鎖後.png]]

**如果更新失敗**
- 先檢查對象的`Mark Word`是否存在指向當前線程`Lock Record`的指針
- 如果有，說明已經取得了鎖，可以直接調用方法塊
- 如果沒有，就說明有多個線程在競爭這一個鎖，這時會升級成*重量級鎖*，**未得到鎖的線程會阻塞**，`Mark Word` 的標誌變爲 `10` ，並存儲指向 `Lock Record` 重量級鎖的指針。


#### 解鎖

解鎖時使用 `CAS` 操作把 `Dispalced Mark Word` 替換到對象頭中。成功就解鎖，失敗說明有競爭，JVM 會轉化爲 重量級鎖。


![[輕量級鎖及膨脹流程圖.png]]

### 偏向鎖

> 在大多实际环境下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，那么在同一个线程反复获取所释放锁中，其中并还没有锁的竞争，那么这样看上去，多次的获取锁和释放锁带来了很多不必要的性能开销和上下文切换。

**Java SE 1.6**

在一個線程加鎖時，會在對象頭和棧的鎖記錄中存儲偏向的線程 ID，之後此線程進入和退出時，會先判斷對象頭的 `Mark Word` 是否存儲了當前線程的偏向鎖，如果是，表示已經取得了鎖，直接進入同步代碼塊。如果未有偏向鎖，再通過 CAS 加輕量級鎖。

![[加偏向鎖過程.png]]

#### 偏向鎖撤銷

**出了代碼塊，偏向鎖不會釋放，只有當有其他線程競爭鎖時，纔會在<u>全局安全點</u>處釋放**。

首先，暫停線程。如果持有偏向鎖的線程還處於不活動，直接把對象頭置爲無鎖狀態。
如果處於活動狀態，遍歷棧幀中所有的 `Lock Record` ，要麼偏向其他線程，要麼設置爲無鎖或標記對象不適合作爲偏向鎖。

![[偏向鎖獲得和撤銷過程.png]]

### 锁的优缺点对比

| 鎖       | 優點                                                             | 缺點                                 | 使用場景                           |
| -------- | ---------------------------------------------------------------- | ------------------------------------ | ---------------------------------- |
| 偏向鎖   | 加鎖和解鎖不使用 CAS，沒有額外的性能消耗，與同步方法比較相差納鈔 | 如果不同線程存在競爭，帶來額外的消耗 | 只有一個線程訪問同步塊             |
| 輕量級鎖 | 使用自旋鎖，競爭的線程不會阻塞，提高響應速度                     | 自旋會消耗 CPU                       | 追求響應時間，同步塊的執行時間很短 |
| 重量級鎖 | 阻塞線程，不消耗 CPU                                             | 響應時間長，加鎖解鎖耗時             | 追求吞吐量，同步塊執行比較快       |

## Synchronized 和 Lock

### synchronized 的缺陷

- 效率低：只有執行完纔能釋放鎖，無法設定超時，不能中斷一個正在使用鎖的線程。
- 不夠靈活：加鎖和釋放的時機單一，只有一個條件
- 不法知道是否成功獲得鎖：Lock 可以拿到鎖的狀態

### Lock 解決方法

`Lock` 和 `Condition` 配合實現多條件加鎖。

當多個線程競爭鎖時，未得到鎖的線程會不停地嘗試獲取鎖。`ReentrantLock` 如果一個線程等待時間太長，可以中斷自己的線程，然後 `ReerntrantLock` 響應這個中斷，不再讓線程繼續等待。使用此機制，可以避免死鎖。

### 使用 synchronized 的注意事項

- 同步代碼不能過大
- 在有選擇的情況下，優先使用 `java.util.concurrent` 中的線程安全的數據結果結構。
- Synchronzied 不是公平鎖，可能會造成線程飢餓。