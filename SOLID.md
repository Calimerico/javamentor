### SOLID principles - Explanation and real world example

#### Single Responsibility principle - one class should have one responsibility

Let's say you have to create program that goes to your favorite website for concerts and write information about
those concerts into excel file. You would like to collect information only about future concerts.
 If concert is in the past, you would not like to scrap that concert.

I used `jsoup` and `apache poi` for scraping from website and writing into excel file but that part is really
irrelevant.

Now, this is our first try:

```
public class Scraper {
    public void scrapConcerts() throws IOException {
        List<String> urlList = new ArrayList<>();
        urlList.add("https://somewebsite.com/concert1");
        urlList.add("https://somewebsite.com/concert2");
        urlList.add("https://somewebsite.com/concert3");
        urlList.add("https://somewebsite.com/concert4");
        for ( int i = 0; i < urlList.size(); i++) {
            //connect to the url
            Document document = Jsoup.connect(urlList.get(i)).get();
            //scrap concert date
            LocalDateTime concertDate = LocalDateTime.parse(document.select(".description [itemProp=startDate]").attr("content"), DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX"));
            //validate date -> if date is in the past, we should skip that concert
            if (concertDate.isBefore(LocalDateTime.now())) {
                continue;
            }
            //at the end, write that concert into our excel file
            XSSFWorkbook workbook = new XSSFWorkbook();
            XSSFSheet sheet = workbook.createSheet("Concerts");
            sheet.createRow(i).createCell(1).setCellValue(concertDate);
            try (FileOutputStream outputStream = new FileOutputStream("Concerts.xlsx")) {
                workbook.write(outputStream);
            }
        }
    }
}
```

Here, we broke **Single Responsibility principle** and this is very common mistake. 
Our scraper class have three responsibilities: 
- connect to the website, parse html and extract concert date somehow
- validate data(see if concert is in the past)
- write data into excel file

If we write unit test for this class and test fails, we don't know what is wrong.
 Is it scraping, validation or writing into excel?
 Also, if we want to change validation, we will change this class but we cannot be sure that by changing validation we
 did not broke writing into file.
 
 Now, let's split this into three classes:
 
 ```
 public class Concert {
     private UUID id;
     private String name;
     private LocalDateTime date;
 
     public Concert(UUID id, String name, LocalDateTime date) {
         this.id = id;
         this.name = name;
         this.date = date;
     }
     //getters and setters
 }
 ```
 
