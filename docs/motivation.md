
# Query JSON Documents using SQLite 


## Overview

Could you write a database schema to store the following JSON?

	{
		"name": "The Parent Trap",
		"released": "29 July 1998",
		"imdb": "http://www.imdb.com/title/tt0120783/",
		"rating": "PG"
		"director": {
			"name": "Nancy Meyers"
			"dob": "December 8, 1949"
		}
	} 

Could you write a query to extract all movies directed by Nancy Meyers?

Can you modify your database schema to additionally store JSON with nested objects (plus other changes)?

	{
		"name": "The Parent Trap",
		"released": 901670400,
		"director": "Nancy Meyers"
		"cast": [
			{"name": "Lindsay Lohan"},
			{"name": "Dennis Quaid"},
			{"name": "Natasha Richardson"}
		]
	}

Could you write a query to return all movies with Lindsay Lohan in the cast?
		
Now. Can you program a machine to do this for you!!? Can your program modify the database, on the fly, as you receive more documents like the above? **Most important:** How do your queries change?

## Problem 

JSON is a nice format to store data, and it has become quite prevalent. Unfortunately, databases do not handle it well; often a human is required to declare a schema that can hold the JSON before it can be queried. If we are not overwhelmed by the diversity of JSON now, I expect we soon will be. I expect there to be more JSON, of more different shapes, as the number of connected devices (and the information they generate) continues to increase.   

https://www.youtube.com/watch?v=4N_ktE4NFIk

## The solution

The easy part is making the schema, and changing it dynamically as new JSON schema are encountered. The harder problem is ensuring the old queries against the new schema have the same meaning. In general this is impossible, but there are particular schema migrations that can leave the meaning of existing queries unchanged.  

By dealing with JSON documents we are limiting ourselves to [snowflake schemas](https://en.wikipedia.org/wiki/Snowflake_schema). This limitation reduces the scope of the problem. Let's further restrict ourselves to a subset of schema transformations that can be handled automatically; we will call them "schema expansions":

1.	Adding a property - This is a common migration
2.	Changing the datatype of a property, or allowing multiple types - It is nice if numbers can be queried like numbers and strings as strings even if they are found in the same property..
3.	Change a single-valued property to a multi-valued property - Any JSON property `{"a": 1}` can be interpreted as multi-valued `{"a": [1]}`. Then assigning multiple values is trivial expansion `{"a": [1, 2, 3]}`.
4.	Change an inner object to nested array of objects - Like the multi-valued case: `{"a":{"b":"c"}}`, each inner object can be interpreted as a nested array `{"a": [{"b":"c"}]}`.  Which similarly trivializes schema expansion.

Each of these schema expansions should not change the meaning of old queries. Have no fear! The depths of history gives us a language that is already immutable under all these transformations!

## Schema-Independent Query Language?

Under an expanding schema, can we write queries that do not change meaning? For hierarchical data, data that fits in a [snowflake schema](https://en.wikipedia.org/wiki/Snowflake_schemahierarchical): **Yes! Yes we can!!!**

Each JSON document can be seen as a single point in a multidimensional Cartesian space; where the properties represent coordinates in that space. Inner objects add dimensions, and nested object arrays represent constellations of points in an even-higher dimensional space. These multidimensional [data cubes](https://en.wikipedia.org/wiki/OLAP_cube) can be represented by [fact tables](https://en.wikipedia.org/wiki/Fact_table) in a [data warehouse](https://en.wikipedia.org/wiki/Data_warehouse). Fact tables can be queried with [MDX](https://en.wikipedia.org/wiki/MultiDimensional_eXpressions). 

With this in mind, we should be able to use MDX as inspiration to query JSON datastores. Seeing data as occupying a (sparse) Cartesian space gives us hints about the semantics of queries, and how they might be invariant over the particular schema expansions listed above.

**Some user documentation may help with understanding the query language**: [JSON Query Expressions](https://github.com/klahnakoski/ActiveData/blob/dev/docs/jx.md)

## Benefits

Having the machine manage the data schema gives us a new set of tools: 

* **Easy interface to diverse JSON** - a query language simplified for JSON documents
* **No schema management** - Schemas, and migrations of schemas, are managed by the machine.
* **Scales well** - Denormalized databases, with snowflake schemas, can be sharded naturally, which allows us to scale.    
* **Handle diverse datasources** - Relieving humans of schema management means we can ingest more diverse data faster. The familiar [ETL process](https://en.wikipedia.org/wiki/Extract,_load,_transform) can be replaced with [ELT](https://en.wikipedia.org/wiki/Extract,_transform,_load), plus we do not loose queriability. Links: [A](http://hexanika.com/why-shift-from-etl-to-elt/), [B](https://www.ironsidegroup.com/2015/03/01/etl-vs-elt-whats-the-big-difference/)
* **Mitigate the need for a (key, value) table** - Automatic schema management allows us to annotate records, or objects, without manual migrations: This prevents the creation of a (key, value) table (the "junk drawer" found in many database schemas) where those annotations usually reside.  
* **Automatic ingestion of changing relational databases** - Despite the limited set of schema expansions, we can handle more general relational database migrations: Relational databases can be [extracted as a series of denormalized fact cubes](https://github.com/klahnakoski/MySQL-to-S3) As a relational database undergoes migrations (adding columns, removing columns, adding relations, splitting tables), the extraction process can continue to capture the changes because each fact table is merely a snowflake subset of the relational whole.
* **Leverage existing database optimization** - This project is about using MDX-inspired query semantics and translating to database-specific query language: We leverage the powerful features of the underlying datastore.  

## Existing solutions

* We might be able to solve the problem of schema management by demanding all JSON comes with a formal JSON schema spec. That is unrealistic; it pushes the workload to the users of the datastore. Plus, it is truly unnecessary given the incredible amount of computer power at our fingertips.
* Elasticsearch 1.x has limited automatic schema detection which has proven useful for indexing and summarizing data of unknown shapes. We would like to generalize this nice feature and and bring machine managed schemas to other datastores.   
* Oracle uses [json_*](http://www.oracle.com/technetwork/database/sql-json-wp-2604702.pdf) functions to define views which can operate on JSON. It has JSON path expressions; mimicking MDX, but the overall query syntax is clunky. Links: [A](https://docs.oracle.com/database/121/ADXDB/json.htm#ADXDB6246), [B](https://blogs.oracle.com/jsondb/entry/s)
* Spark has [Schema Merging](http://spark.apache.org/docs/latest/sql-programming-guide.html#schema-merging) and nested object arrays can be accessed using [explode()](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=explode#pyspark.sql.functions.explode). Spark is a little more elegant, despite the the fact it exposes *how* the query executes.
* Sqlite has the [JSON1 connector](https://www.sqlite.org/json1.html) - Which is a limited form of Oracle's solution; it requires manual schema translation which complicates queries. 

These existing solutions solve the hard problems from the bottom up; managing file formats, organizing indexes, managing resources and query planning. Each built their own stack with their own query conventions guided by the limitations of architecture they built. 

This project is about working from the top down: A consistent way to query messy data; identical, no matter the underlying data store; so we can swap datastores based on scale of the problem.     
