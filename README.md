JSON Query Expression Tests
===========================

This project is a test suite for [JSON Query Expressions](https://github.com/klahnakoski/ActiveData/blob/dev/docs/jx.md).

There are currently connectors for  

* Elastisearch 1.7.x
* Sqlite 2.x

Hopefully, over time, more connectors will be made, and the suite improved. 

Installation
------------

JSON Query Expression connectors are meant to be their own project. It is expected that the `tests` subdirectory of this project be insert into the `tests` subdirectory of those other projects to provide the test suite.  How this is done will be left to the project owner.

History
-------

This test suite is used to test the [ActiveData](https://github.com/klahnakoski/ActiveData) service logic. ActiveData is the project name for the ElasticSearch 1.7.x translation service. This test suite is also in use to help guide development of a Sqlite connector too.  

Maybe a connector will become a GSOC project. [Here is my proposal](docs/GSOC%20Proposal.md)


  
