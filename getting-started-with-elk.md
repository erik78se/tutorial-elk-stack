# Tutorial: Getting started with ELK

Difficulty: Intermediate
Author: [Erik Lönroth] and [Xinyue Mao]

# What you will learn 

* How to deploy the ELK stack
* How to send logs through a custom pyhton application.
* Use a [juju action] to perform workload in a charm.
* How to add the [filebeat charm] to track logs in a custom logfile.

# Preparations
* You should have a working juju client setup. (Like in this example: [Installing juju])
* You should be fairly confident deploying charms with juju.
* Basic understanding of python is good.

# The ELK stack
 - Basic Concept
   The ELK Stack comprises three projects: [Elastic Search], [Logstash] and [Kibana], which allows for search, analyse and visualize log data in real time.
   Here is a picture of the [ELK architecture]. 
   
 Lets use juju to deploy the full stack.

 - Deploy it from charm store
    *Note: The constraints are needed on some clouds since instances can get too small to start logstash unless asked for about 4G ram.*
    ```sh
    juju deploy cs:~omnivector/bundle/elk --constraints mem=4G cpu=2
    ```
    Watch the deploy status:
    ```sh
    juju status
    ```
    #TODO: Insert picture of a ready deploy.
    
    #TODO: Explain why we need to generate noice for our test...
    
 - Generate noise (for testing)
   When the deploy is ready, lets use the `generate-noise` [juju action] to test if the ELK Stack components work as they should.
   ```sh
   juju run-action logstash/0 generate-noise
   ```
   Retrieve the results from the action by:
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
[ELK architecture]: https://cdn2.howtodoinjava.com/wp-content/uploads/2017/08/ELK.jpg
[Installing juju]: https://discourse.jujucharms.com/t/installing-juju/1164
[Elastic Search]: https://jujucharms.com/u/omnivector/elasticsearch
[Logstash]: https://jujucharms.com/u/omnivector/logstash
[Kibana]: https://jujucharms.com/u/omnivector/kibana
[juju action]: södofjsdf
