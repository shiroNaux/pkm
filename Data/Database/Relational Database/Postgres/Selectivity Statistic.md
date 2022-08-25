---
---
# Postgres Statistics
- PostgreSQL has an undeniably clever query planning system that auto-tunes based on the data in the system. It samples tables to gain statistics about the distribution of data, and uses those statistics to choose the order of joins and filters applied to the data for the most efficient query execution
- Even more amazing, the query planning system is modular enough to integrate user-defined data types, like the `geometry` and `geography` types in [[PostGIS]]. So complex queries involving spatial filters are also correctly planned on-the-fly.

## Statistics Targets
- The system has a rough idea of the selectivity of any given filter or join condition, and can use that estimate to choose the most efficient order of operations to execute multi-table and multi-filter queries over complex collections of relational tables.
- By default, a PostgreSQL data ships with a `default_statistics_target` of `100`. This means that to populate the statistics for a column the database will draw a sample of `300 * default_statistics_target = 30000` rows
- The system will then use that data to populate the "common value list" and column histogram with roughly `default_statistics_target` entries
- A large table of keys arranged in a Pareto distribution can make things hard for the statistics system. There may be more common keys than can easily fit into the "common value list", so joins get poorly planned for example

- The statistics target on each table and column can be individually changed, so that more data are sampled, and more granular statistics are gathered.
```sql
ALTER TABLE people ALTER COLUMN age SET STATISTICS 200;
```
- Gathering more statistics is usually sufficient to correct planning behavior. However, in special cases (like when a query uses multiple filters on strongly correlated columns) it might be necessary to pull out "[extended statistics](https://www.postgresql.org/docs/current/sql-createstatistics.html)" to gather even more data for planning.

## Indexes Make (Some) Things Faster


For most uses, this rule-of-thumb is correct: indexes make random access of a **small number** of items faster, at the expense of making inserts and updates to a table **slightly slower**.

What happens, though, if you are accessing a **large number** of items from a table?

An index is a tree. It's fast to find one item traversing through a tree, it's called an "index scan". But finding a lot of items involves running through the tree over and over and over. It is slow to index scan all the records in a tree.
An efficient index scan query will have a narrow filter:

```sql
SELECT * FROM people WHERE age = 4;
```

Conversely, reading a table from from front to back can be quite fast, if you are interested in most of the records. This is called a "sequence scan".

An efficient sequence scan query will have a wide filter:

```sql
SELECT * FROM people WHERE age BETWEEN 45 AND 95;
```

For any given filter, a relational database will figure out whether the filter is most efficiently run as an index scan or a sequence scan. But how?

## Selectivity

The "selectivity" of a filter is a measure of what proportion of the records will be returned by the filter. A filter that returns only 1% of the records is "very selective", and vice versa.
An ordinary filter on numbers or strings or dates might define an equality or a range as a filter. A spatial filter on geographic objects defines a bounding box as a filter. A filter is just a true/false test that every object in the column can be subjected to.

Here's two spatial filters, which one do you suppose is more selective?

The red one, right? Just like our SQL example above on ages, the smaller rectangle must return fewer records. Except not necessarily. Suppose our data table looked like this.

So the blue box is selective and should use an index scan and the red one should use a sequence scan.

But wait, how does the database know which box is selective, without running a query and actually **applying the filters** to the data?

## Statistics

When the `ANALYZE` command is run, the database will gather "statistics" for every column that has an index on it. (The "autovacuum" system that runs in the background of every PostgreSQL cluster is also an "autoanalyze", gathering statistics at the same time it does table maintenance.)

For data types like integers, real numbers, text and dates the system will gather a "common value list" of values that show up frequently in the table, and a histogram of the frequency of values in different ranges.

For spatial data types, the system builds a spatial histogram of frequencies of occurrance. So, for our example data set, something like this.

Instead of being a potentially unconstrained collection of maybe millions of spatial objects, the database now just has to make a quick summary of a few cells in a histogram to get an estimate of how many features a given query filter will return.

The spatial statistics analyzer in PostGIS is actually a little more sophisticated than that: before filling out the histogram it adjusts the cell widths for each dimension, to try and catch more resolution in places where the data is more dense.

All these statistics do have a cost. Increasing statistics targets will make the `ANALYZE` command run a little slower, as it samples more data. It will also make the planning stage of queries slower, as the larger collection of statistics need to be read through and compared to the filters and joins in the SQL. However, for complex BI queries, more statistics are almost always worth while to get a better plan for a complex query.

# Conclusion
-   PostgreSQL has an undeniably clever query planning system which auto-tunes based on the data in the system.
-   PostGIS plugs right into that clever system and provides spatial selectivity estimates to ensure good plans for spatial queries too.
-   For even more complex situations, such as selectivity estimates on highly correlated columns, PostgreSQL also includes an "[extended statistics](https://www.postgresql.org/docs/current/sql-createstatistics.html)" system to capture those cases.

# References
1. [Postgres Indexes, Selectivity, and Statistics](https://www.crunchydata.com/blog/indexes-selectivity-and-statistics)
2. [PostgreSQL Document](https://www.postgresql.org/docs/current/sql-createstatistics.html)
