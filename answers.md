Your answers to the questions go here.
# DataDog Pre-Sales Engineer Exercise

## Intro
I really enjoyed getting the time to play around with the Datadog stack. I decided I'd go outside my comfort zone and go along with the recommended environment and use vagrant and VirtualBox. I decided to keep Azure for work. 

### Prerequisites (Setting up the environment)

First thing I did was take a quick read through the guides and see what the assessment consisted of. 

Setting up test environment.

Assessment:
1. Collecting Metrics
2. Visualizing Data
3. Monitoring Data
4. Collecting APM Data
5. Final/Bonus Questions

I prefer breaking out the sections with the items in small chucks and reading through the relevant documentation section by section rather than all at once and reading each section in depth when I go to each challenge. So, I briefly read through the assessment and then went back and set up the environment. For setting up the environment, I opted to use Vagrant and VirtualBox because it was the recommended option, so I guessed that's what the team use when working at Datadog.  I've never used Vagrant or VirtualBox before this challenge, it was fairly straight forward and easy to use.

The version of Ubuntu I used was 18.04. 

I then created my Datadog account and connected the agent to my new VM by following the easy steps here: [Setting up the Datadog Agent - Ubuntu](https://app.datadoghq.com/account/settings#agent/ubuntu)

I ran through the following page and decided to leave it open to get the agent commands [Datadog Agent commands](https://docs.datadoghq.com/agent/guide/agent-commands/?tab=agentv6v7)

I quickly ran a the status check below, just to make sure everything was running and going into the datadog portal online. So, everything is up and running. Let's get started.
```
sudo datadog-agent status
```


 [Datadog-agent status](https://imgur.com/a/zx2P8Mq)
 
 
## Section 1: Collecting Metrics
### Q1. Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

I took a quick read on the following link: [Datadog Tagging](https://docs.datadoghq.com/tagging/)

In order to add tags, I needed to change the yaml file, so I got the file path from the status command I ran above, /etc/datadog-agent/datadog.yaml once I had this, I ran the below command to get change the yaml file. I changed the hostname and tags. 
```
sudo vim /etc/datadog-agent/datadog.yaml 
```

#### Config changes
    Hostname: owen.barr

    Tags: 
      -james:bond
      - donegal:gaa
  
Once this was done, we have to restart the agent with the below command and wait for a few minutes to see it on the hostmap. [Datadog Host Map](https://app.datadoghq.eu/infrastructure/map)
 ```
sudo service datadog-agent restart
 ```
  [Datadog Yaml File](https://imgur.com/T9cJkpY)
  
  [Datadog Host Map](https://imgur.com/6cbVh2C)



### Q2. Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

I went with MySQL as it's what I'm most familiar with. I took a quick look through the documentation provided: [Datadog Integrations MySQL](https://app.datadoghq.eu/account/settings#integrations/mysql) It was a pretty straight forward integration. I'd probably add in a line or two around the installation of the chosen DB on your OS of choice, but I get there are a lot of permutations out there. 
So, I ran a few commands below to get MySQL installed.

  #### Install MySQL Server on the Ubuntu operating system
    sudo apt-get install mysql-server
    sudo mysql -u root

Now that is completed, I connect Datadog to the MySQL server, using the steps I followed in the following page: [Datadog Integrations MySQL](https://app.datadoghq.eu/account/settings#integrations/mysql)

I followed the steps above and created the user called Datadog and gave it the appropriate permissions.

#### SQL commands
```
CREATE USER 'datadog'@'localhost' IDENTIFIED BY '<UNIQUEPASSWORD>';
```

I verified the user was created successfully using the following commands - and replacing <UNIQUEPASSWORD> with the password I created when creating the user.

```
mysql -u datadog --password='REPLACE_WITH_YOUR_PASSWORD' -e "show status" | grep Uptime && echo -e "\033[0;32mMySQL user - OK\033[0m" || echo -e "\033[0;31mCannot connect to MySQL\033[0m"
```
Then I ran the following:
```
mysql -u datadog --password=<UNIQUEPASSWORD> -e "show slave status" && \ echo -e "\033[0;32mMySQL grant - OK\033[0m" || \
echo -e "\033[0;31mMissing REPLICATION CLIENT grant\033[0m"
  ```
This gave back an error until I ran the below command:
```
GRANT REPLICATION CLIENT ON *.* TO 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;
```
Once the above command was run, I ran the following command again, 
``` 
mysql -u datadog --password=<UNIQUEPASSWORD> -e "show slave status" && \ echo -e "\033[0;32mMySQL grant - OK\033[0m" || \ echo -e "\033[0;31mMissing REPLICATION CLIENT grant\033[0m" 
```

I saw that the user was created with the appropriate permissions. 

 [User permissions](https://imgur.com/Fxs1Xb5)
  
I ran the following SQL commands, went to the Metrics explorer and saw that the spikes from running a few queries.
```
mysql> show databases like 'performance_schema';

mysql> GRANT SELECT ON performance_schema.* TO 'datadog'@'localhost';
 ```
 [SQL Monitor Metrics](https://imgur.com/vEVnaqR)
  

### Q3. Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

I followed the documentation outlined here:  [Agent Checks](https://docs.datadoghq.com/developers/agent_checks/) and [Write Checks](https://docs.datadoghq.com/developers/write_agent_check/?tab=agentv6v7)

It was pretty straight forward to follow, practically all the code was already written for me. So, I just followed the following steps:
```
sudo vim /etc/datadog-agent/conf.d/hello.yaml
```
I inserted the following into the yaml file, saved and closed:

#### Hello yaml file
```
instances: [{}] 
```

I then ran the below command to create hello.py: 
```
sudo vim /etc/datadog-agent/checks.d/hello.py
```
Which allowed me to add the following code into the python file.

#### Python Code
```
The following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from new versions of the Agent...
    from datadog_checks.base import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version < 6.6.0
    from checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"

class HelloCheck(AgentCheck):
    def check(self, instance):
        self.gauge('hello.world', 1, tags=['TAG_KEY:TAG_VALUE'])
```

I restarted the agent and was able to see the update on the metrics explorer:
 [Hello.py Metrics](https://imgur.com/SlvAQhz)
  
Once the test metric was created, I started on the new metric that reports a random number from 0-1000. I used the contents from the previous yaml file but created a new file with the following command: 
```
sudo vim /etc/datadog-agent/conf.d/my_metric.yaml
```
I did the same command to create the py file and inputted the below into it: 
```
sudo vim /etc/datadog-agent/checks.d/my_metric.py 
```
#### Python code
```
import random  # imports the random module for python
from checks import AgentCheck

class RandomCheck(AgentCheck): # same as documentation just change HelloCheck to RandomCheck
     def check(self, instance): # same as hello.py file
         self.gauge('my_metric', random.randint(0, 1000)) #changed the name to "my_metric" and  returning a random Integer from 0 to 1000, 
```

The below links will help anyone understand the random module in python:
1. [Python Random Libary]("https://docs.python.org/3/library/random.html")
2. [Python Generate Random Numbers]("https://docs.python.org/3.1/library/random.html")


Now that I was able to create a check, I was ready to create the new metric that reports a random number from 0-1000. I used the exact same yaml file as from the hello.world check but I renamed it to my_metric.yaml.

The my_metric python code is also very similar to the hello.world code but I added/modified a few things. First, I wanted to change the name of the metric from hello.world to my_metric.

#### Python code for My Metric
```
self.gauge('my_metric', 1)
```

Then, instead of reporting a constant 1, I wanted to generate a random number from 0-1000. To do that, I needed to import the "random" library into python by adding the line "import random" to the top of our code. I can generate a random number between 0-1000 with the randint member of the random class "random.randint(0,1000))" My final code looks like the following:


#### Python code for Random number generator
```
import random
from checks import AgentCheck
class RandomCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(0,1000))
```

Then, I restarted the Datadog agent by running the command:
```
sudo service datadog-agent restart
```
I finally checked my new metric "my_metric" in the metrics explorer and confirmed that our check is successfully generating random numbers between 0-1000.

[Metric Explorer My_Metric](https://imgur.com/E29gZs3)


### Q4. Change your check's collection interval so that it only submits the metric once every 45 seconds.
If you go to Collection interval section of the above file, it gives you the code to change the collection interval. 
To do so, insert this into your my_metric yaml file. 
```
sudo vim /etc/datadog-agent/conf.d/my_metric.yaml
```
Make sure you change the 30 to 45.


#### Collection interval code - 45 Seconds
```
init_config:

instances:
   -  min_collection_interval: 45
```

### Q5. Can you change the collection interval without modifying the Python check file you created?

I referred to this article when answering the bonus question: [Write Agent Check](https://docs.datadoghq.com/developers/write_agent_check/?tab=agentv6v7) 

I can change the collection interval by altering the below yaml file at the min_collection interval variable. In the below I changed it from 45 seconds to 60 seconds. This means there is no need to alter my python code.

#### Collection interval code - 60 Seconds
```
init_config:

instances:
   -  min_collection_interval: 60
   ```
Now I have changed the collection interval, I restart the agent with the following code.
```
sudo service datadog-agent restart
```

In the documentation there is a caveat:

Note: If the min_collection_interval is set to 30, it does not mean that the metric is collected every 30 seconds, but rather that it could be collected as often as every 30 seconds. The collector will try to run the check every 30 seconds, but the check might need to wait in line depending on how many integrations are enabled on the same Agent. Also, if the check method takes more than 30 seconds to finish, the Agent will notice the check is still running and will skip its execution until the next interval.

As per the documentation, it is important to note that this does not guarantee a collection every 45 seconds. Instead, it means that this particular check *may be collected* as often as every 45 seconds. 

You can see from the metric explorer that this has successfully been deployed.

[Metric explorer Collection interval](https://imgur.com/wge5uFi)
  

## Section 2: Visualizing Data
First, we start off with the requirements:

Utilize the Datadog API to create a Timeboard that contains:

1. Your custom metric scoped over your host.
2. Any metric from the Integration on your Database with the anomaly function applied.
3. Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket.

In order to accomplish this task, I first installed the Datadog python library by following the instructions on the Datadog Python GitHub: [Datadog Python](https://github.com/DataDog/datadogpy)

I ran the command:
```
sudo pip3 install datadog
```

At this point I was ready to begin writing the Timeboard code in python. To do this, I referenced the following article:
[Datadog Timeboards](https://docs.datadoghq.com/api/?lang=python#timeboards)

I then wanted to compile this code, but I needed to get the API keys from Datadog. To obtain them, I went to the API tab on the integrations menu and generated a new key by selecting "Create Application Key":

The code I saved was taken from the Datadog documentation and is seen below:

#### Python code for Creating a timeboard
```python
from datadog import initialize, api
options = {
    'api_key': '<YOUR_API_KEY>',
    'app_key': '<YOUR_APP_KEY>'
}
initialize(**options)
title = "My Timeboard"
description = "An informative Timeboard."
graphs = [{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:system.mem.free{*}"}
        ],
        "viz": "timeseries"
    },
    "title": "Average Memory Free"
}]
template_variables = [{
    "name": "host1",
    "prefix": "host",
    "default": "host:my-host"
}]
read_only = True
api.Timeboard.create(title=title,
                     description=description,
                     graphs=graphs,
                     template_variables=template_variables,
                     read_only=read_only)                     
```
I had an issue where the data wasn't sending from "my_timeboard.py" or "my_screenboard.py", so it wasn't creating a new dashboard for me. 

I went to try and look what were the reasons why it wasn't working. 

I ran the below commands and seen the below:
```
tail -f /var/log/datadog/agent.log 
```
  #### Datadog agent logs
```  
2020-06-07 16:57:35 UTC | CORE | WARN | (pkg/collector/python/datadog_agent.go:118 in LogMessage) | mysql:1fa1a2e54ef75ec6 | (mysql.py:1295) | Failed to fetch records from the perf schema                                  'events_statements_summary_by_digest' table.
2020-06-07 16:57:35 UTC | CORE | WARN | (pkg/collector/python/datadog_agent.go:118 in LogMessage) | mysql:1fa1a2e54ef75ec6 | (mysql.py:1324) | Failed to fetch records from the perf schema                                  'events_statements_summary_by_digest' table.
```     
This error occurs twice. I could see there was an error with the MySQL server, so I went and checked to see if there were any errors with any of the config files. 

There was a spelling mistake in one of the config files, so I fixed that and restarted the agent. 

#### Config Error
```
vagrant@vagrant:~$ cat /etc/datadog-agent/conf.d/mysql.d/conf.yaml 
nit_config: ## should be init_config

instances:
  - server: 127.0.0.1
    user: datadog
    pass: "Jamesbond1!" # from the CREATE USER step earlier
    port: 3306 # e.g. 3306
    options:
      replication: false
      galera_cluster: true
      extra_status_metrics: true
      extra_innodb_metrics: true
      extra_performance_metrics: true
      schema_size_metrics: false
      disable_innodb_metrics: false
```      
      
Still not working, so I went back to the logs. 


I could see it is down to one now that I have fixed the yaml file. I ran through all the other directories to make sure everything was fine. Everything else looked fine. So, I went and to see if there was anything else blocking this like port forwarding. 

I turned it on and checked but it made no difference, so I reverted back to default. 

#### Turning on port forwarding 
```
$ cat /proc/sys/net/ipv4/ip_forward
0
$ sysctl net.ipv4.ip_forward=1
sysctl: permission denied on key 'net.ipv4.ip_forward'
$ sudo !!
sudo sysctl net.ipv4.ip_forward=1

netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:mysql         localhost:48108         TIME_WAIT  
tcp        0      0 localhost:mysql         localhost:48112         TIME_WAIT  
tcp        0      0 localhost:mysql         localhost:48102         TIME_WAIT  
tcp        0      0 localhost:mysql         localhost:48110         TIME_WAIT  
tcp        0      0 vagrant:ssh             _gateway:58885          ESTABLISHED
tcp        0      0 vagrant:47440           23.172.107.34.bc.:https ESTABLISHED
udp        0      0 localhost:46898         localhost:8125          ESTABLISHED
udp        0      0 localhost:33425         localhost:8125          ESTABLISHED
Active UNIX domain sockets (w/o servers)
```

I then went to look at the MySQL server directly in order to make sure that it was working correctly. 

Everything looked ok, I ran a few commands and the below is what was seen. 

#### SQL code
```
mysql> select * from events_statements_summary_by_digest;
ERROR 1046 (3D000): No database selected
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
```
mysql> use performance_schema;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker start-up with -A

Database changed
```
```
mysql> select * from events_statements_summary_by_digest;
+--------------------+----------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+----------------+----------------+----------------+---------------+------------+--------------+-------------------+---------------+-------------------+-----------------------------+------------------------+----------------------+----------------------------+------------------+------------------------+-----------------+-----------------------+----------------+---------------+---------------+-------------------+------------------------+---------------------+---------------------+
| SCHEMA_NAME        | DIGEST                           | DIGEST_TEXT                                                                                                                                                                                                                                                                                                                                           | COUNT_STAR | SUM_TIMER_WAIT | MIN_TIMER_WAIT | AVG_TIMER_WAIT | MAX_TIMER_WAIT | SUM_LOCK_TIME | SUM_ERRORS | SUM_WARNINGS | SUM_ROWS_AFFECTED | SUM_ROWS_SENT | SUM_ROWS_EXAMINED | SUM_CREATED_TMP_DISK_TABLES | SUM_CREATED_TMP_TABLES | SUM_SELECT_FULL_JOIN | SUM_SELECT_FULL_RANGE_JOIN | SUM_SELECT_RANGE | SUM_SELECT_RANGE_CHECK | SUM_SELECT_SCAN | SUM_SORT_MERGE_PASSES | SUM_SORT_RANGE | SUM_SORT_ROWS | SUM_SORT_SCAN | SUM_NO_INDEX_USED | SUM_NO_GOOD_INDEX_USED | FIRST_SEEN          | LAST_SEEN           |
+--------------------+----------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+----------------+----------------+----------------+---------------+------------+--------------+-------------------+---------------+-------------------+-----------------------------+------------------------+----------------------+----------------------------+------------------+------------------------+-----------------+-----------------------+----------------+---------------+---------------+-------------------+------------------------+---------------------+---------------------+
| NULL               | a8c543810b049d7c74ceac044358180b | SET `AUTOCOMMIT` = ?                                                                                                                                                                                                                                                                                                                                  |        573 |    42803906000 |       48805000 |       74701000 |      522812000 |             0 |          0 |            0 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 30b3a58f8f2701c69650acbe899cf4e6 | SELECT `VERSION` ( )                                                                                                                                                                                                                                                                                                                                  |        573 |    37716911000 |       42748000 |       65823000 |      423947000 |             0 |          0 |            0 |                 0 |           573 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 011da6fa0d4fdfe1518ad83484e620e1 | SHOW GLOBAL STATUS                                                                                                                                                                                                                                                                                                                                    |        573 |   631532828000 |      740291000 |     1102151000 |     6877713000 |   48853000000 |          0 |            0 |                 0 |        203988 |            407976 |                           0 |                    573 |                    0 |                          0 |                0 |                      0 |            1146 |                     0 |              0 |             0 |             0 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 44e3fa936de0773d12e18758e0f18ae1 | SHOW GLOBAL VARIABLES                                                                                                                                                                                                                                                                                                                                 |        573 |   786617580000 |     1007540000 |     1372805000 |    13583987000 |   41508000000 |          0 |            0 |                 0 |        292230 |            584460 |                           0 |                    573 |                    0 |                          0 |                0 |                      0 |            1146 |                     0 |              0 |             0 |             0 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 21bcbccc6a121df9181d4a3add23509f | SELECT ENGINE FROM `information_schema` . `ENGINES` WHERE ENGINE = ? AND `support` != ? AND `support` != ?                                                                                                                                                                                                                                            |        573 |    88198758000 |      109148000 |      153924000 |     1246082000 |   46878000000 |          0 |            0 |                 0 |           573 |              5157 |                           0 |                    573 |                    0 |                          0 |                0 |                      0 |             573 |                     0 |              0 |             0 |             0 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 5ec7cfca899e59c919b6b74bc3775837 | SHOW ENGINE `INNODB` STATUS                                                                                                                                                                                                                                                                                                                           |        573 |   115632756000 |      152205000 |      201802000 |      744189000 |             0 |          0 |            0 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 33945ff3f1bec2aae610f6e43f0d813e | SELECT `avg_us` , `ro` AS `percentile` FROM ( SELECT `avg_us` , @? := @? + ? AS `ro` FROM ( SELECT `ROUND` ( `avg_timer_wait` / ? ) AS `avg_us` FROM `performance_schema` . `events_statements_summary_by_digest` ORDER BY `avg_us` ASC ) `p` , ( SELECT @? := ? ) `r` ) `q` WHERE `q` . `ro` > `ROUND` ( ? * @? ) ORDER BY `percentile` ASC LIMIT ?  |        573 |   245677213000 |      322323000 |      428756000 |     1913516000 |   73596000000 |          0 |            0 |                 0 |           476 |             36284 |                           0 |                   1719 |                    0 |                          0 |                0 |                      0 |            1719 |                     0 |              0 |          9428 |          1146 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 0578d2c44e123881c891f646d41bbaca | SELECT SCHEMA_NAME , `ROUND` ( ( SUM ( `sum_timer_wait` ) / SUM ( `count_star` ) ) / ? ) AS `avg_us` FROM `performance_schema` . `events_statements_summary_by_digest` WHERE SCHEMA_NAME IS NOT NULL GROUP BY SCHEMA_NAME                                                                                                                             |        573 |   149028935000 |      160924000 |      260085000 |    13846412000 |   44799000000 |          0 |            0 |                 0 |           169 |              9291 |                           0 |                    573 |                    0 |                          0 |                0 |                      0 |             573 |                     0 |              0 |           169 |           573 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 6909bda371af7c36751d55c3ca80a467 | SHOW VARIABLES LIKE ?                                                                                                                                                                                                                                                                                                                                 |        573 |   644137956000 |      873424000 |     1124150000 |     2542901000 |   39545000000 |          0 |            0 |                 0 |           573 |            600504 |                           0 |                    573 |                    0 |                          0 |                0 |                      0 |            1146 |                     0 |              0 |             0 |             0 |               573 |                      0 | 2020-06-07 16:38:42 | 2020-06-07 19:02:09 |
| NULL               | 271e1927e3681d4a6b138e6906f8e3de | SELECT @@`version_comment` LIMIT ?                                                                                                                                                                                                                                                                                                                    |          5 |      692859000 |       68300000 |      138571000 |      247685000 |             0 |          0 |            0 |                 0 |             5 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 17:02:54 | 2020-06-07 19:01:12 |
| NULL               | c77bfcb67e8c01701e5bfc330743de22 | SHOW STATUS                                                                                                                                                                                                                                                                                                                                           |          1 |      732714000 |      732714000 |      732714000 |      732714000 |      69000000 |          0 |            0 |                 0 |           359 |               718 |                           0 |                      1 |                    0 |                          0 |                0 |                      0 |               2 |                     0 |              0 |             0 |             0 |                 1 |                      0 | 2020-06-07 17:02:54 | 2020-06-07 17:02:54 |
| NULL               | 16cbe2fa99c109ca45db93dc09aae3bd | SHOW SLAVE STATUS                                                                                                                                                                                                                                                                                                                                     |          1 |      980828000 |      980828000 |      980828000 |      980828000 |             0 |          0 |            0 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 17:03:15 | 2020-06-07 17:03:15 |
| NULL               | 972d39a24a215cc042e20025af862989 | GRANT REPLICATION CLIENT ON * . * TO ? @? WITH MAX_USER_CONNECTIONS ?                                                                                                                                                                                                                                                                                 |          1 |     2923501000 |     2923501000 |     2923501000 |     2923501000 |     272000000 |          0 |            1 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 17:08:19 | 2020-06-07 17:08:19 |
| NULL               | 93d6e744f0de4e0cfe0b8d1cf72eb8c1 | GRANT PROCESS ON * . * TO ? @?                                                                                                                                                                                                                                                                                                                        |          1 |      205110000 |      205110000 |      205110000 |      205110000 |     129000000 |          0 |            0 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 17:08:27 | 2020-06-07 17:08:27 |
| NULL               | 0194ada02c5e9d8c67bf6ffce36979a9 | ALTER SYSTEM_USER ? @? WITH MAX_USER_CONNECTIONS ?                                                                                                                                                                                                                                                                                                    |          1 |      351257000 |      351257000 |      351257000 |      351257000 |     132000000 |          0 |            0 |                 0 |             0 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 17:08:34 | 2020-06-07 17:08:34 |
| NULL               | 6d711d488f3ed491a0d7610aa0d65038 | SHOW SCHEMAS                                                                                                                                                                                                                                                                                                                                          |          2 |     4353974000 |      739321000 |     2176987000 |     3614653000 |     487000000 |          0 |            0 |                 0 |             8 |                 8 |                           0 |                      2 |                    0 |                          0 |                0 |                      0 |               2 |                     0 |              0 |             0 |             0 |                 2 |                      0 | 2020-06-07 18:19:06 | 2020-06-07 19:01:54 |
| NULL               | 2cf2906bafd55ee74be1b693d8234ae8 | SELECT SCHEMA ( )                                                                                                                                                                                                                                                                                                                                     |          2 |     2731988000 |      203722000 |     1365994000 |     2528266000 |             0 |          0 |            0 |                 0 |             2 |                 0 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               0 |                     0 |              0 |             0 |             0 |                 0 |                      0 | 2020-06-07 18:20:04 | 2020-06-07 19:02:13 |
| performance_schema | 6d711d488f3ed491a0d7610aa0d65038 | SHOW SCHEMAS                                                                                                                                                                                                                                                                                                                                          |          2 |      707518000 |      234190000 |      353759000 |      473328000 |     155000000 |          0 |            0 |                 0 |             8 |                 8 |                           0 |                      2 |                    0 |                          0 |                0 |                      0 |               2 |                     0 |              0 |             0 |             0 |                 2 |                      0 | 2020-06-07 18:20:04 | 2020-06-07 19:02:13 |
| performance_schema | 99477144cf0254f505964804b180db5c | SHOW TABLES                                                                                                                                                                                                                                                                                                                                           |          2 |     2596195000 |     1140707000 |     1298097000 |     1455488000 |      59000000 |          0 |            0 |                 0 |           174 |               174 |                           0 |                      2 |                    0 |                          0 |                0 |                      0 |               2 |                     0 |              0 |             0 |             0 |                 2 |                      0 | 2020-06-07 18:20:04 | 2020-06-07 19:02:13 |
| performance_schema | 4e7ab33d008810fc923af1f6ee58f58a | SELECT * FROM `events_statements_summary_by_digest`                                                                                                                                                                                                                                                                                                   |          1 |     2138672000 |     2138672000 |     2138672000 |     2138672000 |     142000000 |          0 |            0 |                 0 |            19 |                19 |                           0 |                      0 |                    0 |                          0 |                0 |                      0 |               1 |                     0 |              0 |             0 |             0 |                 1 |                      0 | 2020-06-07 18:20:25 | 2020-06-07 18:20:25 |
| performance_schema | dcccbb8f13c4aee22ce880dd44b1649f | SHOW GLOBAL VARIABLES LIKE ?                                                                                                                                                                                                                                                                                                                          |          1 |     1660447000 |     1660447000 |     1660447000 |     1660447000 |     180000000 |          0 |            0 |                 0 |             1 |              1020 |                           0 |                      1 |                    0 |                          0 |                0 |                      0 |               2 |                     0 |              0 |             0 |             0 |                 1 |                      0 | 2020-06-07 18:21:56 | 2020-06-07 18:21:56 |
+--------------------+----------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+----------------+----------------+----------------+---------------+------------+--------------+-------------------+---------------+-------------------+-----------------------------+------------------------+----------------------+----------------------------+------------------+------------------------+-----------------+-----------------------+----------------+---------------+---------------+-------------------+------------------------+---------------------+---------------------+
21 rows in set (0.00 sec)
```
```
mysql> show global variables like 'PORT';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+
1 row in set (0.00 sec)
```

The ports looked fine, the user permissions were correct and I could see the data in the DB.

I decided I would type out the python script I had prepared for the next part anyway. 

I broke the python file into separate sections of code for each task that is asked:

1. Your custom metric scoped over your host.
2. Any metric from the Integration on your Database with the anomaly function applied.
3. Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket.


#### Python code
```
from datadog import initialize, api
options = {
    'api_key': 'API_Keys ',
    'app_key': 'My_APP_Keys'
}
##I use the same code as provided by Datadog

initialize(**options)
title = "Datadog Assignment Timeboard"
description = "This would be my datadog Timeboard!"
graphs = [{
```
  ### Task 1: Your custom metric scoped over your host.
  ```
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:my_metric{*}"} ## I would have used the my_metric from previous exercises
        ],
        "viz": "timeseries"
    },
    "title": "my_metric"
},
```
  ###  Task 2: Any metric from the Integration on your Database with the anomaly function applied.
  
  I took a look into the [Datadog Miscellaneous Functions](https://docs.datadoghq.com/graphing/miscellaneous/functions/) for functions, [Datadog Algorithms](https://docs.datadoghq.com/dashboards/functions/algorithms/) and [Datadog Anomaly](https://docs.datadoghq.com/monitors/monitor_types/anomaly/#anomaly-detection-algorithms) for anomalies. 
  
An example I found for anomalies was the following: (METRIC_NAME>{*}, '<ALGORITHM>', <BOUNDS>) and it is what I needed, so I chose the "Mysql.performance.com" for the metric name, "basic" for algorithm (as there is no seasonal repeating), and "1" for bounds.
```
 
   "definition": {
        "events": [],
        "requests": [
            {"q": "anomalies(avg:mysql.performance.com_select{*}, 'basic', 1)"}
        ],
        "viz": "timeseries"
    },
    "title": "SQL Select Anomalies"    
}
```
### Task 3: Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket.
I needed to look at the rollup function for the next part: [Datadog Functions](https://docs.datadoghq.com/dashboards/functions/rollup/)
 
The first thing we are told is that the ".rollup()" function at the end of a query allows you to perform custom time aggregation, i.e. this function enables you to define: They give a few examples in the piece

1. .rollup(sum,120)
2. .rollup(avg,86400)

We're looking for an hour, so that's 3600 seconds and they tell us to use sum, so we use .rollup(sum, 3600)
```
{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:my_metric{*}.rollup(sum, 3600)"}
        ],
        "viz": "timeseries"
    },
    "title": "my_metric rollup"
    
}
]
template_variables = [{
    "name": "host1",
    "prefix": "host",
    "default": "host:my-host"
}]
read_only = True
api.Timeboard.create(title=title,
                     description=description,
                     graphs=graphs,
                     template_variables=template_variables,
                     read_only=read_only)
```
I don't know if all of the above works due to the error I had but reading the documentation and looking at examples they gave, made it easy to understand. 

I couldn't do the below due to the Timeboard not being created:

 1. Set the Timeboard's timeframe to the past 5 minutes
 2. Take a snapshot of this graph and use the @ notation to send it to yourself.
 3. Bonus Question: What is the Anomaly graph displaying?

I could have done it through the UI, but I don't think that was the point of the challenge.

## Section 3: Monitoring data

This was one of the easiest sections of the assessment. I went to the "Monitors" section on the Datadog page and selected "New Monitor", then clicked Metric and selected "my_metric"

[Datadog Create a Monitor](https://app.datadoghq.com/monitors#/create)

We can see the requirements of the new metric monitor are to be:

1. Warning threshold of 500
2. Alerting threshold of 800
3. And also ensure that it will notify you if there is No Data for this query over the past 10m.
4. Send you an email whenever the monitor triggers.
5. Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
6. When this monitor sends you an email notification, take a screenshot of the email that it sends you.
  
I input the requirements for the monitoring. It's easy to follow and you can see the images below

### Setting up the Monitoring 

  1. [Setting up the monitor Part 1](https://imgur.com/FuTFlat)
  2. [Setting up the monitor Part 2](https://imgur.com/TWnJKqV)
  3. [Setting up the monitor Part 3](https://imgur.com/TWnJKqV)
  4. [Setting up the monitor Part 4](https://imgur.com/5nVKY0h)
 
### Email alerts
Here are the alert emails I received:
 1. [Monitoring Email](https://imgur.com/0iEKQ0V)
 2. [No data Email](https://imgur.com/a/1YSuASb)

### Scheduling Downtime
I scheduled the downtime from 7pm to 9am daily on Monday to Friday and then one that silences it on the weekend by clicking the schedule downtime button near the top. You can also go to setup down time by the following link: [Datadog Downtime](https://app.datadoghq.eu/monitors#/downtime)

[MonitoringDowntimeWeekday](https://imgur.com/tHivZmn)
I had an issue with setting the downtime to midnight on a Sunday, it gave an error that wouldn't let me change the time to midnight  [Midnight Error](https://imgur.com/2KCD8Dy)
 
I just changed it to after the current time and let it run for Saturday and Sunday
[MonitoringDowntimeWeekend](https://imgur.com/jyykAA4)

On Monday, I was able to change it back to midnight: 
[Bug_fixed](https://imgur.com/a/1UVYfcj) There must be an error that won't let you change the  time to set an alert before the current time.

### Downtime Emails
Here are the emails I received after creating the downtime:

1. [MonitoringDowntimeEmail](https://imgur.com/CBuV0aQ) UTC is an hour behind GMT, so it reads 6 to 8
2. [MonitoringDowntimeEmail2](https://imgur.com/a/1UVYfcj) Same as above, but it reads 11pm UTC




## Section 4: Collecting APM Data
Given the following Flask app (or any Python/Ruby/Go app of your choice) instrument this using Datadog’s APM solution:

First, I needed to install ddtrace. I did this by using pip and the following command: 
```
`sudo pip3 install --upgrade ddtrace` 
```
It gave me an error, so I had to run through a few steps just to install ddtrace.
```
sudo pip3 install --upgrade ddtrace
The directory '/home/vagrant/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/home/vagrant/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Collecting ddtrace
  Downloading https://files.pythonhosted.org/packages/29/e0/3926742b5c283fc90df1b98107a2881fceaa713a9306a6f28c36003fa925/ddtrace-0.38.1.tar.gz (887kB)
    100% |████████████████████████████████| 890kB 1.7MB/s 
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-tdx86tk0/ddtrace/setup.py", line 9, in <module>
        from Cython.Build import cythonize  # noqa: I100
    ModuleNotFoundError: No module named 'Cython'
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-tdx86tk0/ddtrace/
```
I read that I had to run the below command to upgrade pip
```
sudo pip3 install --upgrade pip
```
I then ran:
```
sudo pip3 install --upgrade ddtrace

WARNING: The directory '/home/vagrant/.cache/pip' or its parent directory is not owned or is not writable by the current user. The cache has been disabled. Check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Collecting ddtrace
  Downloading ddtrace-0.38.1-cp36-cp36m-manylinux2010_x86_64.whl (997 kB)
     |████████████████████████████████| 997 kB 1.4 MB/s 
Collecting msgpack>=0.5.0
  Downloading msgpack-1.0.0-cp36-cp36m-manylinux1_x86_64.whl (274 kB)
     |████████████████████████████████| 274 kB 8.5 MB/s 
Installing collected packages: msgpack, ddtrace
Successfully installed ddtrace-0.38.1 msgpack-1.0.0
```
Once installed I moved onto the next part:

I wanted to read more into Flask, so I looked in the following pages: 
1. [Flask Install](https://flask.palletsprojects.com/en/1.1.x/installation/#install-create-env)
2. [Flask Quickstart](https://flask.palletsprojects.com/en/1.1.x/quickstart/)

I ran a few of the commands below to setup a virtual environment
```
vagrant@vagrant:~$ mkdir myprojectdatadog
vagrant@vagrant:~$  cd myprojectdatadog
vagrant@vagrant:~/myprojectdatadog$ python3 -m venv venv
vagrant@vagrant:~/myprojectdatadog$ . venv/bin/activate
(venv) vagrant@vagrant:~/myprojectdatadog$  pip3 install Flask
```
I saw Flask was installed and I did a version check. 
```
      (venv) vagrant@vagrant:~/myprojectdatadog$ flask --version
       Python 3.6.9
       Flask 1.1.2
       Werkzeug 1.0.1
```

I created my python file and copied the code provided into it. 
```
sudo vim my_flaskapp.py
```
```
## Python code
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')
```    

```      
(venv) vagrant@vagrant:~/myprojectdatadog$  export FLASK_APP=my_flaskapp.py
(venv) vagrant@vagrant:~/myprojectdatadog$ flask run
```

I recieved the below error, so I did a quick search on how to fix this:
```
WARNING: This is a development server. Do not use it in a production deployment.
```
I ran the below to change it to prod: 
```
(venv) vagrant@vagrant:~/myprojectdatadog$ export FLASK_ENV=development
```
I then ran flask with the below:    
```
(venv) vagrant@vagrant:~/myprojectdatadog$ flask run
```

The above command returned this error:
```
OSError: [Errno 98] Address already in use
```

I ran the below based on this documentation: [Flask Errno 98](https://medium.com/@tessywangari05/oserror-errno-98-address-already-in-use-flask-error-ccbff65e2bb5)
```
(venv) vagrant@vagrant:~/myprojectdatadog$  ps -fA | grep flask
vagrant  16433  1630  0 14:31 pts/0    00:00:00 grep --color=auto flask
(venv) vagrant@vagrant:~/myprojectdatadog$ kill 1630 ## PID from above
(venv) vagrant@vagrant:~/myprojectdatadog$ flask run
* Serving Flask app "my_flaskapp.py" (lazy loading)
* Environment: development
* Debug mode: on
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
2020-06-08 14:32:52,321 - werkzeug - INFO -  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
* Restarting with stat
2020-06-08 14:32:52,326 - werkzeug - INFO -  * Restarting with stat
* Debugger is active!
* Debugger PIN: 993-124-556
2020-06-08 14:32:52,520 - werkzeug - INFO -  * Debugger PIN: 993-124-556
```

I tried to run the below commands to see if I could see what was happening with the code:
```
lsof -i :5050
netstat -tulpn
netstat -van | grep 8126.
```

I couldn't figure out what had happened so, I decided to go back and try install the DD-trace recommended on [Datadog Tracing](https://docs.datadoghq.com/tracing/setup/python/)
```
(venv) vagrant@vagrant:/etc/datadog-agent/conf.d$ sudo pip3 install --upgrade pip
(venv) vagrant@vagrant:~/myprojectdatadog$ sudo pip3 install --upgrade ddtrace
```

I added the below from the [Datadog Tracer code python](https://github.com/DataDog/dd-trace-py/blob/master/ddtrace/opentracer/tracer.py) to my_flaskapp.py

#### Python code - Import DDTrace
```
import ddtrace
from ddtrace import Tracer as DatadogTracer
```

Then I ran the below command:
```
ddtrace-run python3 my_flaskapp.py
```
#### Errors when running my flask app with DDtrace
```
(venv) vagrant@vagrant:~/myprojectdatadog$ ddtrace-run python3 my_flaskapp.py
   2020-06-08 14:52:25,913 DEBUG [ddtrace.internal.import_hooks] [import_hooks.py:136] - No hooks registered for module 'jinja2.ext'
2020-06-08 14:52:25,913 - ddtrace.internal.import_hooks - DEBUG - No hooks registered for module 'jinja2.ext'
 * Serving Flask app "my_flaskapp" (lazy loading)
 * Environment: development
 * Debug mode: on
2020-06-08 14:52:25,920 INFO [werkzeug] [_internal.py:113] -  * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
2020-06-08 14:52:25,920 - werkzeug - INFO -  * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
2020-06-08 14:52:25,921 INFO [werkzeug] [_internal.py:113] -  * Restarting with stat
2020-06-08 14:52:25,921 - werkzeug - INFO -  * Restarting with stat
2020-06-08 14:52:26,161 DEBUG [ddtrace.internal.import_hooks] [import_hooks.py:136] - No hooks registered for module 'jinja2.ext'
2020-06-08 14:52:26,161 - ddtrace.internal.import_hooks - DEBUG - No hooks registered for module 'jinja2.ext'
2020-06-08 14:52:26,165 WARNING [werkzeug] [_internal.py:113] -  * Debugger is active!
2020-06-08 14:52:26,165 - werkzeug - WARNING -  * Debugger is active!
2020-06-08 14:52:26,166 INFO [werkzeug] [_internal.py:113] -  * Debugger PIN: 206-210-676
2020-06-08 14:52:26,166 - werkzeug - INFO -  * Debugger PIN: 206-210-676
^C2020-06-08 14:52:30,880 DEBUG [ddtrace._worker] [_worker.py:58] - Stopping AgentWriter thread
2020-06-08 14:52:30,880 - ddtrace._worker - DEBUG - Stopping AgentWriter thread
```

I tried to do this outside the test env I setup and followed all the same steps to install flask, ddtrace and ddtrace[profiling]

It still came back with the same error codes described above.

I wasn't able to take this any further, so I decided that I'd try show you what code I had added to the my_flaskapp.py

### My_flaskapp.py code
```
from flask import Flask
import logging
import sys

import ddtrace
from ddtrace import Tracer as DatadogTracer

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')
```

#### Additional Documentation I used to help me in this section
1. [Datadog APM Documentation](https://docs.datadoghq.com/tracing/)  
2. [Datadog Python Tracing Setup](https://docs.datadoghq.com/tracing/setup/python/)
3. [Setup APM YouTube](https://www.youtube.com/watch?v=faoR5M-BaSw)  
4. [Additional docs](https://datadogpy.readthedocs.io/en/latest/)

#### I used the below to try solve the issues with flask and the python app

1. [Datadog Trace Client](http://pypi.datadoghq.com/trace/docs/#get-started)  
2. [Tracing Setup](https://docs.datadoghq.com/tracing/setup/python/)  
3. [Datadog Agent](https://github.com/DataDog/datadog-agent)  
4. [Tracing code](https://www.datadoghq.com/blog/monitoring-flask-apps-with-datadog/#tracing-your-code)
5. [Example flask](https://gist.githubusercontent.com/davidmlentz/4538db971af0e1d69a7936f4f8046122/raw/4a238f1c74d0f5f5ee2dd40a92b81f4176493c8c/apm_test_flask_custom.py)
6. [Pyddprofile](http://pypi.datadoghq.com/trace/docs/basic_usage.html#basic-usage)
y. [Jinga](https://jinja.palletsprojects.com/en/2.11.x/intro/)

### Bonus question: What is the difference between a Service and a Resource?

A service is a set of processes that provide a feature set. 
"A service is a set of processes that do the same job." Some examples include webapp, admin, and query. 
"A Resource is a particular action for a service." 
In a database service, these would be database queries with the span name db.query

The below link was useful for explaining and giving examples.
[Services and Resources definitions](https://docs.datadoghq.com/tracing/visualization/) 

## Creative use for Datadog

I have just finished the F1 documentary on Netflix and I think Motorsport and F1 in particular would be a great use for Datadog. The average F1 car on a race weekend can produce half a terabyte of data. With the margin of victory being measured in thousandths of a second, rapid processing and analysis of data is key to victory. 

This is a quote from Matt Harris, Head of IT for Mercedes-AMG Petronas Motorsport:

>Data's critical -- without it, we can make very few decisions," says Harris. "That data can be both structured and unstructured. Just because a driver turns around and tells us something, we can't take it for granted -- we prove it with data. We look for the anomalies in data that support configuration changes on the car.

The increased use of telemetry in F1 cars make cloud offerings more and more appealing. Having Datadog come in to the constructor team could make it easier for the F1 constructor to have all their observability metrics in one place. 

Using the three pillars of observability below:
1. Metrics
2. Traces
3. Logs

### Metrics
Since numbers are optimized for storage, processing, compression, and retrieval, metrics enable longer retention of data as well as easier querying. This makes metrics perfectly suited to building dashboards that reflect historical trends. Metrics also allow for gradual reduction of data resolution. After a certain period of time, data can be aggregated into daily or weekly frequency. Datadog could allow F1 teams to create custom metrics around their car from engine speed to gear box data, fuel usage and tyre temperature. Giving them better strategies on what to change week on week and what to change during a race.

### Traces
Traces add critical visibility into the health of an application end-to-end. Traces allow you to analyse all of your application's traces to generate in-depth latency reports to surface performance degradations, and can capture traces from all of your VMs, containers.

### Logs
Logs are the information we look at only when things are bad. A log is a text line that describes an event that happened at a certain time. Depending on the system that produces the logs, logs sometimes come in a plain text format—although the trend now is to provide structured logs so that they can be parsed easily to then run queries to debug effectively. A log consists of a timestamp and a payload that helps give more context about the event. Logs will allow teams to check what happened before an incident so they can prevent this from happening again. This could range from tyres overheating to gearbox failure. 

Datadog Log Management also comes with a set of out of the box solutions to collect your logs and send them to Datadog:

  1. Collect logs from your hosts.
  2. Collect logs from your applications.
  3. Collect logs from a Docker environment.
  4. Collect logs from a serverless environment.
  5. Collect logs from your Cloud provider.

Datadog's fluency with the ability to integrate with all the major cloud providers is a big plus. With AWS being the current leader in F1 data, AWS will be fine for the moment but as more cloud providers try get into the F1 scene. Datadog will be in a great position for multi cloud strategies. With Datadog increasing their platform offering, we could see things like Watchdog come into play when simulating different race day scenarios in the future.


## Conclusion
Thank you for taking the time to review my submission. 

I learned a lot from completing this exercise. It was fun to use tools I had never used before such as Flask and Vagrant, but by carefully reading the detailed Datadog documentation and googling a few things along the way, I was able to try and tackle every task.

I think the toughest part of this exercise was dealing with the Python problems. I couldn't get the programs to execute correctly and I looked through the agent and different troubleshooting areas but couldn't figure out where I was going wrong. I think if I was working with the Datadog team I'd reach out to one of the engineers as they'd be able to point me in the right direction and I’d go from there.



