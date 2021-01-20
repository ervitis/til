# Signal handler

### Handling signal

We can handle some signals like `Ctrl+C` signal to clean our program from resources and terminating them gracefully.

For example:

```java
private volatile boolean keep = true;

public class RunWhenShuttingDown extends Thread {
    public void run() {
        System.out.println("Control-C caught. Shutting down...");
        keep = false;
    }
}

void runApp() {
    Runtime.getRuntime().addShutdownHook(new RunWhenShuttingDown());

    while (keep) {
        ...
        do something
        Thread.sleep(100000);
    }

    resources.clean();
}

public static void main(String[] args) {
    new Main().runApp();
}
```

Registering the event in the hook `addShutdownHook` will solve the problem.