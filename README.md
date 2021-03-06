# otsdb_client

[![GitHub Release](https://img.shields.io/badge/release-v0.0.3-blue.svg)](https://github.com/venidera/otsdb_client/releases)
[![Build Status](https://img.shields.io/badge/build-passing-green.svg)](https://github.com/venidera/otsdb_client)
[![GitHub license](https://img.shields.io/badge/license-GPLv3-yellow.svg)](https://raw.githubusercontent.com/venidera/otsdb_client/master/LICENSE)

A simple Python Client to async consume of OpenTSDB HTTP API
The writing method (put) was initially done using a simple socket and the reading http request made trough the urllib3.
We refactory everything to use only HTTP with grequests.

This package was made and tested with Python 2.7 and 3.5.

## Table of contents

* [Requirements](#requirements)
* [Installation](#installation)
* [Development and Tests](#developmentandtests)
* [To-do list](#todolist)
* [Documentation](#documentation)
* [Maintainers](#maintainers)
* [License](#license)

## Requirements:

* grequests==0.3.0

## Installation

Clone the repo for development purposes:

```
$ git clone https://github.com/venidera/otsdb_client.git
$ cd otsdb_client
$ virtualenv --python=python3.5 --prompt=" OTSDB Client " venv-3.5
$ source venv-3.5/bin/activate ; pip install pip setuptools --upgrade
$ python setup.py install
```

Install using pip:

```
$ pip install git+https://github.com/venidera/otsdb_client.git
```

## Development and tests

To avoid errors, the development can be done in the repo directory but the tests must be executed in an isolated directory because `__init__.py` files can hide import errors.
An example of installation for tests is presented below:

```
$ cd <repo>
$ mkdir tests
$ cd tests
$ virtualenv --python=python3.5 --prompt=" <package name> " venv-3.5
$ source venv-3.5/bin/activate
$ pip install pip setuptools --upgrade
$ cd .. ; python setup.py install ; cd -
```

The steps listed above will create a virtualenv with the package installed. If you change the code of the package you must execute `cd .. ; python setup.py install ; cd -``
The tests must consider the import call of the package and its classes and functions must be tested.
If everything works fine, commit changes and execute `git pull && git push` or a `pull request`.
All files created inside the `tests` directory must be kept untracked in the repository.

## To-do list

* Check for limits and blocking impacts: Done - Solution: use grequests to make async http calls;
* Implement worker or multithread: pending (inpired by [potsdb](https://github.com/orionvm/potsdb));
* Implement the reading with chunks: pending;
* Cover all endpoints listed in the [OpenTSDB HTTP API doc](http://opentsdb.net/docs/build/html/api_http/index.html).

## Documentation

### Connection class

Create objects of otsdb_client to execute read/write operations with OpenTSDB.
The `__init__` method will ping the server port and will cause an exception if it fails.

```python
class Connection(object):
    def __init__(self,server='localhost', port=4242):
        ...
```
Arguments:

* server (str): the IP address or URI of the server that will be accessed;
* port (int): the port that TSD is running.

 Example:

```python
>>> from otsdb_client import Connection
>>> c = Connection()
```

### Methods (API Endpoints Covered)

The following [OpenTSDB HTTP API](http://opentsdb.net/docs/build/html/api_http/index.html) endpoints listed below can be consumed using this package:

#### `version()`

Endpoint **[/api/version](http://opentsdb.net/docs/build/html/api_http/version.html)**

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='192.168.30.80')
>>> c.version()
{'branch': 'next', 'version': '2.3.0-RC1', 'user': 'hobbes', 'repo': '/home/hobbes/opentsdb_OFFICIAL/build', 'short_revision': '306603c', 'full_revision': '306603c313fda191706479e7cc78a931f36ae89b', 'repo_status': 'MINT', 'host': 'clhbase', 'timestamp': '1462215755'}
>>>
```

#### `filters()`

Endpoint **[/api/config/filters](http://opentsdb.net/docs/build/html/api_http/config/filters.html)**

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='192.168.30.80')
>>> c.filters()
{'iwildcard': {'description': 'Performs ...'}, 'not_literal_or': {'description': 'Accepts ...'}, 'regexp': {'description': 'Provides full...'}, 'wildcard': {'description': 'Performs pre, post and in-fix ...'}, 'literal_or': {'description': 'Accepts one or ...'}, 'not_iliteral_or': {'description': 'Accepts one or more ...'}, 'iliteral_or': {'description': 'Accepts one ...'}}
>>>
```

#### `statistics()`

Endpoint **[/api/stats](http://opentsdb.net/docs/build/html/api_http/stats.html)**

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='192.168.30.80')
>>> c.statistics()
[{'metric': 'tsd.connectionmgr.connections', 'timestamp': 1471268960, 'tags': {'type': 'open', 'host': 'opentsdb-server'}, 'value': '1'}, {'metric': 'tsd.connectionmgr.connections', 'timestamp': 1471268960, 'tags': {'type': 'rejected', 'host': 'opentsdb-server'}, 'value': '0'},...]
>>>
```

#### `aggregators`

It's a `list()` created in the `__init__` method. It contains the aggregators returned by the endpoint `aggregators`.

Endpoint **[/api/aggregators](http://opentsdb.net/docs/build/html/api_http/aggregators.html)**

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='192.168.30.80')
>>> c.aggregators
['mult', 'p90', 'zimsum', 'mimmax', 'sum', 'p50', 'none', 'p95', 'ep99r7', 'p75', 'p99', 'ep99r3', 'ep95r7', 'min', 'avg', 'ep75r7', 'dev', 'ep95r3', 'ep75r3', 'ep50r7', 'ep90r7', 'mimmin', 'p999', 'ep50r3', 'ep90r3', 'ep999r7', 'last', 'max', 'count', 'ep999r3', 'first']
>>>
```

#### `put()`

Endpoint **[/api/put](http://opentsdb.net/docs/build/html/api_http/put.html)**

#### `suggest(type='metrics',q='',max=9999)`

Endpoint **[/api/suggest](http://opentsdb.net/docs/build/html/api_http/suggest.html)**

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='192.168.30.80')
>>> c.suggest(type='metrics',q='',max='10')
['test_put', 'test_put1', 'test_put2']
>>> c.suggest(type='tagk',q='',max='10')
['key', 'tagk']
>>> c.suggest(type='tagv',q='',max='10')
['0', '1', 'tagv']
```

#### `query()`

Endpoint **[/api/query](http://opentsdb.net/docs/build/html/api_http/query.html)**

#### `query_exp()`

Endpoint **[/api/query/exp](http://opentsdb.net/docs/build/html/api_http/query/exp.html)**


### Write data to OpenTSDB (`put`):

Insert a point (timestamp+value) into a Time Serie (Metric + Tags). At this moment, only one point can be added per call.

```python
def put(self,metric,ts=None,value=0.0,tags=dict()):
    ...
```

Better take a look at [OpenTSDB data model and naming schema.](http://opentsdb.net/docs/build/html/user_guide/writing.html)

#### Arguments:

* metric (str): the name of the metric;
* ts (int - Optional): point's integer timestamp (Epoch/Unix timestamp), if it is not passed will assume `ts = int(mktime(datetime.now().timetuple()))`;
* value (float): the value of the point;
* tags (dict()): a dictionary with tagn (tag name) equals tagv (tag value). Example: `tags={'tag1':'valtag1','tag2':'valtag2'}`.

#### Example:

Storing memory used for a server at this moment. So, metric `sys.mem.used` and tag `host`:

```python
c.put(metric='sys.mem.used',value=4321,tags={'host':'server1'})
```

Now insert a point (ts/value) for a metric in `2012-01-10 13:00`:

```python
from datetime import datetime
from time import mktime
ts = int(mktime(datatime(2012,01,10,13,00).timetuple()))
c.put(metric='metric.name',ts=ts,value=321.20,tags={'tagname':'tagvalue'})
```

### Read data from OpenTSDB (`query`):

Read points (aggregated  or not) of a time serie. At this moment, only simple use of the `/api/query` endpoint was implemented. The aggregator defined will be validated with the `/api/aggregators` endpoint results.

```python
def query(self, metric, aggr='sum', tags=dict(), start='1h-ago', end=None, nots=False,\
      tsd=True, json=False,show_summary=False,union=False,chunked=False):
    ...
```

Supports only one metric.

#### Arguments:
* metric (str) : the metric name;
* aggr (str) : an aggregator (example: `sum`, [take a look here](http://opentsdb.net/docs/build/html/api_http/aggregators.html));
* tags (dict) : tags to the points that defined a timeserie ([Naming schema](http://opentsdb.net/docs/build/html/user_guide/writing.html));
* start (str) : starting time for the query ([look here](http://opentsdb.net/docs/build/html/user_guide/query/index.html));
* end (str - Optional): a end time for the query (default = now);
* nots (bool): it's NoTimeStamps in results, if is defined as `True`;
* tsd (bool): return timestamps as datetime objects for `True`, and integer timestamps for `False`;
* json (bool): if True will return the exact response of the http request;
* show_summary (bool): will add information summary of the query when defined `True`;
* union (bool) : return the points of the time series (Metric+Tags) in one list, union for different tags (Be careful here);
* chunked: will be implemented in future, to stream over urllib3.

#### Example:

Query a metric for values of all its tags since 1 day ago:

```python
results = c.query(metric='metric_name',aggr='sum',tags={'tagn':'*'},start='1d-ago')
```

#### How to use (Deprecated, will be revised soon)

```python
>>> from otsdb_client import Connection
>>> c = Connection(server='localhost', port=4242)
>>> c.put(metric='test_put',timestamps=[int(mktime(datetime.now().timetuple()))],values=[2000.00],tags={'tagk':'tagv'})

>>> c.query(metric='test_put',aggr='sum',tags={'test':'*'},start='1h-ago')
{'results': [{'metric': u'test_put', 'values': [2000.0], 'ts': [datetime.datetime(2016, 2, 4, 0, 28, 19)], 'tags': {u'test': u'client', u'type': u'telnet'}}]}
>>> c.query(metric='test_put',aggr='sum',tags={'test':'*'},start='1h-ago',union=True)
{'results': {'values': [2000.0], 'ts': [datetime.datetime(2016, 2, 4, 0, 28, 19)]}}
```



## Maintainers

Venidera's development team:

* [Andre E. Toscano](https://github.com/aemitos) - [andre@venidera.com](mailto:andre@venidera.com);
* [Rafael G. Vieira](https://github.com/rafaelgiordano12) - [rafael@venidera.com](mailto:rafael@venidera.com).

## License

This package is released and distributed under the license [GNU GPL Version 3, 29 June 2007](https://www.gnu.org/licenses/gpl-3.0.html).
