# The SQL style guide

The goal of this style guide is to improve the readability, quality, performance, and consistency of SQL queries.

## Formatting

### Keywords

* Keywords should be UPPERCASE.

    ```SQL
    /* Good */
    SELECT COUNT(1) FROM tablename WHERE 1;
    
    /* Bad */
    select count(1) from tablename where 1;
    ```

### Names
* Named objects should not be surrounded by backticks, no matter what MySQL says when it dumps the table structure for you.
  *  If you need to use backticks because of something in your table name, rename your table.
* Table names are singular.
* Avoid tables and columns named with reserved SQL keywords.

### Indentation and newlines

* Newlines should be used for any query that is at all complex or longer than 72 characters.

* Each clause should begin a new line.
  SELECT, JOIN, LEFT JOIN, OUTER JOIN, WHERE, UNION, etc. are keywords that begin new clauses.

    ```SQL
    /* Good */
    SELECT COUNT(1)
      FROM tablename
     WHERE really_loooong_column = CONCAT(other_column, ' street');
    
    /* Bad */
    SELECT COUNT(1) FROM tablename WHERE really_loooong_column = CONCAT(other_column, ' street');
    ```    

* The keywords that begin a clause should be right-aligned.
  The idea is to make a single character column between the keywords and their objects.

    ```SQL
    /* Good */
    SELECT COUNT(1)
      FROM tablename
     WHERE 1;
    
      SELECT key_column, COUNT(1)
        FROM tablename
    GROUP BY key_column;
    
    /* Bad */
    SELECT COUNT(1)
    FROM tablename
    WHERE 1;
    ```

## Common Table Expressions (CTEs)

[From AWS](http://docs.aws.amazon.com/redshift/latest/dg/r_WITH_clause.html):

>`WITH` clause subqueries are an efficient way of defining tables that can be used throughout the execution of a single query. In all cases, the same results can be achieved by using subqueries in the main body of the `SELECT` statement, but `WITH` clause subqueries may be simpler to write and read.

The body of a CTE must be one indent further than the `WITH` keyword. Open them at the end of a line and close them on a new line:

```sql
WITH backings_per_category AS (
  SELECT
    category_id,
    deadline,
    ...
)
```

Multiple CTEs should be formatted accordingly:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    ...
), backers_and_creators AS (
  ...
)
SELECT * FROM backers;
```

If possible, `JOIN` CTEs inside subsequent CTEs, not in the main clause:

__GOOD__:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    backer_id,
    COUNT(backings_per_category.id) AS projects_backed_per_category
  INNER JOIN ksr.users AS users ON users.id = backings_per_category.backer_id
), backers_and_creators AS (
  ...
)
SELECT * FROM backers_and_creators;
```

__BAD__:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    backer_id,
    COUNT(backings_per_category.id) AS projects_backed_per_category
), backers_and_creators AS (
  ...
)
SELECT * FROM backers_and_creators
INNER JOIN backers ON backers_and_creators ON backers.backer_id = backers_and_creators.backer_id
```

Always use CTEs over inlined subqueries.

* Subqueries should be aligned as though the open parenthesis were the 0-column
  So, they should be indented as a unit, to identify them as subqueries.  They should continue to have the opening keywords right-aligned.

    ```SQL
    /* Good */
    SELECT *
      FROM (  SELECT candidates.name, count(1)
                FROM candidates
                JOIN votes ON candidates.id = votes.candidate_id
            GROUP BY candidates.name) name_count
      JOIN city c ON name_count.name = c.mayor;
    
    /* Bad */
    SELECT *
      FROM (SELECT candidates.name, count(1)
        FROM candidates
        JOIN votes ON candidates.id = votes.candidate_id
        GROUP BY candidates.name) name_count
      JOIN city ON name_count.name = city.mayor;
    ```   

## Structure

* Column aliases should always use the keyword AS
  This becomes significant when a query has several columns selected with columns aliased.  Without the AS keyword, a dropped comma makes two columns become a single aliased column.

    ```SQL
    /* Good */
    SELECT ebe_ebs_sox_flag_set_for_all_crs AS sox_ok
      FROM tablename;
    
    /* Bad */
    SELECT ebe_ebs_sox_flag_set_for_all_crs sox_ok
      FROM tablename;
    
    ```    
* Table aliases and column aliases should be descriptive.
  Much like variable names, "a", "b", "x", etc are not generally useful in and of themselves outside of short examples.

* Tiny names for table aliases can sometimes work as abbreviations.
  As an example, if "releases" is referenced frequently, it might make sense to abbreviate it "r".  However, "rel" is almost as short, and much more descriptive.  Have a good reason for "r" instead of "rel".
