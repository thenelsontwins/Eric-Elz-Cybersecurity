# Eric-Elz-Cybersecurity
Cybersecurity 

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)



________________________________________________________

These files have been tested and used to generate a live ELK deployment on Azure. 
They can be used to recreate the entire deployment pictured above. 


- _TODO: Enter the playbook file._


---
- name: Configure Elk VM w docker
  hosts: elk
  remote_user: redteamadmin
  become: True
  tasks:

    - name: Install docker.io
      apt:
        force_apt_get: yes
        update_cache: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Docker python module
      pip:
        name: docker
        state: present

    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

    - name: download docker container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always

        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: enable docker service
      systemd:
        name: docker
        enabled: yes

________________________________________________________


Alternatively, select portions of the __filebeat-playbook.yml__ file may be used to install only certain pieces of it, such as Filebeat.

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

________________________________________________________

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build
________________________________________________________
### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly _accessible and availaible_, in addition to restricting _unwanted traffic_ to the network.

###
Load Balancers function by distributing web traffic across multiple servers to ensure an efficient network load across servers 
aiding in network redundancy and fault tolerance providing protection in the event of an attack or increased web traffic.  In this way, 
in the event that a server is down, comprimised, or when the service is scaled up or down the load balancer can respond in turn to 
ensure data is accessible.

###
In this vitrual network a Jump Box was employed.  The jump box has a number of advantages, one being that it acts as a secure single point 
of entry protecting our network assets from public internet traffic.  Through the jump box we are able to control access to the other 
machines on the network by defining which specific IP addresses have access to the network.  Beyond that, our jump box limits access to 
the administrator account, includes SSH security, and in this instance, allows us configure additional machines utlizing a Docker container
and the provisioner Ansible as well as deploying an ELK stack for log collection on another server.

###
Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the __logs___ and system _health_.
The ELK Beats Filebeat and Metricbeat were deployed.  
Filebeat is used to watch for changes in the file system log files and locations.
Metricbeat is used to collect information about the operating system and services running on servers.  

###
The configuration details of each machine may be found below.

| NAME     | FUNCTION | IP ADDRESS | OPERATING SYSTEM |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.1.0.4   | Linux            |
| WEB-1    | VM       | 10.1.0.5   | Linux            |
| WEB-2    | VM       | 10.1.0.6   | Linux            |
| WEB-3    | VM       | 10.1.0.7   | Linux            |
| Elk_VM   | VM       | 10.0.0.4   | Linux            |


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. 

Access to this machine is only allowed from the following IP address:
74.73.11.69

Machines within the network can only be accessed by _Ansible on the Jump Box_.

- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_

The Jump Box has access to the ELK VM via Ansible.  
10.1.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | No		         | 10.0.0.4             |
|Device IP | No                  | 74.73.11.69          |
|          |                     |                      |



### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous utilizing 
Infrastructure as Code (IaC) and the Ansible provisioner, we are able to quickly and simply set up the ELK server with a simple,
lightweight, text file which we can quickly and easily use to define and create our ELK server without administrator intervention.
Also, with the use of Ansible and IaC we're afforded the ability to share and audit code, make changes, and redeploy new servers in the 
event of a comprimise or if updates are needed.  

The playbook implements the following tasks:

In order to install ELK the following steps were taken:
- Within the JumpBox VM, Docker was downloaded, started, and then a new image of the Docker service containing Ansible was pulled from Docker Hub.
- Following the installation, a new container was launched, then started, and attached. 
- Within the Ansible container, a new SSH public key was generated and added to the the associated VMs in the Azure.
- Once connection was confirmed, the ansible "hosts" config file was updated, as well as the aisible "config" file to contain the associated IP addresses
of the VMs in use.  
- A new VM was created for the ELK server, the Ansible hosts file was updated to include the new VM, and an Ansible playbook was built to install Docker 
and configure the ELK VM automatically.  

###

??????????????????????????????????????????????????????????????????????????????????????????????
??????????????????????????????????????????????????????????????????????????????????????????????

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

??????????????????????????????????????????????????????????????????????????????????????????????
??????????????????????????????????????????????????????????????????????????????????????????????


### Target Machines & Beats

This ELK server is configured to monitor the following machines:

WEB-1 | 10.1.0.5
WEB-2 | 10.1.0.6
WEB-3 | 10.1.0.7

We have installed the following Beats on these machines:
Filebeat
Metricbeat

These Beats allow us to collect the following information from each machine:
Filebeat in this instance will be collecting system logs, including syslog data, sudo commands, SSH logins, and the creation of new users and groups.
Metricbeat will collect information related to Docker containers, images, memory usage, etc. 


### Using the Playbook

In order to use the playbook, you will need to have an Ansible control node already configured. 
Assuming you have such a control node provisioned: 
SSH into the control node and follow the steps below:
  
Copy the install-elk.yml file to /etc/ansible/roles.

###
Update the Ansible <hosts> file to run the Ansible playbook on a specific machines. 
In the Ansible <hosts> file we are able to seperate our virtual machines into into groups, populating the groups with the IP addresses of our VMs.  
For example, we had a group called [webservers] which contained all the IP addresses of the VMs we wanted to install Ansible on.  
We also cretaed an [elk] group in order to configure the ELK server at its specific IP address.   

###
In order to specify which machine to install the ELK server on within the the ELK installer yaml file, we directed the file to look to the [elk] group inside 
the Ansible host file which contains the IP address of the ELK machine.

For Filebeat installation we specified within the <filebeat-config.yml> the [host] configuration for Elasticsearch Output and Kibana configuration [host] 
directing both services toward the IP address of the ELK machine.

###
In order to check that ELK is running, we directed a web browser to 
http://[my elk vm public ip]:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

    commands used: 
	curl 
	sudo apt update
	sudo apt install
	
