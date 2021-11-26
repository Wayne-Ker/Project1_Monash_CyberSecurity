# Project1_Monash_CyberSecurity
Project 1 of the Monash University CyberSecurity Bootcamp

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

Diagrams/Project Drawing.png

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select any playbook .yml file as they may be used to install only certain pieces of it, such as Filebeat etc.

Metricbeat-playbook.yml
Pentest-playbook.yml
Filebeat-playbook.yml

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly flexible, ensuring availability, in addition to restricting access to the network.
Load balancers protect the servers and systems from becoming overloaded and overburdened. This is especially viable when dealing with a DDos attack, as it will balance the load between the available servers to try and diminish the affects. 

Jump boxes provide the admins a secure first connection for the business, as all connections need to first run through this system. This allows the admins to control how much access users have, and decrease the ability for malicious actors to steal credentials and take over systems and networks as you cannot just scope around and find a weak system to infiltrate, but must attack the jump box which is easier to configure and strengthen against attacks then making sure all systems are up to the task of defending itself.  

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.
- The Filebeat record specifies, collects and can forward on log files and locations to Elasticsearch to be indexed so that admins can search through and examine all the logs. Filebeat is described as “a lightweight shipper for forwarding and centralizing log data” as per elastic.co.  Filebeat is made of two main components, inputs and harvesters. These two components work and act together to tail files and send the event data to whatever output you identify. The harvester reads each file line by line and sends the content of that file to the chosen output. The input is the activity in which it manages the harvesters and finds all the sources to which the harvester reads from. So if the input type for example is a LOG, the input finds all the files on the drive or system that match the defined paths and harvests each file.     

- The Metricbeat records metrics from the Operating System and services on the server, collects it and then ships them to the specified output. Metricbeat monitors and collects metrics from services ran on the server such as Apache, HAProxy, MySQL, Nginx and many more. It then sends these metrics to the Elasticsearch or Logstash depending on the configuration.   
 
The configuration details of each machine may be found below.
NameFunctionIP AddressOperation SystemJump-Box ProvisionerGateway VM AnsiblePublic: 20.70.64.17 Subnet: 10.0.0.4LinuxWeb-1DVWA Web APPPublic through LoadBalancer: 52.147.48.89 Subnet : 10.0.0.5LinuxWeb-2DVWA Web APPPublic through LoadBalancer: 52.147.48.89 Subnet: 10.0.0.6LinuxWeb-3DVWA Web APPPublic through LoadBalancer: 52.147.48.89 Subnet: 10.0.0.7LinuxElk-Stack-1Elk Stack Web AppPublic: 20.92.76.65 Subnet: 10.1.0.4Linux

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox and ElkStack machines can accept connections from the Internet. Access to these machines is only allowed from the following IP address: 124.148.198.35

Machines within the network can only be accessed by SSH. This includes the Elk-Stack-1 VM, as they all can only be SSH accessed by the jumpbox container, as that is what holds the SSH keys. 







A summary of the access policies in place can be found in the table below.

NamePublic AccessIP AddressJump-Box ProvisionerYESPublic: 20.70.64.17 Subnet: 10.0.0.4Web-1NOPublic only through LoadBalancer to provide WEB APP: 52.147.48.89 Subnet : 10.0.0.5Web-2NOPublic only through LoadBalancer to provide WEB APP: 52.147.48.89 Subnet: 10.0.0.6Web-3NOPublic only through LoadBalancer to provide WEB APP: 52.147.48.89 Subnet: 10.0.0.7Elk-Stack-1YESPublic: 20.92.76.65 Subnet: 10.1.0.4


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because for starters, ansible is a free open source program. Its main advantage though is that you can easily expand the network, machines and systems in the business and keep them up to date and in check with what they are needed for with a simple playbook that can be run over all VM’s. This can also be done quickly and easily because ansible is relatively easy to learn, and uses the python language which is easy to read. 

