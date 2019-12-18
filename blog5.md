
### Guide to improve code of beginners

I really like to help people and often beginners ask me: 
"Hey I just started programming. Can you take a look at my code and tell me where I can improve?"

I want to sum up things that I usually  correct in their code. Have in mind that this advices
are meant for almost total beginners. Anyone who done programing more than six months for money
, will not make this mistakes.

##### Write code on english language
You should love and speak your language but when it comes to programming, english should be used.
##### Use build tool
I saw many times that beginners commit lib, dist, target etc. folders to the git. 
Don't do that. There are plenty of choices when it comes to build tools like Maven, Gradle, Ant etc. 
I suggest you Maven for the start because most of the tutorials use maven as build tool. When you learn Maven
it will be easier for you to start following tutorials because every non trivial tutorial include some dependency and use maven.
##### Use source control(and use it properly)
Zipping your project and sending like that is not a good idea. It is much better to use source control like Git, SVN, TFS etc.
Be careful what you commit. Don't commit ```.idea``` files, ```.settings``` files, ```.metadata``` etc.
Your commit messages should have sense at least. Commit messages like "I FIXED IT" are not good(although we all do write this kind of messages from time to time :) )

##### Close your resources. Use try with resources block
Consider this code:
```
StringBuilder sb = new StringBuilder();
        try {
            BufferedReader br = Files.newBufferedReader(Paths.get("filename.txt"));
            // read line by line
            String line;
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }

        } catch (IOException e) {
            //handle exception properly
        }
```
You opened your reader but your forgot to call ```br.close()``` so your reader will stay open forever.
in order not to forgot to close your resources it is better to use ```try with resources``` block:

```
StringBuilder sb = new StringBuilder();
        try (BufferedReader br = Files.newBufferedReader(Paths.get("filename.txt"))) {

            // read line by line
            String line;
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }

        } catch (IOException e) {
            //handle exception properly
        }
```
this way, reader will be closed after try block.
##### Handle exceptions properly
Don't just print stack trace in catch block. You should properly handle your exception. 
Now, there is no silver bullet solution how to do that, but you can google a little bit what may be the good solution
for your case. Also, take a look at this retrier post, it is interesting when it comes to exception handling.
##### Try to make fields private and final whenever you can
##### Try to follow SOLID principles
I wrote article that cover how not to violate those principles
##### Good IDE can help you a lot
I personally use Intellij Idea and I would suggest you buying this IDE because it can help you a lot.
When you open your code with Intellij Idea, you will probably see plenty of gray and yellow code. That way, Intellij
is warning you that even if code compile, something is not fine with it. Take a look at those things, how you can fix
them.
##### Your fields and variables should have proper name
Don't name fields and variables ```a```, ```b```, ```field```, ```Map map = new HashMap()```.
##### Class name start with capital letter
##### Don't use String or int where enum should be used

```
public class Foo {
    private int status;//0 = inactive, 1 = active, 2 = suspended, 3 = new
}
```
Better way is to use enum:
```
public class Foo {
    private Status status;
}

public enum Status {
    INACTIVE,		
    ACTIVE,
    SUSPENDED,
    NEW
}
```
##### Don't forget default part of switch command
```
switch(status) {
    case ACTIVE:
        //do domething
    case INACTIVE:
        //do domething else
    default:
        // Don't forget this part
}
```

