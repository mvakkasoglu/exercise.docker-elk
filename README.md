# Docker ELK


## Overview

* **Prerequisites**
	* Install GitCLI
	* Install Docker
	* Install Docker Compose


* **Objective**
	* Use a [Dockerized ELK Stack](https://elk-docker.readthedocs.io/) that results in three Docker containers running in parallel:
		* container for Elasticsearch
		* container for Logstash
		* container for Kibana
	* Ensure the stack has port forwarding set up, and a data volume for persisting Elasticsearch data.
* **Description**
	* The ELK Stack (Elasticsearch, Logstash and Kibana) can be installed on a variety of different operating systems and in various different setups.
	* Linux and other Unix-based systems are the most common installation setup.
	* The intent of this project is to leverage Docker images to manifest the ELK stack.



## Instructions

### Shipping data into the Dockerized ELK Stack
* After verifying that the stack is running properly, forward some data into the stack.
* By default, the stack will be running Logstash with the default [Logstash configuration file](./docker-elk/logstash/pipeline/logstash.conf).
* You can do either of the following:
  * **Option 1**
    * configure the default [Logstash configuration file](./docker-elk/logstash/pipeline/logstash.conf) to suit your purposes.
    * ship any type of data into your [Dockerized ELK](https://logz.io/blog/monitoring-dockerized-elk-stack/)
    * restart the container.
  * **Option 2**
    * install Filebeat either on your host machine or as a container.
    * Use Filebeat forward logs into the stack.
    * Ensure you familiarize with Filebeat using the [project’s documentation site](https://elk-docker.readthedocs.io/#forwarding-logs-filebeat).



### Using Metricbeat

* To install Metricbeat and have it ship data directly to our Dockerized Elasticsearch container (the instructions below show the process for Mac).


* Execute the command below to download and install Metricbeat:
	* `curl -L -O 
https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.1.2-darwin-x86_64.tar.gz`
	* `tar xzvf metricbeat-6.1.2-darwin-x86_64.tar.gz`

* Configure the `metricbeat.yml` file to collect metrics on my operating system and ship them to the Elasticsearch container:

```
cd metricbeat-6.1.2-darwin-x86_64
sudo vim metricbeat.yml
```


* The configurations:

```
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
  processes: ['.*']
  cpu_ticks: false

fields:
  env: dev

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["localhost:9200"]
```

* Last but not least, to start Metricbeat (again, on Mac only):


```
sudo chown root metricbeat.yml 
sudo chown root modules.d/system.yml 
sudo ./metricbeat -e -c metricbeat.yml -d "publish"
```


* After a second or two, you will see a Metricbeat index created in Elasticsearch, and it’s pattern identified in Kibana.
  * `curl -XGET 'localhost:9200/_cat/indices?v&pretty'`


```
health status index                       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .kibana                     XPHh2YDCSKKyz7PtmHyrMw   1   1          2            1       67kb           67kb
yellow open   metricbeat-6.1.2-2018.01.25 T_8jrMFoRYqL3IpZk1zU4Q   1   1      15865            0      3.4mb          3.4mb
```


![Create Index Pattern](https://logz.io/wp-content/uploads/2018/02/Create-Index-Pattern.png)

* Define the index pattern, and on the next step select the @timestamp field as your Time Filter.

![timestamp](https://logz.io/wp-content/uploads/2018/02/timestamp.png)

* Creating the index pattern, you will now be able to analyze your data on the Kibana Discover page.

![bar graph](https://logz.io/wp-content/uploads/2018/02/bar-graph.png)

* Click [here](https://elk-docker.readthedocs.io/) to view more detailed instructions.
* Click [here](./docker-elk/README.md) to view the original `README.md`








## How to Download

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
