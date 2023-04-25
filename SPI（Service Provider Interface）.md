SPI 是一種服務發現機制。

需要在`META-INF/services/`下創建與接口同名的文件，內容則是實現的類。

**目前的JDBC就是 SPI 實現的**

調用時，使用
```java
ServiceLoader<Search> s = ServiceLoader.load(Search.class);
```
來加載所有的實現類。


