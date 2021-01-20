# Concurrency

## Executor service

From [baeldung blog](https://www.baeldung.com/java-executor-service-tutorial).

An `Executor Service` is a service to create tasks in an asynchronous mode. A pool of threads.

To create a pool:

```java
ExecutorService executorService = Executors.newFixedThreadPool(N);
```

Or we can set up any system like `Queues` to the factory to manage its pooling.

We can assign tasks and submit them:

```java
Collection<Callable<T>> callables = new ArrayList();

List<Future<T>> taskList = executorService.invoke(callables);
```

To retrieve the result:

```java
for (Future<T> task : taskList) {
    T result = task.get(TIME_TO_WAIT, TimeUnit.<TIME_UNIT>)
}
```

Some important notes:

- the `get` and `invoke` functions throw the exceptions `InterruptedException`, `CancellationException`, `TimeoutException` and `ExecutionException`.
- we can control the task management shutting down, or canceling the tasks.
