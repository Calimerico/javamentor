
### How to improve performance of your spring boot tests a lot

Imagine you have one hundred spring boot test classes. When you start all of your tests
would you like to load application context for every class or would you like to reuse the same
application context? 
Since loading of application context take time(it can take much more than ten seconds)
you would like to reuse same application context.

Spring is trying to reuse context but there are some situations where context cannot be reused.

Simple way how to see how many contexts your tests started, is to count number of spring logos
in your console. Every time context start to load, big spring logo is shown in console.

If you introduce ```@TestConfiguration``` , you must load new context for that test class.
This have sense since when you ask for ```FooBean``` in your test,
 in test where you introduced ```@TestConfiguration```you will receive bean
from ```@TestConfiguration``` and in some other test you will receive some other bean. 
There is no way how to reuse same context.

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringTest {

    @Test
    //Some test
    
    @TestConfiguration
    public static class CustomConfiguration {
        @Bean
        public FooBean fooBean() {
            return new FooBean();
        }
    }
}
```
Almost the same thing happens when you introduce ```@MockBean```, context cannot be reused
for obvious reason.

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringTest {

    @MockBean
    private FooBean fooBean;
    
    @Test
    //Some test
}
```

If you annotate test class or method with ```@DirtiesContext```, for that class or method new 
context must be loaded. That is the purpose of that annotation. 
When people usually use this annotation? 
Well when they want to reload cache or change some property.
But the truth is that many times you don't need to reload whole context in order to restart cache.
Sometimes you can manually restart that cache by calling ```cacheManager.clearCache()``` or something like that.

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringTest {
    
    @Test
    @DirtiesContext
    //Some test
}
```
If you introduce ```@ActiveProfiles``` annotation one profile 
will be created for every combination of profiles. It have sense since test with ```dev```
profile will have different beans loaded from test that have ```prod``` profile active
```
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles(profiles = {"dev","prod"})
public class SpringTest {
    
    @Test
    //Some test
}
```

If you set some custom property like this:

```
@RunWith(SpringRunner.class)
@SpringBootTest(properties = "os=windows")
public class SpringTest {

   @Test
   ...
}
```
new context will be created for this class with custom property.

When you are loading context, you are loading all beans into context. 
@Controllers, @Repositories, etc... If all of your tests 
don't need controllers because you are writing just unit tests of domain classes or just 
repositories test you can just load one subset of beans.
Instead of ```@SpringBootTest``` you can use: ```@JsonTest```, ```@DataJpaTest```, ```WebMvcTest``` etc...
 
You can find full list of slices <a href="https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure">here</a>.
 
 #### Further reading
 
 There are more ways how to kill performance of your tests by not reusing context.
 
 You can learn more about this topic on this <a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#testcontext-ctx-management-caching">here</a>
 
 