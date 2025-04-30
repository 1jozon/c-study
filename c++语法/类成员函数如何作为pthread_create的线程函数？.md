# [类成员函数如何作为pthread_create的线程函数？](https://www.cnblogs.com/yanghh/p/12924858.html)

1. ### 参考链接

   [1]. [Linux 线程池的概念与实现](https://blog.csdn.net/qq_44443986/article/details/115441456)
   [2]. [【C语言】如何理解【void(*)(void)】](https://blog.csdn.net/Edidaughter/article/details/117194879)
   [3]. [关于typedef void (*sighandler_t)(int)的理解](https://blog.csdn.net/davidsky11/article/details/28278973)
   [4]. [C++类中this指针的理解](https://www.cnblogs.com/liushui-sky/p/5802981.html)
   [5]. [C++ 调用PTHREAD_CREATE函数时，传入类中的成员报错。解决方法。](https://www.cnblogs.com/ZQQH/p/8930193.html)
   [6]. [C++ pthread_create传递静态函数及其参数](https://blog.csdn.net/shitouwuhao/article/details/41823167)
   [7]. [对于void* 的理解](https://blog.csdn.net/Joshua_bu/article/details/80432217)

   ### 问题引入

   C++中线程传入类内成员函数时，需要将该成员函数定义为[静态成员函数](https://so.csdn.net/so/search?q=静态成员函数&spm=1001.2101.3001.7020)[1]）:

   ```cpp
   //threadpool.hpp
   #include <iostream>
   #include <cstdio>
   #include <queue>
   #include <stdlib.h>
   #include <pthread.h>
   using namespace std;
   
   typedef void (*handler_t)(int);
   #define MAX_THREAD 5
   //任务类
   class ThreadTask
   {
   public:
       ThreadTask()
       {
   
       }
       //将数据与处理方式打包在一起
       void setTask(int data, handler_t handler)
       {
           _data = data;
           _handler = handler;
       }
       //执行任务函数
       void run()
       {
           return _handler(_data);
       }
   private:
       int _data;//任务中处理的数据
       handler_t _handler;//处理任务方式
   };
   
   //线程池类
   class ThreadPool
   {
   public:
       ThreadPool(int thr_max = MAX_THREAD)
           :_thr_max(thr_max)
       {
           pthread_mutex_init(&_mutex, NULL);
           pthread_cond_init(&_cond, NULL);
           for (int i = 0; i < _thr_max; i++)
           {
               pthread_t tid;
               int ret = pthread_create(&tid, NULL, thr_start, this);
               if (ret != 0)
               {
                   printf("thread create error\n");
                   exit(-1);
               }
           }
       }
       ~ThreadPool()
       {
           pthread_mutex_destroy(&_mutex);
           pthread_cond_destroy(&_cond);
       }
       bool taskPush(ThreadTask &task)
       {
           pthread_mutex_lock(&_mutex);
           _queue.push(task);
           pthread_mutex_unlock(&_mutex);
           pthread_cond_signal(&_cond);
           return true;
       }
       //类的成员函数，有默认的隐藏参数this指针
       //置为static，没有this指针，
       static void *thr_start(void *arg)
       {
           ThreadPool *p = (ThreadPool*)arg;
           while (1)
           {
               pthread_mutex_lock(&p->_mutex);
               while (p->_queue.empty())
               {
                   pthread_cond_wait(&p->_cond, &p->_mutex);
               }
               ThreadTask task;
               task =p-> _queue.front();
               p->_queue.pop();
               pthread_mutex_unlock(&p->_mutex);
               task.run();//任务的处理要放在解锁之外
           }
           return NULL;
       }
   private:
       int _thr_max;//线程池中线程的最大数量
       queue<ThreadTask> _queue;//任务缓冲队列
       pthread_mutex_t _mutex; //保护队列操作的互斥量
       pthread_cond_t _cond; //实现从队列中获取结点的同步条件变量
   };
   
   12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576777879808182838485868788899091929394AI写代码
   ```

   上面代码中：

   ```cpp
   class T.. {
   int ret = pthread_create(&tid, NULL, thr_start, this);
    ...
   }
   1234AI写代码
   ```

   在类中thr_strat定义为:

   ```cpp
          //类的成员函数，有默认的隐藏参数this指针
      //置为static，没有this指针，
   static void *thr_start(void *arg)
       {
           ThreadPool *p = (ThreadPool*)arg;
           while (1)
           {
               pthread_mutex_lock(&p->_mutex);
               while (p->_queue.empty())
               {
                   pthread_cond_wait(&p->_cond, &p->_mutex);
               }
               ThreadTask task;
               task =p-> _queue.front();
               p->_queue.pop();
               pthread_mutex_unlock(&p->_mutex);
               task.run();//任务的处理要放在解锁之外
           }
           return NULL;
       }
   1234567891011121314151617181920AI写代码
   ```

   类的成员函数，有默认的隐藏参数[this指针](https://so.csdn.net/so/search?q=this指针&spm=1001.2101.3001.7020)，设为为static，没有this指针是什么意思？
   为何要将成员函数设为static？
   类中this的返回值是什么？
   错误出现在哪？
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff10aee4a1aa7cc0a3f98b79502643ec.png)
   运行报错，第三个参数类型不匹配：
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d60b10401661e3c1188c7bb31ee1ab20.png)

   ### pthread函数原型

   头文件

   ```
   #include <pthread.h>
   1AI写代码
   ```

   函数声明：

   ```
   int  pthread_create(pthread_t *tidp, const  pthread_attr_t *attr,
                   ( void *)(*start_rtn)( void *), void  *arg);
   12AI写代码
   ```

   若线程创建成功，则返回0。若线程创建失败，则返回出错编号；
   (对于`(void*) (*) (void*)` 指一个参数为void*，返回值为void*的函数指针[2][3] )
   **参数**
   第一个参数为指向线程 标识符的 指针。
   第二个参数用来设置线程属性，一般为NULL。
   第三个参数是线程运行函数的起始地址。
   最后一个参数是运行函数的参数。

   ### 解决

   C++中，我们知道在类中成员函数调用类内的成员变量或成员函数时，会隐式调用this指针[4][5],也就是说，上例中类成员函数加上this指针后，类型为`void* (Box::) (void*)`的成员函数指针，而pthread_create参数类型为`void* (*) (void*)`两个不同的类型，肯定不会匹配。

   既然错误出现在类中成员被隐式加上this，那么加上static变为静态成员函数以后，就不会加上this指针。

   静态成员函数不能访问类内的非静态成员，但只需要在pthread_create时将this指针传入，就可以使得静态成员函数访问类内非静态成员[5]。

   ### 传入成员函数参数

   上面如果需要给函数传入其他参数，可以使用数组的方式[6]

2. 

3. C++成员函数隐藏的this指针

  C++中类的普通成员函数都隐式包含一个指向当前对象的this指针，即：T *pThis，其中T为类类型。

  C++通过传递一个指向自身的指针给其成员函数从而实现程序函数可以访问C++的数据成员。这也可以理解为什么

  C++类的多个实例可以共享成员函数但是确有不同的数据成员。

 

2. 类成员函数如何作为pthread_create的线程函数

  在C++的类中，普通成员函数作为pthread_create的线程函数就会出现参数问题，因为隐含的this指针的存在。

  具体解决方法有如下几种：

  **a.** 将函数作为为类内静态成员函数，即使用static修饰。将this指针作为参数传递，以使该方法可以访问类的非静态方法或者是变量。

```
class` `MyClass``{``public``:``  ``static` `void``* ThreadFunc(``void``*);``  ``void` `Func()``  ``{``  ``}` `  ``bool` `StartThread()``  ``{``    ``int` `ret = pthread_create(&_pthreadId, NULL, MyClass::ThreadFunc, ``this``);``    ``if``(ret != 0)``    ``{``      ``return` `false``;``    ``}``    ``else``    ``{``      ``return` `true``;``    ``}``  ``}``private``:``  ``pthread_t _pthreadId;``};` `void``* MyClass::ThreadFunc(``void` `*arg)``{``  ``MyClass *thiz = ``static_cast``<MyClass *>(arg);``  ``thiz->Func();``  ``return` `NULL;``};
```

  **b.** 对成员函数进行强制转换，当作线程的回调函数。具体代码如下：

```
class` `MyClass``{``  ``/* 定义Func类型是一个指向函数的指针，``    ``该函数参数为void*，返回值为void*``   ``*/``  ``typedef` `void``* (*pFUNC)(``void` `*);` `public``:``  ``void` `Func()``  ``{``  ``}` `  ``bool` `StartThread()``  ``{``    ``pFUNC callback = ``reinterpret_cast``<pFUNC>(&Func);``    ``int` `ret = pthread_create(&_pthreadId, NULL, callback, NULL);``    ``if``(ret != 0)``    ``{``      ``return` `false``;``    ``}``    ``else``    ``{``      ``return` `true``;``    ``}``  ``}``private``:``  ``pthread_t _pthreadId;``};
```