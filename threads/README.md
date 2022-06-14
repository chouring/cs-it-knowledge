# 多线程编程
    多线程编程可以用应用层的library，也可以选择操作系统提供的interface
## 前置
需要了解[锁](https://github.com/chouring/cs-it-knowledge/tree/main/lock)
## Linux多线程编程
提供一个linux pthread + 各种锁的例子，这个例子在[这篇文章](https://github.com/chouring/cs-it-knowledge/tree/main/lock)中已经介绍过。
```cpp
// 主要接口
pthread_create();
pthread_join();
pthread_cancel();
pthread_exit();
```

```cpp
// case 10个线程给count++,主线程打印count值
const int THREAD_COUNT = 10;
pthread_mutex_t mutex;  // 互斥锁
pthread_spinlock_t spinlock; //自旋锁

 
void* thread_entry(void* arg) {
    int* pcount = (int*)arg;
    int i = 0;
    while (i++ < 100000) {
#if 0
        (*pcount)++; 
#elif 0
        pthread_mutex_lock(&mutex);
        (*pcount)++; 
        pthread_mutex_unlock(&mutex);
#else
        pthread_spin_lock(&spinlock);
        (*pcount)++; 
        pthread_spin_unlock(&spinlock);
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


## C++多线程编程