Responsibility of `Concert` class is to hold data about concert.
 
 ```
 public interface Validator {
    boolean validate(Concert concert);
 }
 
 public class ValidatorImpl implements Validator {
     //As you can see, this line can be simplified even more but I didn't wanted to confuse beginners with one liners :)
     @Override
     public boolean validate(Concert concert) {
         if (concert.getDate().isBefore(LocalDateTime.now())) {
             return false;
         } 
         return true;
     }
 } 
 ```
 
 Responsibility of `Validator` is only to validate concert and tell us if concert is valid or not.
 As you can see we program to an interface so we can easily substitute our implementation later.
 
 
 ```
 public interface Writer {
      void write(List<Concert> concerts) throws IOException;
  }
   
  public class ExcelWriter implements Writer {
      @Override
      public void write(List<Concert> concerts) throws IOException {
          XSSFWorkbook workbook = new XSSFWorkbook();
          XSSFSheet sheet = workbook.createSheet("Concerts");
          for ( int i = 0; i < concerts.size(); i++) {
              sheet.createRow(i).createCell(1).setCellValue(concerts.get(i).getDate());
              try (FileOutputStream outputStream = new FileOutputStream("Concerts.xlsx")) {
                  workbook.write(outputStream);
              } 
          }
          
      }
  }
 ``` 
 
 Responsibility of `Writer` is to write our concerts somewhere. Our Excel implementation
 write our concerts to excel file.
 
 ```
 public interface HtmlParser {
      List<Concert> parse(List<String> urlList) throws IOException;
  }
   
 public class JsoupHtmlParser implements HtmlParser {
   @Override
   public List<Concert> scrap(List<String> urlList) throws IOException {
       List<Concert> concerts = new ArrayList<>();
       for ( int i = 0; i < urlList.size(); i++) {
           Document document = Jsoup.connect(urlList.get(i)).get();
           LocalDateTime concertDate = LocalDateTime.parse(document.select(".description [itemProp=startDate]").attr("content"), DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX"));
           //I wrote some hardcoded values for id and name since those are not relevant for our example
           concerts.add(new Concert(UUID.randomUUID(), "Concert name", concertDate));
       }
       return concerts;
   }
 }
 ```
 
 And finally responsibility of HtmlParser is just to parse concerts(it doesn't care about validation or writing into the file)
 
 Now our program looks much better:
 
 ```
 public class Scraper {
 
     private final Validator validator;
     private final Writer writer;
     private final HtmlParser parser;
 
     public Scraper() {
         this.validator = new ValidatorImpl();
         this.writer = new ExcelWriter();
         this.parser = new JsoupHtmlParser();
     }
 
     public void scrapConcerts() throws IOException {
         List<String> urlList = new ArrayList<>();
         urlList.add("https://somewebsite.com/concert1");
         urlList.add("https://somewebsite.com/concert2");
         urlList.add("https://somewebsite.com/concert3");
         urlList.add("https://somewebsite.com/concert4");
         for ( int i = 0; i < urlList.size(); i++) {
             List<Concert> concerts = parser.parse(urlList);
             List<Concert> validConcerts = concerts
                     .stream()
                     .filter(validator::validate)
                     .collect(Collectors.toList());
             writer.write(validConcerts);
         }
     }
 }
 ```
 
 If we want to test if validation is working we can test `Validator` class and be sure 
 that we will not break anything else. If we want to change the way how data is written into 
 the excel file we just simply have to change `ExcelWriter` class and we can be sure that everything 
 else is working properly.
 
 We resolved the problem but we have new one. Can you spot it?
 
 #### Dependency inversion principle - class should not care about instantiating it's dependencies
 
 We are violating **Dependency inversion principle**. Imagine that instead of writing into
 excel file we would like to save our concerts as json file and that we see that we don't like
 jsoup library and instead we would like to use some other library for connecting to the website
 and parsing concerts.
 
 We will create those two classes:
 
 ```
 public class JsonWriter implements Writer {
     @Override
     public void write(List<Concert> concerts) throws IOException {
         //write to json somehow
     }
 }
 
 public class NewHtmlParser implements HtmlParser {
     @Override
     public List<Concert> parse(List<String> urlList) throws IOException {
         List<Concert> concerts = new ArrayList<>();
         //use some other library to parse concerts
         return concerts;
     }
 }
 ```
 
 But our Scraper class can only work with `JsoupHtmlParser` and `ExcelWriter`.
 When we create out `Scraper` like this:
 
 ```
 Scraper scraper = new Scraper();
 ```
 
 we cannot specify which parser or writer we would like to use. If we want to have flexible Scraper we should do this:
 
 ```
 public class Scraper {
 
     private final Validator validator;
     private final Writer writer;
     private final HtmlParser parser;
 
     public Scraper(Validator validator, Writer writer, HtmlParser parser) {
         this.validator = validator;
         this.writer = writer;
         this.parser = parser;
     }
 
     public void scrapEvents() throws IOException {
         List<String> urlList = new ArrayList<>();
         urlList.add("https://somewebsite.com/concert1");
         urlList.add("https://somewebsite.com/concert2");
         urlList.add("https://somewebsite.com/concert3");
         urlList.add("https://somewebsite.com/concert4");
         for ( int i = 0; i < urlList.size(); i++) {
             List<Concert> concerts = parser.parse(urlList);
             List<Concert> validConcerts = concerts
                     .stream()
                     .filter(validator::validate)
                     .collect(Collectors.toList());
             writer.write(validConcerts);
         }
     }
 }
 ```
 
 Now we can specify new parser and writer when we create scraper:
 
 ``` Scraper scraper = new Scraper(new ValidatorImpl(), new JsonWriter(), new NewHtmlParser());```
 `Scraper` does not care about it's own dependencies now, it's client class now who has control.
  Client class controlls which implementation of `Validator`, `Writer` and `Parser` will be used.
  
  #### Liskov substitution principle - if class A extends class B, we should be able to replace B with A without breaking our app
  
  Let's first see almost ideal example of Liskov principle violation(taken from [here](https://stackoverflow.com/questions/22050848/do-collections-unmodifiablexxx-methods-violate-lsp))

```
class SomeClass{
      public List<Integer> list(){
           return new ArrayList<Integer>();
      }
}
```
and
```
class SomeClass{
     public List<Integer> list(){
           return Collections.unmodifiableList(new ArrayList<Integer>());
     }
}
```
Now, it seems like our clients will have surprising behaviours if we return then unmodifiable version of the list. And this would really be great example of violation of Liskov principle. Depending on the class implementation, class can have surprising behavior, it throws some unexpected exception. Why this is not the violation of the principle? Take a close look at `add` method signature and pay attention that actually it sais it can throw `UnsupportedOperationException`:

```
/**
     * Ensures that this collection contains the specified element (optional
     * operation).  Returns {@code true} if this collection changed as a
     * result of the call.  (Returns {@code false} if this collection does
     * not permit duplicates and already contains the specified element.)<p>
     *
     * Collections that support this operation may place limitations on what
     * elements may be added to this collection.  In particular, some
     * collections will refuse to add {@code null} elements, and others will
     * impose restrictions on the type of elements that may be added.
     * Collection classes should clearly specify in their documentation any
     * restrictions on what elements may be added.<p>
     *
     * If a collection refuses to add a particular element for any reason
     * other than that it already contains the element, it <i>must</i> throw
     * an exception (rather than returning {@code false}).  This preserves
     * the invariant that a collection always contains the specified element
     * after this call returns.
     *
     * @param e element whose presence in this collection is to be ensured
     * @return {@code true} if this collection changed as a result of the
     *         call
     * @throws UnsupportedOperationException if the {@code add} operation
     *         is not supported by this collection
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this collection
     * @throws NullPointerException if the specified element is null and this
     *         collection does not permit null elements
     * @throws IllegalArgumentException if some property of the element
     *         prevents it from being added to this collection
     * @throws IllegalStateException if the element cannot be added at this
     *         time due to insertion restrictions
     */
    boolean add(E e);
```
So, unmodifiable collection is not throwing some random unexpected Exception, it is stated that `UnsupportedOperationException` may be thrown. *You can switch implementations of the list and your app will still behave correctly(as defined in contract)*

Let's say you have this situation:
```
class A {
  List<String> strings;
}
```

```
class B extends A {
}
```
Now, when overriding your class, let's see what you **shouldn't do** in order not to break Liskov principle:
- In class `B` you shouldn't throw some **exception** that is not specified in `A`
- You shouldn't
