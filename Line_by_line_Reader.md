### How to read file from classpath line by line easily
Often our task is to read file or input stream line by line and return some result.

For example:
- We have file with many lines where one line have format `{"name": "some name","age": 29}`
and we have to return `List<Person>`
- Also file with many lines in format `paramA:valueA,paramB:valueB,paramC:valueD` 
and we need to return `Map<String,String>`

Now, here is simple solution(assume that file is on classpath and that path is relative
, those two things are not important in this example):

Reading persons:

```
public List<Person> readPersons(String path) throws IOException {
        List<Person> persons = new LinkedList<>();
        InputStream inputStream = getClass().getResourceAsStream(path);
        if (inputStream == null) {
            return new LinkedList<>();
        }
        try (
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))
        ) {
                    String body;
            while ((body = reader.readLine()) != null) {
                //parse json, create Person object and add that object into list
                    }
        } catch (IOException e) {
            throw e;
        } finally {
            inputStream.close();
        }
        return persons;
    }
```

Similar thing, reading params:

```
public Map<String,String> readParams(String path) throws IOException {
        Map<String,String> params = new HashMap<>();
        InputStream inputStream = getClass().getResourceAsStream(path);
        if (inputStream == null) {
            return new LinkedList<>();
        }
        try (
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))
        ) {
                    String body;
            while ((body = reader.readLine()) != null) {
                //parse params and add key value pairs into map
                    }
        } catch (IOException e) {
            throw e;
        } finally {
            inputStream.close();
        }
        return params;
    }
```

And this solution works. But you will notice that you had repeated a lot of code, basically the only
thing that differs is how to parse that one line, everything else is the same.

So why not try something like this?

```
public class LineByLineReader <R> {
    public List<R> read(String path, Function<String,R> function) throws IOException {
        List<R> lines = new LinkedList<>();
        InputStream inputStream = getClass().getResourceAsStream(path);
        if (inputStream == null) {
            return new LinkedList<>();
        }
        try (
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))
        ) {
                    String body;
            while ((body = reader.readLine()) != null) {
                lines.add(function.apply(body));
                    }
        } catch (IOException e) {
            throw e;
        } finally {
            inputStream.close();
        }
        return lines;
    }
}
```
We specified `Function` as param and now things that differed in last example are out of our reader.
We can call this reader this way:

```
LineByLineReader<Map<String, String>> lineByLineReader = new LineByLineReader<>();
lineByLineReader.read("somePath",line -> {
    //convert this one line to <Map<String, String>>
})
```

This way, our job is much easier and also we will not ever again to close our resources,
our code will not look ugly because of try catch block etc.