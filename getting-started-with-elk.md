# Tutorial: Getting started with ELK

Difficulty: Intermediate
Author: [Erik Lönroth] and [Xinyue Mao]

## What you will learn
bla blabla 

* Learn how to deploy the ELK stack
* How to seld logs to it via a pyhton application
* How to add in "filebeat" to monitor a remote file log

# Preparations
ble ble ble

# The ELK stack
 - basic concept
 - deploy it
 - generate some noice
 - login kibana, setup first index
 - look at the noice in the web

# Send remote data via TCP/UDP
 - deploy tiny-python
 - install python-logstash
 - write a test application


```python
import logging
import logstash
import sys

host = '127.0.0.1'

test_logger = logging.getLogger('python-logstash-logger')
test_logger.setLevel(logging.INFO)
# test_logger.addHandler(logstash.LogstashHandler(host, 5000, version=1))
test_logger.addHandler(logstash.TCPLogstashHandler(host, 6000, version=1))

test_logger.error('python-logstash: test logstash error message.')
test_logger.info('python-logstash: test logstash info message.')
test_logger.warning('python-logstash: test logstash warning message.')

# add extra field to logstash message
extra = {
    'test_string': 'python version: ' + repr(sys.version_info),
    'test_boolean': True,
    'test_dict': {'a': 1, 'b': 'c'},
    'test_float': 1.23,
    'test_integer': 123,
    'test_list': [1, 2, '3'],
}

test_logger.info("python-logstash: test extra fields", extra=extra)
```

 - send logs to some index

# Adding in filebeat
 - deploy and relating to logstash
 - configure a file to monitor
 - echo to the file and watch logs

# Contributors
 - anyone that contributes should be attributed

[Erik Lönroth]: http://eriklonroth.wordpress.com
[Xinyue Mao]: http://awesome
[tiny-python]: https://jujucharms.com/new/u/erik-lonroth/tiny-python
[Getting started]: https://docs.jujucharms.com/2.5/en/getting-started
