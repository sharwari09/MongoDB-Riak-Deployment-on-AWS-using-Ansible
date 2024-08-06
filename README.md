# Deployment of MongoDB and Riak Cluster using Ansible


## Ansible Overview

Ansible is an open-source software provisioning, configuration management, and application deployment tool. It is a DevOps tool that can be used to deploy apps and automate IT infrastructure.</br>
[Ansible Tool](https://www.ansible.com/)

- Ansible has been used to deploy EC2 instances for MongoDB and Riak DBs on AWS and configure them for testing network partition. 

- Ansible uses Playbooks to get the tasks done. The following are some components that have been used in this project:
  - **Playbook**:</br>
    It is an organized unit of scripts that defines work for a server configuration managed by Ansible.
  - **Tags**:</br>
    Tags in a playbook enable the user to deploy tasks as per the tags provided. This makes it easy to run only the desired tasks out of all the many tasks in a playbook.
  - **Inventory File**:</br>
    Ansible works on SSH protocol, hence to run tasks on hosts we need to create an inventory file that has entries of all the hosts to where we want to SSH and run tasks. 
  - **Vars File**:</br>
    The Vars file consists of all the vars that we may want to use in a playbook to run. It is similar to a config file.
    
## Commands to run playbooks to deploy and configure Mongo Cluster

```shell
    $ cd ~/ansible-playbooks
    $ vim system_config.yml
```
- system_config.yml is the Vars file that has been used in the deployment playbook.  The system_config.yml file looks like below.
   
```yaml
    ---

    db_type: mongo
    key_name: <key_name>
    instance_type: <instance_type>
    image_id: <image_id>
    security_group: <security_group>
    vpc_subnet_id_1: <vpc_subnet_id_1>
    vpc_subnet_id_2: <vpc_subnet_id_2>
    region: <regio>
    aws_access_key: ''
    aws_secret_key: ''
```
   
- For deploying a Mongo cluster, provide _mongo_ as the _db_type_ and the Mongo AMI ID.

- Create a Security Group with 22, 27017 ports open. And provide the newly created SG's name as a value to the key security_group in the system_config.yml.
   
   - Provide your AWS access and security keys in the system_config.yml file.
   
   - Create an inventory file with the following contents:
   
```
    [localhost]
     localhost ansible_python_interpreter=/usr/local/bin/python3  ansible_connection=local
```
  - Create a file named mongo-inventory

- The following command deploys 5 Mongo EC2 instances on AWS. With tags provided as mongo.
   
```shell
    ansible-playbooks$ ansible-playbook deploy_db_cluster.yml -i inventory --tags "mongo" 
```
    
- The above command will deploy all the Mongo nodes in AWS.
    
## Commands to run playbooks to deploy and configure Riak Cluster

### Deploy complete RIAK Cluster on AWS

```shell
    $ cd ~/ansible-playbooks
    $ vim system_config.yml
```
   - system_config.yml is the Vars file that has been used in the deployment playbook.  The system_config.yml file looks like below.
   
```yaml
    ---

    db_type: riak
    key_name: <key_name>
    instance_type: <instance_type>
    image_id: <image_id>
    security_group: riak-cluster
    vpc_subnet_id_1: <vpc_subnet_id_1>
    vpc_subnet_id_2: <vpc_subnet_id_2>
    region: <region>
    aws_access_key: ''
    aws_secret_key: ''
```
   
   - For deploying a Riak cluster, provide _riak_ as the _db_type_ and the riak AMI ID.
   
   - Create a Security Group with 22, 80, 443, and 0-65535 ports open. And provide the newly created SG's name as a value to the key security_group in the system_config.yml.
   
   - Provide your AWS access and security keys in the system_config.yml file.
   
   - Create an inventory file with the following contents:
   
```
    [localhost]
     localhost ansible_python_interpreter=/usr/local/bin/python3  ansible_connection=local
```

   - Create a file named riak-inventory
  
   - The following command deploys 5 Riak EC2 instances on AWS. With tags provided as riak.
   
```shell
    ansible-playbooks$ ansible-playbook deploy_db_cluster.yml -i inventory --tags "riak" 
```

   - Once all the nodes have been deployed, we will start the Riak service on each Riak node. To do that run the following command:

```shell
    ansible-playbooks$ ansible-playbook setup_riak_node.yml -i riak-inventory  
```

   - The above command will start Riak service on each node, ping and check the admin status
   
   - It is time to for every node to join the Riak cluster, for this execute the following playbook:
   
```shell
    ansible-playbooks$ ansible-playbook join_riak_cluster.yml -i riak-inventory --extra-vars "host=riak-secondary1 primary_host={private_ip_address_of_the_primary_node}
```
   _host_ in the above command is the hostname of the secondary node which has to be joined to the cluster, and _primary_host_ is the IP or hostname of the primary RIAK node. 
   Execute the above command for each secondary node by replacing host var's value with _secondary-1_, _secondary-2_, _secondary-3_ and _secondary-4_
   
   - Once all the nodes have joined the Riak cluster, commit the cluster plan as follows:
 
```shell
    ansible-playbooks$ ansible-playbook commit_plan.yml -i riak-inventory --extra-vars host=riak-primary
```

  - Basic configuration is now done on the Riak cluster. 
   
