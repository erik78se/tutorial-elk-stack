# Tutorial: Getting started with ELK

Difficulty: Intermediate
Author: [Erik Lönroth] and [Xinyue Mao]

# What you will learn 

* Learn how to deploy the ELK stack
* How to seld logs to it via a pyhton application
* How to add in "filebeat" to monitor a remote file log

# Preparations
* Linux client
* JUJU installed

# The ELK stack
 - Basic concept
   ELK Stack comprises of three popular projects: [Elastic Search], [Logstash] and [Kibana], which is an effective tool to help users search, analize and visualize log data in real time.
   The ELK architectrue is detaily introduced in [ELK Stack Tutorial]. 
   **Elastic Search** is a distributed, JSON-based search and analytics engine designed for horizontal scalability, maximum reliability, and easy management.
   **Logstash** is a dynamic data collection pipeline with an extensible plugin ecosystem and strong Elasticsearch synergy.
   **Kibana** is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster.

 - Deploy it from charm store
       
    ```sh
    juju deploy ~elasticsearch-charmers/bundle/elk-stack
    ```
    To test this deployment, we use:
    ```sh
    juju status
    ```
    Or we could also track juju status by using:
    ```sh
    watch juju status --color
    ```
 - Generate some noise
   When the status message of unit says they are ready, we can use the provided `generate-noise` action to test whether ELK Stack conponents work well:
   ```sh
   juju run-action logstash/0 generate-noise
   ```
   When the action is completed, we could also retrieve the resuls by using:
   ```sh
   watch juju show-action-status
   ```
   copy action id and do this:
   ```sh
   juju show-action-output <action-id>
   ```
   
 - Login kibana, setup first index
   We check JUJU structure status by using this command again:
   ```sh
   juju status
   ```
   Find unit Kibana and copy corresponding IP address, then paste into any browser.
   In this bundle, we use username:`admin` and password:`mysecret`, and hit Enter!
   Now we login to Kibana!
   
   We can see **Management** in the bottom of the left bar, choose **Management** -> **Index Patterns** -> **Create Index Pattern**
   In search box, if we input `logstash*`, then the noise generated will appear underneath. Then **Next Step**.
   We could choose filter index by **@timestamp**, and **Create Index Pattern**.
   
 - Look at the noice in the web
   We go back to see the left bar, choose **Discover** on the top. Then click the name we created for index pattern several steps ago. In this case, the option should be `logstash*`.
   Adjust the filter parameters according to your requirement, then you would see the visualized log (noise) data come out! 
   Yay~~
   


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
[ELK Stack Tutorial]: https://howtodoinjava.com/microservices/elk-stack-tutorial-example/
