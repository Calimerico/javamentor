### Decorator pattern explained

##### Table of Contents


[Explanation of the pattern](#decorator-pattern-explanation)  
[Example](#example)  
[Conclusion](#conclusion)  

<a name="decorator-pattern-explanation"/>

#### Explanation of the pattern

The main idea behind decorator pattern is to add some feature to the class in the runtime. Now, many of you might ask: "Why we need all those acrobations with extending class, delegation, etc. , can't we just add that feature inside the class?". Let's see what are the benefits of the decorator on the example.

<a name="example"/>

#### Example
Let's say this is your requirement: "You need to create some way to enable all `List` implementations to be immutable(immutable means once list is initialized, you cannot add or remove elements from the list)". Now you think something like this might work:

```
boolean isImmutable = true;
List<String> list = new ArrayList<>();
list.add("string1");
list.add("string2");
list.add("string3");
List<String> unmodifiableList = new ArrayList<>(isImmutable, list);
```

Well, it would work. But there are few disatvanges. First of all, you cannot change `ArrayList`(there is no constructor with this `boolean` immutable param and you are not Oracle developer who maintain Java). Even if you can, you need to implement this logic in every `List` implementation. Already sounds so bad, but there is even worse thing. If you add features like this(you might want to add synchronization feature inside `ArrayList` in the similar fashion) you will end up in a big mess very fast. You polluted your `ArrayList` with some weird logic for immutability. `ArrrayList` should not know anything about immutability.

Let's try something better. This might be a good idea:

```
List<String> list = new ArrayList<>();
list.add("string1");
list.add("string2");
list.add("string3");
List<String> unmodifiableList = new UnmodifiableList<>(list));
unmodifiableList.add("new string");// this should throw UnsupportedOperationException
```

If we pass `ArrayList` to the `UnmodifiableList` then `UnmodifiableList` should behave like `ArrayList` in all operations that are not mutating(mutating operations are `add`, `remove`, etc.). If we pass `LinkedList` then `UnmodifiableList` behaves like `LinkedList` etc. So, in some way, we wrapped(decorated) any `List` with new feature(new feature is whenever you call mutating operation `UnsupportedOperationException` is thrown). Let's see part of the implementation of the `UnmodifiableList`:

```
public class UnmodifiableList<E> implements List<E> {
    private List<E> list;

    public UnmodifiableList(List<E> list) {
        this.list = list;
    }
    
    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return list.contains(o);
    }


    @Override
    public E get(int index) {
        return list.get(index);
    }

    @Override
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }

    @Override
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }

    @Override
    public int indexOf(Object o) {
        return list.indexOf(o)
    }
    //bunch of other methods
}
```

As you can see, on mutating methods, you are throwing exception. In all other cases, you just delegate to the underlying list. Voila, you created a decorator! Now, in runtime you can just wrap your "normal" list with this `UnmodifiableList` and you got your feature. In similar fashion you can introduce syncronization feature!

The only sad part is that you reinvented the wheel. Java developers already created those two decorators(immutabile list and synchronized list). You can take a look at `UnmodifiableList` and `SynchronizedList` classes. You can call those Java wrappers like this:

```
Collections.unmodifiableList(new ArrayList<>());
Collections.synchronizedList(new ArrayList<>());
```
<a name="conclusion"/>


#### Conclusion

To summarize: If you want to add some feature to your class in runtime(and not to pollute your class with some random feature) you should use decorator.
