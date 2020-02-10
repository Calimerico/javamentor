### How to shoot yourself in your own leg with improper publication of object

First, we need to define what is improper publication. If in thread A you are creating an object X
and in thread B you have a reference to object X before constructor of X finished his work,
 that is improper publication. Thread B can work with object X while it is not still constructed properly.
 
Take a look at this example from [Java Concurrency in Practice](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)
book.

```
public class Holder {
     private int n;
         public Holder(int n) { this.n = n; }
         
         public void assertSanity() {
            if (n != n)
                throw new AssertionError("This statement is false.");
     }
} 
```
Believe it or not, you can receive ```AssertionError```!
 
 ##### How AssertionError is possible?
 
If one thread is creating `Holder` instance and another thread have reference to that holder
instance while it is not fully created (example with thread A,B and object X)
 than in reading thread can happen when you call `assertSanity` first time you read `n`
 (reading `n` first time is when we are reading `n` on left side of the `!=` sign) is 0 and 
 in the mean time thread that is creating instance changed value of `n` to `5` so second time
 in reading thread(`n` on the right side of the `!=` sign) `n` is `5`. Boom, you got error.
 
 ##### How to solve this problem?
 
 There are 4 ways:
 
 - Initializing an object reference from a static initializer
 - Storing a reference to it into a  volatile field or AtomicReference
 - Storing a reference to it into a  final field of a properly constructed object
 - Storing a reference to it into a field that is properly guarded by a lock.
 
If you would like to learn more about how to solve this problem, take a look at this [article](https://shipilev.net/blog/2014/safe-public-construction/). 
I mentioned this article here but I thought that improper publication is too
important so I decided to write this article about this problem.

##### Further reading

Like always, I strongly recommend [Java Concurrency in Practice](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601) book.