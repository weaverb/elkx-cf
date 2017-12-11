# Configuring the ELK (with x-pack) stack using Docker Compose with sebp/elkx image

## Helpful information
* [sebp/elkx docker image](https://hub.docker.com/r/sebp/elkx/)
* [sebp/elk documentation](https://elk-docker.readthedocs.io/)

## Pre-reqs

* Docker installed
  * Docker must be confgure with atleast 4GB of RAM
  * A limit on mmap counts equal to 262,144 or more
    * [setting vm-max-map-count](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/vm-max-map-count.html#vm-max-map-count) 

## Setup

0. Create the docker compose file

    ```docker
    elkx:
    image: sebp/elkx
    ports:
        - "5601:5601"
        - "9200:9200"
        - "5044:5044"
    environment:
        - ELASTIC_BOOTSTRAP_PASSWORD="changeme"
    ```

0. Boot container with `docker-compose up`
0. After the container has started, open a second terminal and use docker exec to enter the container `docker exec -it <name of the running container> /bin/bash`
0. Run the xpac password config: `$ES_HOME/bin/x-pack/setup-passwords interactive`
0. After the setup-password is complete stop the container and then modifiy the docker-compose.yml file to have the username/passwords you created in the previous steps
    ```docker
    elkx:
    image: sebp/elkx
    ports:
        - "5601:5601"
        - "9200:9200"
        - "5044:5044"
    environment:
        - ELASTICSEARCH_USER=elastic
        - ELASTICSEARCH_PASSWORD=changeme
        - LOGSTASH_USER=elastic
        - LOGSTASH_PASSWORD=changeme
        - KIBANA_USER=kibana
        - KIBANA_PASSWORD=changeme
    ```
0. restart the container with `docker-compose up`
0. Again open a second terminal and docker exec into the running container.
0. modify `/opt/logstash/config/logstash.yml` to have the following lines:
    ```bash
    xpack.monitoring.elasticsearch.username: "logstash_system"
    xpack.monitoring.elasticsearch.password: "changeme"
    ```
0. Create a dummy log to jumpstart kibana.  First run this command in the second terminal: `/opt/logstash/bin/logstash --path.data /tmp/logstash/data -e 'input { stdin { } } output { elasticsearch { hosts => ["localhost"] user => "elastic" password => "changeme" }}'`
0. when the agent prompts enter a message (e.g. "this is a dummy message") and press enter to insert it.  
0. Ctrl+C to exit the logstash agent terminal
0. Open a browser and navigate to http://localhost:5601 and log into kibana with the user "elastic" and the password you have set.
0. Create and index pattern and off you go!