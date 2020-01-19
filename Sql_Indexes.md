### What you should know about SQL indexes as Java developer?

Majority of developers don't know a lot about sql performance tuning. I find this topic very interesting 
and I read a lot about it but I think that every developer should at least know a little bit theory about sql
indexing and database caching system.

##### Why we need SQL indexes? 

Imagine your database have about 50 million records. ```10000``` of them have ```Smith```
as they surname. How fast database can fetch records when you perform 
 ```select first_name,id from citizen where last_name  = 'Smith'``` query?

In this case, database don't have choice, it have to read every record, check condition and evict records that don't fit.
Doing that is slow as hell. But if we add ```INDEX``` on ```last_name``` field,
query will perform much faster.
 
##### How index works
Basically ```INDEX``` is just ```B-TREE``` where only on leaf nodes we store values.
We use other nodes just to find path to leaf nodes. In sql index tree every node have about 300 values. This number
depends on lot of factors but for this simple tutorial you can say that it is 300. So in our case when we have an index and
our table contains 50 million records, in 4 hops(300^3 = 27000000 and 300^4 = 8100000000, 50 milion is between those 2 values)
 we can easily find 34 leaf nodes (10000/300=33,3333) where our records are saved. So, instead of reading 50 million records
we easily found 10000 records and read just them! 
This can improve query execution time 30x or 50x, depending on column selectivity.
Wow, that is great improvement! 

##### Can we add index on every field? Unfortunately, not.

There is a penalty when adding indexes. When you update ```last_name```, you have to update your index too.
When you add citizen to db ,remove citizen from db, you have to update your index
(particularly updating and deleting can screw up performance). You are using your
CPU to update index instead of adding rows faster. 
Also, indexes are stored on disk, so you will need more disk space.
Fortunately, hard disks are cheap so this problem with hard disk is not really big like it can be.

##### How to choose where to add index?
Choosing on which column to add index is hard but some simplest advice would be: 
"Add index on those tables which are not updated often and on column which is really often in WHERE clause"

##### Further reading
This was the simplest explanation where I omitted lot of important details. 
If this was interesting article for you, consider reading [Sql tuning](https://www.amazon.com/SQL-Tuning-Generating-Optimal-Execution/dp/0596005733).
I really enjoyed this book and there are a lot of useful things on how to tune your sql queries so 
they can run like a charm.