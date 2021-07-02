### Why we use private mutex?

You may saw something like this:

```
public class ValueHolder {
    private int a;
    private final Object mutex = new Object();

    public int getA() {
        return a;
    }

    public void increment() {
        synchronized (mutex) {
            a++;
        }
    }
}
```
And you start wondering: "Hey, why we use that mutex object, can't we just use `this` and everything would work?"

Let's see this in practical example. Multiple threads will update state of our `ValueHolder`.

Thread that will increment our counter 1000 times:

```
public class UpdaterThread implements Runnable {
    
    private ValueHolder valueHolder;

    public UpdaterThread(ValueHolder valueHolder) {
        this.valueHolder = valueHolder;
    }

    @Override
    public void run() {
        for (int i = 0 ; i < 1000; i++) {
            valueHolder.increment();
        }
    }
}
```
Starting 10 threads to update our counter:
```
public class Main {
    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        ValueHolder valueHolder = new ValueHolder();

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new UpdaterThread(valueHolder));
            thread.start();
            threads.add(thread);
        }
    }

}
```
In this case, both `mutex` and `this` works fine as a lock. But here is where the problem lies:
```
public class EvilThread implements Runnable {

    private final ValueHolder valueHolder;

    public EvilThread(ValueHolder valueHolder) {
        this.valueHolder = valueHolder;
    }


    @Override
    public void run() {
        try {
            synchronized (valueHolder) {
                Thread.sleep(20000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
Start evil thread and then incrementing threads:
```
public class Main {
    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        ValueHolder valueHolder = new ValueHolder();
        
        Thread evilThread = new Thread(new EvilThread(valueHolder));
        evilThread.start();

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new UpdaterThread(valueHolder));
            thread.start();
            threads.add(thread);
        }
    }

}
```
`EvilThread` is holding `valueHolder` lock. Now, if inside `UpdaterThread` we use `this` as lock then we are using the same lock as `EvilThread` 
and we will have to wait. If we use `mutex`, then we don't wait for `EvilThread` since lock is not the same.

#### Conclusion

We have more control if we use private mutex. Also, we can see all the places where lock is used inside the class. If we use `this` as lock,
we would need to search whole project to see where that lock is used. 
