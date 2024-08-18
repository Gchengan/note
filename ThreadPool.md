## 线程池

[(93条消息) 面试突击：线程池有几种创建方式？推荐使用哪种？_Java面试那些事儿的博客-CSDN博客_线程池一般用那种](https://blog.csdn.net/HongYu012/article/details/123331122?ops_request_misc=&request_id=&biz_id=102&utm_term=推荐怎样创建线程池&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-3-123331122.142^v70^wechat,201^v4^add_ask&spm=1018.2226.3001.4187)

1.Executors固定大小线程池

```java
public class Thread_02 {
    public static void main(String[] args) {
        //第一个参数线程是核心线程=最大线程数，第二个参数是取线程名字
        ExecutorService pool= Executors.newFixedThreadPool(2,new ThreadFactory(){
private AtomicInteger t=new AtomicInteger();
            @Override
            public Thread newThread(Runnable r) {
                return new Thread(r,"pool_t"+t.getAndIncrement());
            }
        });
        pool.execute(()->{
            System.out.println(Thread.currentThread().getName());
        });
        pool.execute(()->{
            System.out.println(Thread.currentThread().getName());
        });
        pool.execute(()->{
            System.out.println(Thread.currentThread().getName());
        });

    }
}

    执行结果:
    pool_t0
    pool_t1
    pool_t0
```