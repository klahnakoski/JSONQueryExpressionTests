# JSON Query Expression Tests

This project is a test suite for [JSON Query Expressions](https://github.com/klahnakoski/ActiveData/blob/dev/docs/jx.md).

## Motivation

JSON Query Expressions are meant to be a communication standard between services. They are targeted toward business intelligence, analytic processing, data warehouse and dashboard applications.  

This test suite is used to ensure consistency between various datastores. This consistency allows us to exchange datastores based on factors like data volume, ACID requirements, and desired query speed.  

## Installation

JSON Query Expression Tests are designed to be part of a larger test suite in. It is expected that the `tests/test_jx` subdirectory of this project be inserted into the `tests/test_jx` subdirectory of those larger suites.  How this is done will be left to the project owner.  

## Test Structure

Every test has `data`, a `query`, and a few `expecting` responses. Here is an example:

```python
def test_single_star_select(self):
    test = {
        "data": [
            {"a": "b"}
        ],
        "query": {
            "select": "*",
            "from": TEST_TABLE
        },
        "expecting_list": {
            "meta": {"format": "list"}, "data": [
            {"a": "b"}
        ]},
        "expecting_table": {
            "meta": {"format": "table"},
            "header": ["a"],
            "data": [["b"]]
        },
        "expecting_cube": {
            "meta": {"format": "cube"},
            "edges": [
                {
                    "name": "rownum",
                    "domain": {"type": "rownum", "min": 0, "max": 1, "interval": 1}
                }
            ],
            "data": {
                "a": ["b"]
            }
        }
    }
    self.utils.execute_tests(test)
```

* **data** - A list of documents to be loaded into the datastore under a name assigned to `TEST_TABLE`
* **query** - The query we will be executing. It must include the `TEST_TABLE`, which is a placeholder for the dynamically assigned test-time table name
* **expecting_*** - An entry for each of the expected formats (`list`, `table` or `cube`). 

When using this test suite, the project must implement `self.utils.execute_tests()` to accept this datastructure and perform the necessary loading, querying and comparison. To do this, make a class that with an `execute_tests()` method, and ensure `tests.__init__.py` assigns it to `test_jx_utils`, like so:  

```python
	# Somewhere in tests.__init__.py
    test_jx.utils = SQLiteUtils(test_jx.global_settings)
```
see <insert url here> as an example.

The `test_jx` module will ensure your `tests` module is imported before tests are executed: 

```python
class BaseTestCase(FuzzyTestCase):
    def __init__(self, *args, **kwargs):
        FuzzyTestCase.__init__(self, *args, **kwargs)
        if not utils:
            try:
                import tests
            except Exception:
                Log.error("Expecting ./tests/__init__.py to set `global_settings` and `utils` so tests can be run")
        self.utils = utils
```


## Implementations

Here are the projects that use this test suite. 

* [jx-sqlite](https://github.com/mozilla/jx-sqlite) Python implementation using Sqlite3
* [ActiveData](https://github.com/mozilla/ActiveData) Python implementation for Elasticsearch versions 1.7, 5.x and 6.x



  
