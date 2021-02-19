# ELK For Flask


## Overview
* Prerequisites
* Cloning Project
* Objective
* Description

### Prerequisites
* Install GitCLI
* Install Python
* Install Docker
* Install Docker Compose




## Instructions
* **Objective**
	* Use a [Dockerized ELK Stack](https://github.com/platformps/docker-elk) that results in three Docker containers running in parallel:
		* container for Elasticsearch
		* container for Logstash
		* container for Kibana
	* port forwarding set up, and a data volume for persisting Elasticsearch data.
* **Description**
	* The ELK Stack (Elasticsearch, Logstash and Kibana) can be installed on a variety of different operating systems and in various different setups.
	* Linux and other Unix-based systems are the most common installation setup.
	* The intent of this project is to leverage Docker images to manifest the ELK stack.









### Shipping data into the Dockerized ELK Stack

* Our next step is to forward some data into the stack.
* By default, the stack will be running Logstash with the default [Logstash configuration file](https://github.com/platformps/docker-elk/blob/master/logstash/pipeline/logstash.conf).
* You can configure the default [Logstash configuration file](https://github.com/platformps/docker-elk/blob/master/logstash/pipeline/logstash.conf) to suit your purposes and ship any type of data into your [Dockerized ELK](https://logz.io/blog/monitoring-dockerized-elk-stack/) and then restart the container.



Alternatively, you could install Filebeat — either on your host machine or as a container and have Filebeat forward logs into the stack. I highly recommend reading up on using Filebeat on the [project’s documentation site](https://elk-docker.readthedocs.io/#forwarding-logs-filebeat).




I am going to install Metricbeat and have it ship data directly to our Dockerized Elasticsearch container (the instructions below show the process for Mac).


* First, I will download and install Metricbeat:
	* `curl -L -O 
https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.1
.2-darwin-x86\_64.tar.gz`
	* `tar xzvf metricbeat-6.1.2-darwin-x86\_64.tar.gz`

* Next, configure the `metricbeat.yml` file to collect metrics on my operating system and ship them to the Elasticsearch container:

```
cd metricbeat-6.1.2-darwin-x86\_64
sudo vim metricbeat.yml
```


The configurations:

metricbeat.modules:
- module: system
  metricsets:
    - cpu
    - filesystem
    - memory
    - network
    - process
  enabled: true
  period: 10s
  processes: \['.\*'\]
  cpu\_ticks: false

fields:
  env: dev

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: \["localhost:9200"\]

Last but not least, to start Metricbeat (again, on Mac only):

sudo chown root metricbeat.yml 
sudo chown root modules.d/system.yml 
sudo ./metricbeat -e -c metricbeat.yml -d "publish"

After a second or two, you will see a Metricbeat index created in Elasticsearch, and it’s pattern identified in Kibana.

curl -XGET 'localhost:9200/\_cat/indices?v&pretty'

health status index                       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .kibana                     XPHh2YDCSKKyz7PtmHyrMw   1   1          2            1       67kb           67kb
yellow open   metricbeat-6.1.2-2018.01.25 T\_8jrMFoRYqL3IpZk1zU4Q   1   1      15865            0      3.4mb          3.4mb

![Create Index Pattern](https://logz.io/wp-content/uploads/2018/02/Create-Index-Pattern.png)

Define the index pattern, and on the next step select the @timestamp field as your Time Filter.

![timestamp](https://logz.io/wp-content/uploads/2018/02/timestamp.png)

Creating the index pattern, you will now be able to analyze your data on the Kibana Discover page.

![bar graph](https://logz.io/wp-content/uploads/2018/02/bar-graph.png)

## Endnotes

For a sandbox environment used for development and testing, Docker is one of the easiest and most efficient ways to set up the stack. Perhaps surprisingly, ELK is being increasingly used on Docker for production environments as well, as reflected in this survey I conducted a while ago:

![twitter](https://logz.io/wp-content/uploads/2018/02/twitter.png)

Of course, a production ELK stack entails a whole set of different considerations that involve cluster setups, resource configurations, and various other architectural elements.

Do you want to compare DIY ELK vs Managed ELK?










### How to Download

#### Part 1 - Forking the Project
* To _fork_ the project, click the `Fork` button located at the top right of the project.


#### Part 2 - Navigating to _forked_ Repository
* Navigate to your github profile to find the _newly forked repository_.
* Copy the URL of the project to the clipboard.

#### Part 3 - Cloning _forked_ repository
* Clone the repository from **your account** into the `~/dev` directory.
  * if you do not have a `~/dev` directory, make one by executing the following command:
    * `mkdir ~/dev`
  * navigate to the `~/dev` directory by executing the following command:
    * `cd ~/dev`
  * clone the project by executing the following command:
    * `git clone https://github.com/MYUSERNAME/NAMEOFPROJECT`



## How to run this

* Pull Elastic’s individual images and run the containers separately or use Docker Compose to build the stack from a variety of available images on the Docker Hub.
* Execute the command below to clone the repository:
	* `git clone https://github.com/platformps/docker-elk.git`

* Execute the command below to change directories to the newly cloned local repository.
	* `cd /docker-elk`


* Execute the command below to run the stack 
	* `docker-compose up -d`

### Verifying the installation

* Execute the command below to list your containers:
	* `docker ps`

```
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                                            NAMES
a1a00714081a        dockerelk\_kibana                  "/bin/bash /usr/loca…"   54 seconds ago      Up 53 seconds       0.0.0.0:5601->5601/tcp                           dockerelk\_kibana\_1
91ca160f606f        dockerelk\_logstash                "/usr/local/bin/dock…"   54 seconds ago      Up 53 seconds       5044/tcp, 0.0.0.0:5000->5000/tcp, 9600/tcp       dockerelk\_logstash\_1
de7e3368aa0c        dockerelk\_elasticsearch           "/usr/local/bin/dock…"   55 seconds ago      Up 54 seconds       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   dockerelk\_elasticsearch\_1
```


* You’ll notice that ports on my localhost have been mapped to the default ports used by Elasticsearch (9200/9300), Kibana (5601) and Logstash (5000/5044).
* Everything is already pre-configured with a privileged username and password:
	* username: `_elastic_`
	* password: `_changeme_`

* Execute the command below to query Elasticsearch

```
curl http://localhost:9200/\_security/\_authenticate
{
  "name" : "VO32TCU",
  "cluster\_name" : "docker-cluster",
  "cluster\_uuid" : "pFgIXMErShCm1R1cd3JgTg",
  "version" : {
    "number" : "6.1.0",
    "build\_hash" : "c0c1ba0",
    "build\_date" : "2017-12-12T12:32:54.550Z",
    "build\_snapshot" : false,
    "lucene\_version" : "7.1.0",
    "minimum\_wire\_compatibility\_version" : "5.6.0",
    "minimum\_index\_compatibility\_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

* Access Kibana by navigating to http://localhost:5601 from a browser.

![Welcome to Kibana](https://logz.io/wp-content/uploads/2018/02/Welcome-to-Kibana.png)


<img src="./VIEWME.gif">






## How to Submit

#### Part 1 -  _Pushing_ local changes to remote repository
* from a _terminal_ navigate to the root directory of the _cloned_ project.
* from the root directory of the project, execute the following commands:
    * add all changes
      * `git add .`
    * commit changes to be pushed
      * `git commit -m 'I have added changes'`
    * push changes to your repository
      * `git push -u origin master`

#### Part 2 - Submitting assignment
* from the browser, navigate to the _forked_ project from **your** github account.
* click the `Pull Requests` tab.
* select `New Pull Request`
