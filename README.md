Prerequisites
-------------

* Python 2.6 or 2.7
* pip

Dependencies
------------

Common:

    pip install paramiko

EC2:

    pip install boto

Performance tests:

    pip install couchdbkit

    pip install pymongo

Documentation:

    pip install sphinx

    pip install sphinx-pypi-upload

Usage
-----

    $ ./testrunner  -h
    Usage: testrunner [options]

    Options:
      -h, --help            show this help message and exit
      -q
      -p PARAMS, --params=PARAMS
                            Optional key=value parameters, comma-separated -p
                            k=v,k2=v2,...
      -n, --noop            NO-OP - emit test names, but don't actually run them
                            e.g -n true
      -l LOGLEVEL, --log-level=LOGLEVEL
                            e.g -l info,warning,error

      TestCase/Runlist Options:
        -i INI, --ini=INI   Path to .ini file containing server information,e.g -i
                            tmp/local.ini
        -c RUNLIST, --config=RUNLIST
                            Config file name (located in the conf subdirectory),
                            e.g -c py-view.conf
        -t TESTCASE, --test=TESTCASE
                            Test name (multiple -t options add more tests) e.g -t
                            performance.perf.DiskDrainRate

Resource Files
--------------

Ini files represents the ns_server information which is accessible to the tests.

**[global]** section defines the rest username, password that tests use to login to
ns_server.

Example:

    [global]
    username:Administrator
    password:membase

    [membase]
    rest_username:Administrator
    rest_password:asdasd

**[servers]** section lists port and ssh related information. ssh connection
information is required for small subset of tests where test needs to perform
installation,backup or restore. If ns_server instances are started using
ns_server/cluster_run script then you only need to define ip and port for those
nodes.

Example:

    [servers]
    1:10.1.6.104_1
    2:10.1.6.104_2
    3:10.1.6.104_3
    4:10.1.6.104_4

    [10.1.6.104_1]
    ip:10.1.6.104
    port:9000

    [10.1.6.104_2]
    ip:10.1.6.104
    port:9001

    [10.1.6.104_3]
    ip:10.1.6.104
    port:9002

    [10.1.6.104_4]
    ip:10.1.6.104
    port:9003

Test Execution and Reporting
----------------------------

For every test run testrunner creates a temp folder and dumps the logs and
xunit reports in the newly generated folder.

for instance if you run

    $ ./testrunner -i resources/jenkins/single-node-centos-32.ini -t setgettests.MembaseBucket.value_100b

you will see this summary after each test is ran:

    summary so far suite setgettests.MembaseBucket , pass 1 , fail 0
    logs and results are available under tmp-12-11-47

and logs:

    $ ls  tmp-12-11-47/
    report-12-11-47.xml-setgettests.MembaseBucket.xml	value_100b.log

Development
-----------

When using git on Linux/OSX systems, you might run into issues where git
incorrectly believes Windows-related files have been modified. In reality, git
is merely mis-treating CRLF line endings.
Try the following...

    $ cd testrunner
    $ git config core.autocrlf false

Running End to End Tests Before Submitting a Patch to Gerrit
------------------------------------------------------------

Testrunner project has different test suites which can be run priori to
submitting the code to gerrit. There are test suites that can be run against a
single node which validates basic database operations such as persistence and
bucket management. There are also key-value clustering related test cases which
can be run against Membase/Couchbase 1.8 multiple nodes. Recently we have also
been adding more tests which validates basic view functionalities on a cluster
and on a single.

### "make e2e-kv-single-node"

This make target will start ns_server using ```cluster_run -n1``` and run all the
test cases listed in conf/py-all-dev.conf. The test runtime can vary between
15-30 minutes depending on your machine.

Testrunner prints out a human readable pass/fail of the tests. Please refer to
the "rerunning test" section for more information of how to re-run one single
test against cluster_run.

### "make test-views"

This make target will start four ns_server(s) using ```cluster_run -n4``` and
runs all the test cases listed in conf/py-view.conf. The test runtime can vary
between 30-45 minutes. py-view.conf contains basic test cases which validates
clustering operations such as rebalancing.Each test is also parametrized so you
can easily modify this run list and change the number of docs or the
load_duration.

For instance, ```test_get_view_during_x_min_load_y_working_set,num-docs=10000,load-time=1,run-view-time=1```
test will create a view, inserts 1000 documents, mutate those documents for 1
minute and run view queries in parallel to the load for 1 min. You can easily
change the parameters there to insert 1M items and keep the load running for 10
mins for example.
