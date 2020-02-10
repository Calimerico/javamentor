### Compare performance of AtomicLong and LongAdder classes

`LongAdder` class is added in Java 8. Official [Java docs](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/LongAdder.html)
say: `Under high contention, expected throughput of this class compared to AtomicLong is significantly higher, at the expense of higher space consumption.`
 
Let's check that. I will create few threads and every thread
will have task to increment number 10000000 times. Very simple code:

```
public class Main {


    public static void main(String[] args) {
        //warming up before benchamrk test
        List<Integer> numbers = new ArrayList<>();
        IntStream.range(0,1000000).forEach(numbers::add);
        System.out.println(numbers.get(50000));

        //we would like to compare those 2 classes
        AtomicLong atomicLong = new AtomicLong(0);
        LongAdder longAdder = new LongAdder();

        //create threads that will increment atomicLong 10 milion times, execute those threads and measure result
        List<Thread> threads = IntStream.range(0, 10).mapToObj(i -> new Thread(() -> {
            IntStream.range(0, 10000000).forEach(i1 -> atomicLong.incrementAndGet());
        })).collect(Collectors.toList());
        long start = System.currentTimeMillis();
        threads.forEach(Thread::start);
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end= System.currentTimeMillis();
        System.out.println(end - start);

        //do the same thing as with atomicLong just this time with Long Adder
        List<Thread> threads2 = IntStream.range(0, 10).mapToObj(i -> new Thread(() -> {
            IntStream.range(0, 10000000).forEach(i1 -> longAdder.increment());
        })).collect(Collectors.toList());
        long start1 = System.currentTimeMillis();
        threads2.forEach(Thread::start);
        threads2.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end1 = System.currentTimeMillis();
        System.out.println(end1 - start1);
    }
}
```

I executed this program 3 times and on my machine 
execution time when incrementing `AtomicLong` is 2119,2188,2180 and incrementing `LongAdder`
is 369,435,455. Thus, we can conclude that `LongAdder` class is much faster and should be favoured instead of
`AtomicLong` if we are fine with more space consumption.