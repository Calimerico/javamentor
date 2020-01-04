### Retrying method call with Java

##### Problem
We have this class:
```
class DummyClass {
    public String method() throws IOException {
        //do something
    }
}

```

Of course, this method will return ```String``` sometimes and sometimes it will throw an ```Exception```.

If you receive ```Exception``` you would like to retry few more times
(maybe next time  in few seconds you will not receive exception). Number of retries is configurable parameter.
Of course, between those retries you would like to wait a little bit. Wait time is configurable parameter.
If you specified 10 retries(for example) and if even after 10 retries you receive ```Exception``` you would like 
to specify some default value or maybe just throw that exception. How would you do that?

I created helper ```Retrier``` class. Here is how I did it:

```
public class Retrier {

    public static <T> T retry(Callable<T> action) throws Exception {
        return retry(action, 5, 6000);
    }

    public static <T> T retry(Callable<T> action, T defaultValue) {
        return retry(action, 5, 6000, defaultValue);
    }

    public static <T> T retry(Callable<T> action, int numberOfRetries, int milisecondsBetweenRetries, T defaultValue) {
        try {
            return retry(action, numberOfRetries, milisecondsBetweenRetries);
        } catch (Exception e) {
            return defaultValue;
        }
    }

    public static <T> T retry(Callable<T> action, int numberOfRetries, int milisecondsBetweenRetries) throws Exception {
        try {
            return action.call();
        } catch (Exception e) {
            e.printStackTrace();
            //todo log that we are retrying
            if (numberOfRetries > 0) {
                try {
                    Thread.sleep(milisecondsBetweenRetries);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();//todo Thread.currentThread().interrupt(); what to do with this? https://dzone.com/articles/how-to-handle-the-interruptedexception
                }
                return retry(action, numberOfRetries - 1, milisecondsBetweenRetries);
            }
            throw e;
        }
    }
}
```
Here is how to use this helper:
```
    DummyClass dummyClass = new DummyClass();
    //5 retries and 500 milis between retries, empty String is default value
    Retrier.retry(() -> dummyClass.method(),5,500,"");
```
If you would like to just throw an Exception(not having default value) just omit that default value param.
Also, there is overloaded method where you can just pass action. This method will retry ```5``` times,
it will wait ```6``` seconds between retries and it will throw an ```Exception``` if you receive ```Exception```
on last retry.