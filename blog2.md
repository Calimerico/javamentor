
### Are you sure that Java Set cannot contains duplicates?

Let's say we have this class:

```
public class Foo {
    private int number;

    public Foo(int number) {
        this.number = number;
    }
    
    public int getNumber() {
        return number;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Foo foo = (Foo) o;
        return number == foo.number;
    }

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }
    
    @Override
    public String toString() {
        return Integer.toString(number);
    }
}
```
As you can see, it is a simple class that has one field. We implemented hashCode and equals methods properly. Two instances are same if their number field is same. So far so good.

Now, take a look at this simple example.
```
public class Main {
    public static void main(String[] args) {
        Set<Foo> foos = new HashSet<>();
        Foo foo1 = new Foo(1);
        Foo foo2 = new Foo(2);
        Foo foo3 = new Foo(3);
        foos.add(foo1);
        foos.add(foo2);
        foos.add(foo3);
        
        //Until now, we created a Set and added 3 elements, nothing fancy
        
        foo1.setNumber(2);
        //now, after this change it seems like foo1 and foo2 are equals, so Set should contains only two elements?
        System.out.println(foo1.equals(foo2));
        System.out.println(foos);
        
    }
}
```
What do you think, what will be the output?
Actually, you will end up with two same elements in Set.
