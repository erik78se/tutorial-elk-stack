# Tutorial: Getting started with ELK

Difficulty: Intermediate

Author: [Erik Lönroth] and [Xinyue Mao]

# What you will learn 

* How to deploy the ELK stack
* How to send logs through a custom pyhton application.
* Use a [juju action] to perform workload in a charm.
* How to add the [filebeat] charm to track logs in a custom logfile.

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
    ```sh
    juju deploy cs:~omnivector/bundle/elk
    ```
    Watch the deploy status:
    ```sh
    juju status
    ```
    An example shows machines are ready:
    ![status] 
    
    
 - Generate noise (for testing)
 
   How can we test if the ELK Stack components work as expected?
   
   `generate-noise` is a build-in function generates logs for temporary use.
   
   When the deploy is ready, lets use the `generate-noise` [juju action]: 
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
   Check [kibana document] to get the login information, then hit Enter!
   Now we login to Kibana!
   
   Choose **Management** -> **Index Patterns** -> **Create Index Pattern**
   In search box, if we input `logstash*`, then the noise generated will appear underneath. Then **Next Step**.
   We could choose filter index by **@timestamp**, and **Create Index Pattern**.
   ![kibanascreenshoot]

 - Look at the noise in the web
 
   We go back to see the left bar, choose **Discover** on the top. Then click the name we created for index pattern several steps ago. In this case, the option should be `logstash*`.
   Adjust the filter parameters according to your requirement, then you would see the visualized log (noise) data come out! 

   Yay~~
   


# Send logs with python-logstash
Its now possible to send logs to logstash with you applications. To demo this, we will deploy a charm called [tiny-python]. From that tiny-python unit, we will send some logs to the ELK stack.

Start by deploying tiny-python.

``` juju deploy cs:~erik-lonroth/tiny-python ```

Now, login to the tiny-python unit which will be used to send logs from.

```juju ssh tiny-python/0```

Once logged in, install [python-logstash] with pip: 

```
sudo pip3 install python-logstash
```

Now, create a python script: my-logstash-script.py

For success, you need to figure out the IP address of the logstash unit. We leave that up to you to figure out before we move on.

The script should look like this:

```python
import logging
import logstash
import sys

# IP address of the logstash unit need to go here.
host = '<ip-of-the-logstash-unit>'

test_logger = logging.getLogger('python-logstash-logger')
test_logger.setLevel(logging.INFO)

# Port for UDP is: 5000, TCP: 6000
test_logger.addHandler(logstash.LogstashHandler(host, 5000, version=1))
# test_logger.addHandler(logstash.TCPLogstashHandler(host, 6000, version=1))

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

Good work! You are now ready to send logs to logstash!

## Execute the test
Go ahead and run the script that will send some logs over UDP to logstash.

```
python3 my-logstash-script.py
```

You should now be able go back to Kibana and add another index and see your logs coming in.

Here is an example of visualized log data in Kibana with index "logstash-":
![kibana logstash]

Great work, you should now have the basics for sending logs to logstash. 

Before you move on, why dont you modify the script to use TCP as the carrier protocol?

## Next step: logging to files

What if you have a file with logs and want to monitor that for log entries? After all, this is very common.

We can use juju again to add in [filebeat] to the equation. Filebeat is made to monitor files.

Making use of filebeat is easy with juju.

## Add filebeat
Deploy [filebeat] and relate it to the exising [tiny-python] charm, then [filebeat] charm is related to logstash. Like this:
```
juju deploy filebeat
juju relate filebeat tiny-python
juju relate filebeat logstash
```

## Decide on files to monitor
The [filebeat] charm can be configured to monitor any file. Lets change the default config and monitor the file: '/home/ubuntu/mylogfile.log'

```
juju config filebeat logpath="/home/ubuntu/mylogfile.log"
```

Now, lets write some log messages to the file:

```
echo "HELLO WORLD OF LOGS" >> /home/ubuntu/mylogfile.log
```

## Add the new index to kibana
To see the logs coming in from filebeat (your logfile), add another index (filebeat-) in kibana and you are all set.

Amazing job! You are now an ELK wizard.

# Contributors



[Erik Lönroth]: http://eriklonroth.wordpress.com
[Xinyue Mao]: http://awesome
[tiny-python]: https://jujucharms.com/new/u/erik-lonroth/tiny-python
[Getting started]: https://docs.jujucharms.com/2.5/en/getting-started
[ELK architecture]: https://cdn2.howtodoinjava.com/wp-content/uploads/2017/08/ELK.jpg
[Installing juju]: https://discourse.jujucharms.com/t/installing-juju/1164
[Elastic Search]: https://jujucharms.com/u/omnivector/elasticsearch
[Logstash]: https://jujucharms.com/u/omnivector/logstash
[Kibana]: https://jujucharms.com/u/omnivector/kibana
[juju action]: https://docs.jujucharms.com/2.5/en/actions
[filebeat]: https://jujucharms.com/filebeat
[python-logstash]: https://pypi.org/project/python-logstash/
[status]: https://github.com/erik78se/tutorial-elk-stack/blob/master/jujustatus.PNG?raw=true
[kibana document]: https://jujucharms.com/u/omnivector/kibana/
[kibanascreenshoot]: https://github.com/erik78se/tutorial-elk-stack/blob/master/kibana_screenshoot.PNG?raw=true
[kibana logstash]: https://github.com/erik78se/tutorial-elk-stack/blob/master/kibana_logstash_pythonlogger.PNG?raw=true
