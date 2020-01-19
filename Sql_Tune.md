### How would you tune this sql query?

Imagine having index on ```person_id``` and you would like to perform query:
```select bank_account_number from bank_account where person_id = :1 or :1 is null```.
 
Unfortunately, most databases are not storing `null` values into the index.
Also, since database is preparing execution plan before execution of sql query
and this plan is cached and it is the same for all values of `:1`
 database don't know in advance what will be a value for `:1` and thus can't use index on 
 `person_id` column because maybe `:1` can be null . What a damage!

Fortunately, we can rewrite this query so we can use index! Take a look:

```
//1st select
select bank_account_number from bank_account where person_id = :1 and :1 is not null
 
union all
 
//2nd select
select bank_account_number from bank_account where :1 is null
```

With this query, when `:1` is null, database will recognize that first select return 0 rows and only
second select will be executed. `Full table scan` will be performed in this case.

On the other hand, when `:1` have some value, database will recognize that second select return 0 rows
and it will perform just first select. But this time, database is sure that `:1` is not null in
 first select(because there is `:1 is not null` condition) and it can use index on `person_id` column! `Index scan` will be used this time 
 instead of `Full table scan`! This will dramatically improve time of this sql query.
 
 ##### Further reading
 
 If this was interesting article for you, consider reading [Sql tuning](https://www.amazon.com/SQL-Tuning-Generating-Optimal-Execution/dp/0596005733).
 I really enjoyed this book and there are a lot of useful things
 (this particular article that I wrote was mentioned in the book) on how to tune your sql queries so 
 they can run like a charm.