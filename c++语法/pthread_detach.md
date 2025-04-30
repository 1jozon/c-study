`pthread_detach` 是 POSIX 线程（pthreads）库中的一个函数，用于将指定的线程设置为分离状态（detached state）。在分离状态下，线程结束时会自动释放其资源，而不需要显式地由其他线程通过 `pthread_join` 来回收。这简化了线程管理，特别是在创建大量短生命周期的线程时。

### 函数原型

	#include <pthread.h>
	
	int pthread_detach(pthread_t thread);

- **参数**:
  - `thread`: 要设置为分离状态的线程标识符。
- **返回值**:
  - 成功时返回 `0`。
  - 如果发生错误，则返回一个非零的错误代码。

### 分离状态与连接状态

POSIX 线程可以处于两种状态之一：连接（joinable）或分离（detached）。

- **连接状态**：默认情况下，线程是连接状态的。这意味着你需要调用 `pthread_join` 来等待该线程终止，并回收其资源。如果一个线程处于连接状态且没有被 `pthread_join` 回收，那么它的资源不会被立即释放，可能会导致资源泄漏。
  
- **分离状态**：一旦线程被设置为分离状态，它就不能再被其他线程通过 `pthread_join` 回收。当分离状态的线程结束执行时，系统会自动回收它的所有资源。

### 使用场景

- **短生命周期的线程**：如果你创建了许多短生命周期的线程，并且不关心它们何时结束，使用 `pthread_detach` 可以简化线程管理。
- **减少资源泄漏的风险**：对于一些你确定不需要等待其结束的线程，尽早将其设置为分离状态可以避免潜在的资源泄漏问题。

### 示例代码

以下是一个简单的例子，展示了如何使用 `pthread_create` 创建一个线程，并使用 `pthread_detach` 将其设置为分离状态：

	#include <pthread.h>
	#include <stdio.h>
	#include <stdlib.h>
	#include <unistd.h>
	
	void* thread_function(void* arg) {
	    int thread_id = *(int*)arg;
	    printf("Thread %d: Running...\n", thread_id);
	    sleep(2); // 模拟工作
	    printf("Thread %d: Finished.\n", thread_id);
	    pthread_exit(NULL);
	}
	
	int main() {
	    pthread_t thread;
	    int thread_id = 1;
	
	    if (pthread_create(&thread, NULL, thread_function, &thread_id) != 0) {
	        perror("pthread_create failed");
	        return EXIT_FAILURE;
	    }
	
	    // 将线程设置为分离状态
	    if (pthread_detach(thread) != 0) {
	        perror("pthread_detach failed");
	        return EXIT_FAILURE;
	    }
	
	    printf("Main thread: Detached thread %d\n", thread_id);
	
	    // 主线程继续做其他事情，无需等待子线程结束
	    sleep(5);
	
	    return EXIT_SUCCESS;
	}


在这个示例中：
- 我们创建了一个新线程并立即将其设置为分离状态。
- 子线程完成后，其资源会被自动回收，主线程不需要调用 `pthread_join` 来等待子线程结束。

### 注意事项

- **不能重复回收**：一旦线程被设置为分离状态，就不能再通过 `pthread_join` 回收。尝试这样做会导致错误。
- **线程属性设置**：除了使用 `pthread_detach` 动态地将线程设置为分离状态之外，还可以在线程创建时通过设置线程属性来预先定义线程是否为分离状态。例如：


		  pthread_attr_t attr;
		  pthread_attr_init(&attr);
		  pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
		  pthread_create(&thread, &attr, thread_function, &thread_id);
		  pthread_attr_destroy(&attr);


  这种方法可以在创建线程时就指定线程为分离状态，从而省去了后续调用 `pthread_detach` 的步骤。

通过合理使用 `pthread_detach` 和线程属性，可以根据具体的应用需求灵活管理线程的生命周期和资源回收。