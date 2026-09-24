RDFLib-SQLAlchemy
=================

This is IfcOpenShell's fork of [RDFLib/rdflib-sqlalchemy](https://github.com/RDFLib/rdflib-sqlalchemy),
published on PyPI as `ifcopenshell-rdflib-sqlalchemy`, so that Bonsai's Brick support has a store that
works with current SQLAlchemy and rdflib. It carries upstream's `develop` plus what upstream has not
released or merged: SQLAlchemy 2.x support (including the `.subquery()` 2.1 needs), the where-clause
guards from the `brickschema-rdflib-sqlalchemy` fork by Gabe Fierro, `importlib.metadata` instead of
`pkg_resources`, and a graph-aware store as rdflib 7's Dataset requires. Fixes are offered upstream
(RDFLib/rdflib-sqlalchemy#116, #117). The module name stays `rdflib_sqlalchemy` and the rdflib store
plugin stays `SQLAlchemy`, so it is a drop-in replacement; do not install it alongside another
`rdflib-sqlalchemy` distribution.


A SQLAlchemy-backed, formula-aware RDFLib Store. It stores its triples
in the following partitions:

- Asserted non rdf:type statements.
- Asserted rdf:type statements (in a table which models Class membership). The motivation for this partition is primarily query speed and scalability as most graphs will always have more rdf:type statements than others.
- All Quoted statements.

In addition, it persists namespace mappings in a separate table. Table names are prefixed `kb_{identifier_hash}`, where `identifier_hash` is the first ten characters of the SHA1 hash of the given identifier.

Back-end persistence
--------------------

Back-end persistence is provided by SQLAlchemy.

Tested dialects are:

- SQLite, using the built-in Python driver
- MySQL, using the MySQLdb-python driver or, for Python 3, mysql-connector
- PostgreSQL, using the psycopg2 driver or the pg8000 driver.

pysqlite: https://pypi.python.org/pypi/pysqlite

MySQLdb-python: https://pypi.python.org/pypi/MySQL-python

mysql-connector: http://dev.mysql.com/doc/connector-python/en/connector-python.html

psycopg2: https://pypi.python.org/pypi/psycopg2

pg8000: https://pypi.python.org/pypi/pg8000

Development
===========
***Note: Currently, rdflib-sqlalchemy is in maintenance mode. That means the
current maintainer (@mwatts15) will do what he can to keep the package working
for existing use-cases, but new features will not be added and newer versions
of SQLAlchemy will not be supported. If you have an interest in further
development of rdflib-sqlalchemy, please get in touch with @mwatts15 or [core
RDFLib project developers][rdflib-contact].***

[rdflib-contact]: https://rdflib.readthedocs.io/en/stable/#further-help-contact

Github repository: https://github.com/RDFLib/rdflib-sqlalchemy

Continuous integration: https://travis-ci.org/RDFLib/rdflib-sqlalchemy/

![Travis CI](https://travis-ci.org/RDFLib/rdflib-sqlalchemy.png?branch=develop)
![PyPI](https://img.shields.io/pypi/v/rdflib-sqlalchemy.svg)
![PyPI](https://img.shields.io/pypi/status/rdflib-sqlalchemy.svg)
![PyPI](https://img.shields.io/pypi/dw/rdflib-sqlalchemy.svg)

![PyPI](https://img.shields.io/pypi/pyversions/rdflib-sqlalchemy.svg)
![PyPI](https://img.shields.io/pypi/l/rdflib-sqlalchemy.svg)
![PyPI](https://img.shields.io/pypi/wheel/rdflib-sqlalchemy.svg)
![PyPI](https://img.shields.io/pypi/format/rdflib-sqlalchemy.svg)


An illustrative unit test:
==========================

```python

import unittest
from rdflib import plugin, Graph, Literal, URIRef
from rdflib.store import Store


class SQLASQLiteGraphTestCase(unittest.TestCase):
    ident = URIRef("rdflib_test")
    uri = Literal("sqlite://")

    def setUp(self):
        self.graph = Graph("SQLAlchemy", identifier=self.ident)
        self.graph.open(self.uri, create=True)

    def tearDown(self):
        self.graph.destroy(self.uri)
        try:
            self.graph.close()
        except:
            pass

    def test01(self):
        self.assert_(self.graph is not None)
        print(self.graph)

if __name__ == '__main__':
    unittest.main()
```

Running the tests
=================
`pytest` is supported as a test runner, typically called via `tox`. Select the
SQL back-end by setting a `DB` environment variable. Select the database
connection by setting the `DBURI` variable. With `tox`, you can also specify
the Python version.

Using pytest::

    DB='pgsql' DBURI='postgresql+psycopg2://user:password@host/dbname' pytest

Using tox::

    DB='pgsql' DBURI='postgresql+psycopg2://user:password@host/dbname' tox -e py310

DB variants are 'pgsql', 'mysql' and 'sqlite'. Except in the case of SQLite,
you'll need to create the database independently, before execution of the test.

Sample DBURI values::

    dburi = Literal("mysql://username:password@hostname:port/database-name?other-parameter")
    dburi = Literal("mysql+mysqldb://user:password@hostname:port/database?charset=utf8")
    dburi = Literal('postgresql+psycopg2://user:password@hostname:port/database')
    dburi = Literal('postgresql+pg8000://user:password@hostname:port/database')
    dburi = Literal('sqlite:////absolute/path/to/foo.db')
    dburi = Literal("sqlite:///%(here)s/development.sqlite" % {"here": os.getcwd()})
    dburi = Literal('sqlite://') # In-memory
