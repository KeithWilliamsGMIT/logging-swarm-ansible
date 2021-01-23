# Logging Swarm Ansible

This repository shows how to setup a logging stack running in Docker Swarm using Ansible to automate the task. This stack consists of the following well known technologies from Elastic:

+ [Elasticsearch](https://www.elastic.co/products/elasticsearch)
+ [Filebeat](https://www.elastic.co/products/beats/filebeat)
+ [Kibana](https://www.elastic.co/products/kibana)

The services in this stack are also configured to work with the Traefik reverse proxy. The [proxy-swarm-ansible](https://github.com/KeithWilliamsGMIT/proxy-swarm-ansible) repository can be used to setup Traefik.

## Getting Started

The first step to getting started is to clone the repository:

```bash
git clone https://github.com/KeithWilliamsGMIT/logging-swarm-ansible.git
```

Next, install Ansible and then we can use two existing Ansible roles to install Docker and initialise a Docker Swarm. These roles are defined in the `requirements.yml` file and will be installed to `~/.ansible/roles/` by default using the below commands:

```bash
cd playbooks/requirements
ansible-galaxy install -r requirements.yml
```

We can install [Docker](https://docs.docker.com/install/) on all the hosts now using the `install-docker.yml` playbook as shown below:

```bash
cd playbooks
ansible-playbook -i ./inventories/localhost install-docker.yml
```

Similarly, to initialise a Docker Swarm between the hosts we can run the `initialize-swarm.yml` playbook. Note that we set extra variables to override some defaults in the playbook. We want to skip everything except the actual initialisation of Docker Swarm.

```bash
ansible-playbook -i ./inventories/localhost initialize-swarm.yml --extra-vars="{'skip_engine': 'True', 'skip_group': 'True', 'skip_docker_py': 'True'}"
```

Finally, we can deploy the logging stack to Docker Swarm using the `deploy-logging.yml` playbook.

```bash
ansible-playbook -i ./inventories/localhost deploy-logging.yml --ask-vault-pass --ask-become-pass
```

The `--ask-vault-pass` flag will prompt you for the vault password to access sensitive data. A default password used in this case, for demostraion purposes, is `password`.

## Does it work?

Once you followed the above steps we can check if the logging services are running as expected on the docker manager.

```bash
ubuntu:~/logging-swarm-ansible/playbooks$ sudo docker service ls
ID                  NAME                    MODE                REPLICAS            IMAGE                                                 PORTS
ytuxymspy2o3        logging_elasticsearch   replicated          1/1                 docker.elastic.co/elasticsearch/elasticsearch:6.5.0   *:9200->9200/tcp, *:9300->9300/tcp
i66adbfsgi3y        logging_filebeat        global              1/1                 docker.elastic.co/beats/filebeat:6.5.0
vioplzrgpf2e        logging_kibana          replicated          1/1                 docker.elastic.co/kibana/kibana:6.5.0                 *:5601->5601/tcp
```

We can now check these services by navigating to `http://localhost:port/` in a browser. For example, to navigate to Kibana go to [http://localhost:5601/](http://localhost:5601/). You can login to Kibana using the default username `elastic` and password `password` set in this repository.

## Further configuration

There are also a number of Ansible variables that can be overridden. These can be found in the table below:

| Variable Name | Description | Default Value |
|---------------|-------------|---------------|
| temporary_directory | A path to a directory where temporary files such as the compose file can be written to | "/tmp" |
| volume_directory | A path to a directory where service data can be persisted | "/tmp" |
| docker_network_name | The name of the new Docker overlay network | "logging_network" |
| elastic_version | The tag of the various Elastic Docker images to use | 6.5.0 |
| elasticsearch_username | The username of the user to log into Elasticsearch | admin |
| domain_name | The domain name used to access the services | example.local |

## Using this role

To use this role add the following to your `requirements.yml` file:

```
- src: https://github.com/KeithWilliamsGMIT/logging-swarm-ansible.git
  version: master
  name: deploy-logging
```

## Contributing

Any contribution to this repository is appreciated, whether it is a pull request, bug report, feature request, feedback or even starring the repository. Some potential areas that need further refinement are:

+ Hardening of playbooks
+ Publish to Ansible Galaxy
+ Documentation

## Conclusion

This repository demonstates how to use the Elastic stack to provide logging capabilities in Docker Swarm. This stack passes all logs from all services on all Docker nodes from Filebeat to Elasticsearch where they are stored. These can then be queried and visualised in Kibana. The Elastic stack is highly flexible and configurable and while this repository offers a basic implementation there is room for further enhancements, for example, the addition of other beats and possible Logstash to standardise the log formats. Finally, running all of these commands manually everytime we need to deploy this logging stack would be time consuming and so Ansible is quite useful in automating this process.
