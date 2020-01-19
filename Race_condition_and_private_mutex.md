### How many problems you can spot in this simple code?

We are trying to create program that  run ten threads and every thread add thousand numbers to the list.
Here is our client code where we spawn threads:

```
public class Main {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new FooThread(list));
            thread.start();
        }
        System.out.println(list);
    }
}
```
and our thread that add numbers to the list:
```
public class FooThread implements Runnable {

    private final List<Integer> numbers;

    public FooThread(List<Integer> numbers) {
        this.numbers = numbers;
    }

    @Override
    public void run() {
        for ( int i = 0 ; i < 1000 ; i++ ) {
            numbers.add(i);
        }
    }
}
```
Pause here and try to find as much as possible code smells and bugs and think how would you resolve them.

Now, let's resolve those problems. :) If you run this code you will end up with this exception:
```
Exception in thread "main" java.util.ConcurrentModificationException
	at java.base/java.util.ArrayList$Itr.checkForComodification(Unknown Source)
	at java.base/java.util.ArrayList$Itr.next(Unknown Source)
	at java.base/java.util.AbstractCollection.toString(Unknown Source)
	at java.base/java.lang.String.valueOf(Unknown Source)
	at java.base/java.io.PrintStream.println(Unknown Source)
	at com.Main.main(Main.java:39)
```
This exception is thrown when you try to modify list (in this case add element into the list)
 while iterating through that list. You may think "but I am not iterating through my list" 
 but actually ```System.out.println``` called ```toString``` method of the ```list``` and if you take a look
 at the implementation of ```toString``` method in the ```AbstractCollection``` class you will notice there is iterating.

Ok, let's give some time to that adding threads to finish their work so we can print elements when list is populated. 
You may be tempted to call ```Thread.sleep(2000)``` method but this is wrong way. You cannot be sure that threads will finish their work in two seconds. Good way to solve this problem is calling ```thread.join()```. Now, your main thread will not proceed until thread where ```join``` method is called is not done. 

#### IMPORTANT: Don't do this

```
public class Main {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new FooThread(list));
            thread.start();
            thread.join();
        }
        System.out.println(list);
    }
}
```
This way, you essentially created single threaded application. When you call ```join``` method in first iteration, you cannot proceed to the second iteration while you thread from first iteration is not over. This is the way to go:

```
public class Main {
    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        List<Integer> list = new ArrayList<>();

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new FooThread(list), "Thread " + i);
            thread.start();
            threads.add(thread);
        }
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(list.size());
    }
}
```
Now, you are not receiving ```ConcurrentModificationException``` but you can see that your list contains different number of elements every time you run your program. Your problem is that your program contains **race condition**.

Problem is that more than one thread is trying to write to the same memory location at the same time(in our case it is ```list``` shared variable) so basically one thread override change of the other one.
 There are two ways how to prevent this. Introduce ```synchronized``` block or switch our ```list``` from ```ArrayList``` to some list implementation that is ```synchronized```.

First solution(introduce ```synchronized``` block):
```
public class FooThread implements Runnable {

    private final List<Integer> numbers;

    public FooThread(List<Integer> numbers) {
        this.numbers = numbers;
    }

    @Override
    public void run() {
        for ( int i = 0 ; i < 1000 ; i++ ) {
            synchronized (numbers) {
                numbers.add(i);
            }
        }
    }
}
```
Second solution(change implementation of the ```list```):

```
public class FooThread implements Runnable {

    private final List<Integer> numbers;

    public FooThread(List<Integer> numbers) {
        this.numbers = numbers;
    }

    @Override
    public void run() {
        for ( int i = 0 ; i < 1000 ; i++ ) {
            numbers.add(i);
        }
    }
}
```
**Note**: You don't need ```synchronized``` block in second solution. 
```
public class Main {
    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        List<Integer> list = Collections.synchronizedList(new ArrayList<>());

        for(int i = 0 ; i < 10 ; i++ ) {
            Thread thread = new Thread(new FooThread(list), "Thread " + i);
            thread.start();
            threads.add(thread);
        }
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(list.size());
    }
}
```
You can also try to add number into list just if number is not present. ```synchronizedList``` 
don't work in that case, you have to use ```synchronized``` block:
```
            synchronized (numbers) {
                if(!numbers.contains(i)) {
                    numbers.add(i);
                }
            }
```
**Bonus**: Since you learned about ```synchronized``` block let me give you simple example how things can go wrong with it.

Let's suppose we have class that holds one number and we can increment that number with method ```increment```:

```
public class ValueHolder {
    private int a;

    public int getA() {
        return a;
    }

    public void increment() {
        synchronized (this) {
            a++;
        }
    }
}
```

We create ten threads that will update number thousand times:

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
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(valueHolder.getA());
    }

}
```
UpdaterThread class: 
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
And everything is "working". But here is the problem.

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
This thread is just holding lock on ValueHolder so none else can acquire that lock. If we start that evil thread before we start other threads for incrementing, we are in trouble:

```
Thread evilThread = new Thread(new EvilThread(valueHolder));
        evilThread.start();
```
Resolving this is simple. Change ```ValueHolder``` class:

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
Now, we are locking on private lock that no one else is having access to so evil thread cannot acquire it.

#### Further reading 
If this article was interesting for you, I would suggest you learn more about threads in three resources:

- <a href="http://tutorials.jenkov.com/java-concurrency/index.html">Java Concurrency and Multithreading Tutorial</a>
- <a href="http://tutorials.jenkov.com/java-util-concurrent/index.html">Java Concurrency Utilities</a>
- <a href="https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601">Java Concurrency in Practice</a>

I was in your shoes and I know how hard is to understand multithreading concepts without help of someone more experienced.
If you need help understanding those concepts, I can help you and be your mentor. More information about mentoring can be found <a href="http://tutorials.jenkov.com/java-concurrency/index.html">here</a>.

Cheers. :)