This is the ELK docker playbook
---
- name: Config ELK VM with Docker
  hosts: elk
  become: true
  tasks:
    - name: Install Packages
      apt:
        update_cache: yes
        name: docker.io
        state: present

    - name: Install Python Docker Module
      pip:
        name: docker
        state: present

    - name: Set VM Map Count
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes

    - name: install pip3
      apt:
        name: python3-pip
        state: present

    - name: Download and launch docker web container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        ports:
          - "5601:5601"
          - "9200:9200"
          - "5044:5044"

    - name: enable docker
      systemd:
        name: docker
        enabled: yes


The playbook for the elk stack implements the following tasks:
1. Installs the docker.io on the target machine
2. Installs the pip docker module for python to the target machine
3. Sets the max number of memory map areas a process can have, which we set to 262144 which is the maximum amount
4. Installs the python3-pip to the target machine
5. Downloads and launches the elk docker that contains the webapp, from the online image of sebp/elk:761.
6. Enables the docker to start automatically on reboot. 


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

/pictures/Docker ps

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
WEB-1 10.0.0.5 With LB Public IP address is 52.147.49.89
WEB-2 10.0.0.6 With LB Public IP address is 52.147.49.89
WEB-3 10.0.0.7 With LB Public IP address is 52.147.49.89

We have installed the following Beats on these machines:
- Filebeats and Metricbeats

These Beats allow us to collect the following information from each machine:
-In our Metricbeat, the data collected is operating system metrics such as CPU, memory or data related services running on the servers. Within Kibana we can find timestamps of all the data collected by Metricbeat, showing when system processes have occured and where within the systems. Metricbeat also monitors the Network activity, capturing the IN and OUT bytes although if we want to specifically monitor network data we should install Packetbeat.    
-In our Filebeat, the data collected is log files, data and log events. This is then sent to Kibana to be monitored and checked. Filebeat is made of two main components, inputs and harvesters. These two components work and act together to tail files and send the event data to whatever output you identify. The harvester reads each file line by line and sends the content of that file to the chosen output.   
### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the Metricbeat and Filebeat files from the elastic website to the jumpbox container. Then we must install, configure and update it.  This is done with these commands : curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb | dpkg -i metricbeat-7.6.1-amd64.deb and curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb | dpkg -i filebeat-7.6.1-amd64.deb
- Update the /etc/ansible/hosts file to include the 'groups' you want to have, such as webservers and elk groups. This will then need to be edited to show the internal IP addresses of the targeted systems, followed by 'ansible_python_interpreter=/usr/bin/python3'
This shows the ansible control what the groups are, and depending on the playbook, which systems to put it on.
- Run the playbook, and navigate to the target system through ssh, and run 'sudo docker ps' to check if the docker is installed on the system and for DVWA systems, run curl localhost/setup.php to check the connection.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? The playbook files are called Filebeat-Playbook.yml, metricbeat-playbook.yml, install-elk.yml and Pentest-Playubook.yml. These are found in the Ansible folder of this repository. You should keep all your playbooks in the one folder to maintain a neat, maintable and easily accessible system. I suggest making a Playbook folder within the Ansible folder to store these. 
- _Which file do you update to make Ansible run the playbook on a specific machine? To run the playbooks on specific machines, we must update the /etc/ansible/hosts file. In  this we must allocate machines to certain groups depending on what we want installed and where. We then must identify such group in the playbook at the start, such as in the Install-elk.yml playbook we specify the group as the "elk" group, which only has the Elk-Vm-1 in it.
- _Which URL do you navigate to in order to check that the ELK server is running? We need to navigate to http://[your.VM.IP]:5601/app/kibana. So in our case of IP addresses it was http://20.92.76.65:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

To get the files off this repository, you can either click the green code button on the right and choose download or you can enter these commands from a terminal.

git clone https://github.com/Wayne-Ker/Project1_Monash_CyberSecurity.git
This will clone the whole repository to your destination. We can then move and edit the files as needed. This can be used with the mv or cp commands, and nano to edit the text files.
As always, before you ever git clone anything, always run apt-get update && apt-get upgrade to make sure everything is up to date.

