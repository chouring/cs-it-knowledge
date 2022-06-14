# 锁
    锁是一种进程同步机制，有很多种类。
## 互斥锁
    未获得锁时，线程会进入休眠，让出CPU，mutex锁复杂，时间长。常用于对复杂操作的加锁，比如红黑树插入节点。
```cpp
pthread_mutex_t mutex;
```

## 自旋锁
    未获得锁时，线程会进入僵持等待，占据CPU，spinlock简单，时间短。常用于对简单操作的加锁，比如链表插入节点。
```cpp
pthread_spin_t spinlock;
```

## 原子操作
    将操作原子化，这需要深入到体系结构，需要CPU指令支持。
    有xaddl, CAS指令
```cpp
// 基于原子操作机制，我们可以实现一些无锁结构

// 无锁队列
```
     


## 实例
    一份进程同步进制的代码实例，用于参考。

```cpp
// case 10个线程给count++,主线程打印count值
const int THREAD_COUNT = 10;
pthread_mutex_t mutex;  // 互斥锁
pthread_spinlock_t spinlock; //自旋锁


int inc(int* value, int add) {
    int old;
    
    __asm__ volatile(
        "lock";xaddl %2, %1;"
        :"=a"(old)
        :"m"(value), "a"(add)
        :"cc", "memory"
    );
    return old;
}
 
void* thread_entry(void* arg) {
    int* pcount = (int*)arg;
    int i = 0;
    while (i++ < 100000) {
#if 0 // 不加任何同步
        (*pcount)++; 

#elif 0 // 加互斥锁
        pthread_mutex_lock(&mutex);
        (*pcount)++; 
        pthread_mutex_unlock(&mutex);

#elif 0 // 加自旋锁 
        pthread_spin_lock(&spinlock);
        (*pcount)++; 
        pthread_spin_unlock(&spinlock);

#else // 原子操作
        inc(pcount, 1);

#endif
         usleep(1);
    }
}

int main() {
    pthread_t thread_id[THREAD_COUNT] = {0};
    int count = 0;
    
    pthread_mutex_init(&mutex, NULL);
    pthread_spin_init(&spinlock, PTHREAD_PROCESS_SHARED);
    
    for (int i = 0; i < THREAD_COUNT; i++) {
        pthread_create(&thread_id[i], NULL, thread_entry, &count);
    }
    for (int i = 0; i < 100; i++) {
        printf("count = %d\n", count);
        sleep(1);
    }
}
```
    
