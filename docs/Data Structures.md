# Data Structures

This is document does not cover academic data structures for solving problems

This document covers engineering strategies for keeping data simple

## Intended Audience 

Application programmers with little or no SQL experience will benefit most. If you are a wizard at normalization and denormalization you may find this a pleasant read with little new information.

## Overall structure

The best way to instruct on the principals of data structures is to go through the anti patterns, say why they cause problems, and show a solution.

## Anti-patterns

These anti-patterns assume you are limiting yourself to classic database organizing principals and query languages.  We assume you are limited to standard database operators. You choice of operators dictates your operands. If you are considering a document query language, some of these anti-patterns are acceptable.

Without loss of generality, we will restrict ourselves to JSON objects. 

### Objects are rows in a table

Throughout this documentation we will show objects, or lists of objects. This will aid readablity, but it is not intended for underlying implementation. Every object should be seen as a row in a table; the properties indicate the values for each of the cells in that row. 

    [
        {"url": "mozilla.org", "last_visited": "2018-12-10 14:48:23Z"},
        {"url": "google.com", "last_visited": "2018-12-09 15:59:00Z"}
    ]

Remember, this form is meant for your viewing pleasure. It is intended that this data be represented as compact rows in a table:

|     url     |      last_visited    |
| ----------- | -------------------- |
| mozilla.org | 2018-12-10 14:48:23Z |
| google.com  | 2018-12-09 15:59:00Z |




### Avoid meaningful dictionary keys

Dictionaries are objects that map values to other values. In Python, the `dict` is a type of mapping. In Javascript, objects can map strings to objects. These are useful data structures, but using the key as a meaningful value should be avoided: 

It is tempting to use dictionaries as efficient lookup tools: Databases call those dictionaries "indexes", and it is considered an optimization on top of a table. You, as a programmer, should not be optimizing your datastructures unless there is a measurable reason to do so.

Here is one dictionary from a list that maps a URL, to some metadata about that URL:

    sites = {
        "mozilla.org": {"last_visited": "2018-12-10 14:48:23Z"}
        "google.com": {"last_visited": "2018-12-09 15:59:00Z"}
    }

The code that uses this data looks something like

    last_visit = sites[url]["last_visited"]

Notice the key, the `url`, is not named in the data; it must be inferred from the code that uses this data.


This data can not be understood without the code, this data is dependent on the code. We should remove that dependency by giving the key a useful name:

    [
        {"url": "mozilla.org", "last_visited": "2018-12-10 14:48:23Z"},
        {"url": "google.com", "last_visited": "2018-12-09 15:59:00Z"}
    ]


Your data access pattern must change though:

**javascript**

    last_visit = from(sites).find(row => row.url==url).last visited;


**python**

	last_visit = first(row['last_visited'] for row in sites if row['url']==url)


**SQL**

    last_visit := SELECT last_visited FROM sites WHERE url = url LIMIT 1

**Pandas**

	last_visit := df.sites[df.url==url, ["last_visited"]][0]

**JSON Query Expression**

    last_visit = jx.query({"from": sites, "select":"last_visited", "where":{"eq":{"url": url}}})

Accessing the single value is more verbose, and arguably slower. But please notice all forms have he same three pieces of information:

* collection - `sites`
* filter - `url = url`
* projection - `last_visited`

The rest is boilerplate: Specifically, the use of `row` in the procedural languages.  I contend that the boilerplate is a problem with the language, not with the data structure.



If you want faster access, use a 



### Avoid Inner Objects


### Uniform list structure


### Null aware operators


### Treat nested object arrays as separate tables






