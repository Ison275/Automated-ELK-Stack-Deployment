## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below:

(Automated ELK Stack Deployment/Images/Network_Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. 

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
- Beats in Use
- Using the Playbooks


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. The load balancer ensures that work to process inccoming traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorized users will be able to connect in the first place.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as: CPU usage; attempted SSH logins; sudo escalation failures; etc.

The configuration details of each machine may be found below.

| Name     |  Function     | IP Address | Operating System |
|----------|---------------|------------|------------------|
| Jump Box | Gateway       | 10.0.0.4   | Linux            |
| DVWA 1   | Web Server    | 10.0.0.5   | Linux            |
| DVWA 2   | Web Server    | 10.0.0.6   | Linux            |
| DVWA 3   | Backup Server | 10.0.0.7   | Linux            |
| ELK      | Monitoring    | 10.1.0.4   | Linux            |

Redundancy for the DVWA servers is ensured by using multiple servers organized into the following availability zones:

- Availability Zone 1: DVWA 1 + DVWA 2
- Availability Zone 2: DVWA 3
- Availability Zone 3: ELK


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 45.132.115.70.

Machines within the network can only be accessed by each other. The DVWA 1, DVWA 2, and DVWA 3 VMs send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 45.132.115.70        |
| DVWA 1   | No                  | 10.0.0.0/16          |
| DVWA 2   | No                  | 10.0.0.0/16          |
| DVWA 3   | No                  | 10.0.0.0/16          |
| ELK      | No                  | 10.1.0.0/16          |


### Elk Configuration

The ELK VM uses Docker to download and manage an ELK container that exposes an Elastic Stack instance. Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. The playbook implements the following tasks:

- Installs docker.io: the Docker engine, used for running containers
- Installs python3-pip: Package used to install Python software
- Installs docker module: Python client for Docker
- Increases the virtual memory of the target VM in order to run the ELK container
- Downloads and launches the Docker container called sebp/elk:761
- Configures the container to start with the following port mappings
  - 5601:5601
  - 9200:9200
  - 5044:5044

(Resources/install_elk.yml) 


### Target Machines & Beats

This ELK server is configured to monitor the following machines:

- DVWA 1      10.0.0.5
- DVWA 2      10.0.0.6
- DVWA 3      10.0.0.7

The following Beats are installed on these machines:

- Filebeat
- Metricbeat
- Packetbeat

These Beats allow us to collect the following information from each machine:

- Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
- Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed sudo escalations, and CPU/RAM statistics.
- Packetbeat collects packets that pass through the NIC, similar to Wireshark. We use it to generate a trace of all activity that takes place on the network, in case later forensic analysis should be warranted.

The playbooks below install Filebeat and Metricbeat on the target hosts. 

(Resources/filebeat_playbook.yml)
(Resources/metricbeat_playbook.yml)


### Using the Playbooks

In order to use the playbooks, you will need to have an Ansible control node already configured. We use the Jump Box for this purpose.

First we must SSH into the Jump Box and locate, start, and attach to our Ansible container.

$ ssh RedAdmin@23.96.55.16

$ sudo docker ps

$ sudo docker list -a

$ sudo docker start goofy_allen

$ sudo docker attach goofy_allen


The easiest way to copy the playbooks is to use Git:


$ cd /etc/ansible

$ mkdir files

# Clone Repository + IaC Files

$ git clone https://github.com/Ison275/Automated-ELK-Stack-Deployment.git

# Move Playbooks and hosts file Into `/etc/ansible`

$ cp Automated-ELK-Stack-Deployment/resources/* .

$ cp Automated-ELK-Stack-Deployment/resources/* ./files


This copies the playbook files to the correct place.

Next, you must create a hosts file to specify which VMs to run each playbook on. Run the commands below:

$ cd /etc/ansible

$ cat > hosts <<EOF

[webservers]

10.0.0.5

10.0.0.6

10.0.0.7


[elk]

10.1.0.4

EOF


After this, run the commands below to run the playbooks:


$ cd /etc/ansible/files

$ ansible-playbook install_elk.yml elk

$ ansible-playbook install_filebeat.yml webservers

$ ansible-playbook install_metricbeat.yml webservers


The following screenshots display the results of successfully running the playbooks:

(Images/ELK_Screen_Shot)
(Images/Beats_Screen_Shot)

To verify success, wait five minutes to give ELK time to start up. 

Verify that you can access your server by opening a web browser and navigating to http://104.45.218.23:5601. (dynamic public ip of ELK server will vary) You should see the following Kibana homepage: 

(Images/Kibana_Home.png)

Kibana is a free and open user interface that lets us visualize our Elasticsearch data and navigate the Elastic Stack. From here we can easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as: CPU usage; attempted SSH logins; sudo escalation failures; etc.

Enjoy!
