## redis相关知识
redis 4.x开始引入多线程的功能和内容 
redis的性能瓶颈主要在于内存或者网络带宽 而不在于cpu。所以单线程就可以了 大key的异步删除。
flushall flushall
2.unix系统才支持epoll
3. io多路复用 简单来说就是通过检测文件的读写事件通知线程执行相关操作。保证Redis的非阻塞IO能够顺利执行完成的机制
4. 多路是指多个socket连接 复用指的是复用一个线程
5. redis6.0将读取socket 解析请求和写入socket 分配给多个io线程 主线程负责执行操作
6. 缓存击穿 热点key失效 缓存穿透 不存在的key
7. doc.redisfans.com
8. 9大类型 
9. String  
9. Hash
10. List
11. Set
12. sortedSet
13. Bitmap位图
14. HyperLogLog 统计
15. GEO地理
16. Stream 主要用于redisMq pub subsuribe