# Mono/Flux 创建方式
## 简单元素生成
just
```java
Flux.just("a", "b", "c");
```
Flux#range 从整数范围进行遍历
Mono#justOrEmpty 从可能是null值的生成元素
Mono#fromSupplier 从Supplier生成单一元素
Mono#fromRunnable 从一个Runnable Task生成单一元素
Mono#fromFuture 从CompletableFuture生成单一元素
empty 空元素集
error 立即抛出异常
defer 推迟到订阅时执行
using 从disposable source生成元素流

## 操作Mono/Fluxmap
通过 mapper 对元素进行转换操作
```java
map(Function<? super T, ? extends V> mapper)
  Flux<Integer> ints = Flux.range(1, 4);
  Flux<Integer> mapped = ints.map(i -> i * 2);
```
filter
通过给定 Predicate 对每个元素进行判断，如果条件为真，则该元素会被发送；反之被忽略
```java
filter(Predicate<? super T> p)

        Flux<Integer> ints = Flux.range(1, 4);
        Flux<Integer> filtered = ints.filter(i -> i % 2 == 0);
```
buffer
将上游元素分段，当元素积累了一定数量后一并发送（组成列表，注意 buffer 后的类型 List）
```java
Flux<Integer> ints = Flux.range(1, 40);
Flux<List<Integer>> buffered = ints.buffer(3);
```
retry
在响应式流传递过程中报错时重新订阅这个发布（在 Flux 中要注意元素重复消费的问题）
```java
 Flux.log().retry(3);
```
zip
将两个响应式流合并成一个，直到其中任意一个流元素耗尽，这个是非常有用的。
```java
  Flux<Integer> fluxA = Flux.range(1, 4);
  Flux<Integer> fluxB = Flux.range(5, 5);
  fluxA.zipWith(fluxB, (a, b)-> a+b)
```
其他
cast 对元素进行强制类型转换
Flux#index 对元素在流中的位置下标进行注明（从0开始），返回 Tuple2<Long, T>类型的Flux。
flatMap 将元素进行1对多的映射，可以是异步的方法调用(map只能是同步映射)
Flux#startWith 在元素序列开头添加元素
Flux#concatWithValues 在元素序列结尾添加元素
Flux#collectList 将Flux聚合成一个List
Flux#collectMap 将Flux聚合成一个Map
Flux#count 计算序列的大小
Flux#reduce 对相邻元素进行计算（归约）
Flux#take 取一个序列从头开始的n个元素
Flux#takeLast 取一个序列最后的的n个元素
Flux#skip 跳过n个元素
Flux#takeUntil 取元素直到满足条件（类似repeat until循环)
Flux#takeWhile 当满足条件时取元素（类似while循环)
## 观察Mono/Flux
这里的观察是指对序列进行观察且不会改变其中元素的状态
doOnNext 在序列向下游传送onNext时进行调用
doOnComplete 在序列向下游传送onComplete时进行调用
doOnError 在序列向下游传送onError时进行调用
doOnCancel 在Subscription.cancel时执行该回调（先于cancel信号向上游发送前执行）
doFirst 在序列开始前执行
doOnSubscribe 在Subscription开始后，它比doFirst执行要晚
doOnRequest 在Flux/Mono接收到（请求元素）request时调用（先于request信号向上游发送前执行）
doOnTerminate 在Flux/Mono结束(无论成功与否）执行（先于onComplete/onError信号向下游发送前执行）
doAfterTerminate 在Flux/Mono结束(无论成功与否)执行(后于onComplete/onError信号向下游发送前执行)
doOnEach 任何元素发送前皆会执行(onComplete/onError/onNext)
doFinally Flux/Mono因为任何原因（COMPLETE/ERROR/CANCEL)终结后皆会执行
log 记录内部日志（常用于调试）
