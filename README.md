otsdb_client
============

A simple Python client to OpenTSDB.
The writing method (put) uses a simple socket and the reading a http request made trough the urllib3.

The current implementation will not use a worker (like the good [potsdb](https://github.com/orionvm/potsdb) package]), fancy multithread control or stream for chunked results. It's a start point to a good (hope) OpenTSDB client.

This package was made and tested only with Python 2.7.

# ToDo:

* Check for limits and blocking impacts;
* Implement worker or multithread;
* Implement the reading with chunks.

Installation
===
Clone this repo, then
```
$ cd otsdb_client
$ python setup.py install
```
or
```
$ pip install git+https://github.com/venidera/otsdb_client.git
```

Usage
===
Complete example of connect, put and read points from OpenTSDB:

```
>>> from otsdb_client import Connection
>>> c = Connection('localhost',4242)
Connected to OpenTSDB
>>> c.put(metric='test_put',value=2000.00,tags={'test':'client','type':'telnet'})
cmd: put test_put 1454552899 2000.00 test=client type=telnet
point written successfully
True
>>> c.query(metric='test_put',aggr='sum',tags={'test':'*'},start='1h-ago')
{'results': [{'metric': u'test_put', 'values': [2000.0], 'ts': [datetime.datetime(2016, 2, 4, 0, 28, 19)], 'tags': {u'test': u'client', u'type': u'telnet'}}]}
>>> c.query(metric='test_put',aggr='sum',tags={'test':'*'},start='1h-ago',union=True)
{'results': {'values': [2000.0], 'ts': [datetime.datetime(2016, 2, 4, 0, 28, 19)]}}
```

## Methods

### Class instantiation (`__init__`):

Create objects of otsdb_client to execute read/write operations with OpenTSDB:

```
class Connection(object):
    def __init__(self,server='localhost', port=4242):
        ...
```

#### Arguments:

* server (str): the IP address or URI of the server that will be accessed;
* port (int): the port that TSD is running.

#### Example:

```
>>> from otsdb_client import Connection
>>> c = Connection()
```

same of

```
>>> c = Connection('localhost',4242)
```

### Write data to OpenTSDB (`put`):

Insert a point (timestamp+value) into a Time Serie (Metric + Tags). At this moment, only one point can be added per call.

```
def put(self,metric,ts=None,value=0.0,tags=dict()):
    ...
```

Better take a look at (http://opentsdb.net/docs/build/html/user_guide/writing.html)[OpenTSDB data model and naming schema.]

#### Arguments:

* metric (str): the name of the metric;
* ts (int - Optional): point's integer timestamp (Epoch/Unix timestamp), if it is not passed will assume `ts = int(mktime(datetime.now().timetuple()))`;
* value (float): the value of the point;
* tags (dict()): a dictionary with tagn (tag name) equals tagv (tag value). Example: `tags={'tag1':'valtag1','tag2':'valtag2'}`.

#### Example:

Storing memory used for a server at this moment. So, metric `sys.mem.used` and tag `host`:

```
c.put(metric='sys.mem.used',value=4321,tags={'host':'server1'})
```

Now insert a point (ts/value) for a metric in `2012-01-10 13:00`:

```
from datetime import datetime
from time import mktime
ts = int(mktime(datatime(2012,01,10,13,00).timetuple()))
c.put(metric='metric.name',ts=ts,value=321.20,tags={'tagname':'tagvalue'})
```

### Read data to OpenTSDB (`query`):

Read points (aggregated  or not) of a time serie. At this moment, only simple use of the `/api/query` endpoint was implemented. The aggregator defined will be validated with the `/api/aggregators` endpoint results.

```
def query(self, metric, aggr='sum', tags=dict(), start='1h-ago', end=None, nots=False,\
      tsd=True, json=False,show_summary=False,union=False,chunked=False):
    ...
```

Supports only one metric.

#### Arguments:
* metric (str) : the metric name;
* aggr (str) : an aggregator (example: `sum`, (http://opentsdb.net/docs/build/html/api_http/aggregators.html)[take a look here]);
* tags (dict) : tags to the points that defined a timeserie ((http://opentsdb.net/docs/build/html/user_guide/writing.html)[Naming schema]);
* start (str) : starting time for the query ((http://opentsdb.net/docs/build/html/user_guide/query/index.html)[look here]);
* end (str - Optional): a end time for the query (default = now);
* nots (bool): it's NoTimeStamps in results, if is defined as `True`;
* tsd (bool): return timestamps as datetime objects for `True`, and integer timestamps for `False`;
* json (bool): if True will return the exact response of the http request;
* show_summary (bool): will add information summary of the query when defined `True`;
* union (bool) : return the points of the time series (Metric+Tags) in one list, union for different tags (Be careful here);
* chunked: will be implemented in future, to stream over urllib3.

#### Example:

Query a metric for values of a tag since 1 day ago:

```
c.query(metric='metric_name',aggr='sum',tags={'tagn':'*'},start='1d-ago')
```

License
=======
This package is released and distributed under the license GNU GPL Version 2, June 1991.
