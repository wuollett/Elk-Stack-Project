## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Network Diagram](Diagrams/Bigger-Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.

  - [Elk playbook](Ansible/elkplaybook.yml)

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network. Load balancers ensure availability of the web server that is being accessed, if one machine hosting the web address should go down, the other one will still be available. The load balancer serves as the public facing web address for access to the D*mn Vulnerable Web App, conversely, the Jump-Box-Provisioner machine serves as the only means of access directly to the Web-1 and Web-2 virtual machines which host the web server.  

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the usage and system logs.
- Filebeat monitors for changes to the system logs, collects them and sends them to the ELK server.
- Metricbeat collects the metrics and statistics of the system and services.

The configuration details of each machine may be found below.


| Name                 | Function        | Private IP | Public IP     | Operating System | Docker Containers  | Container Function                        |
|----------------------|-----------------|------------|---------------|------------------|--------------------|-------------------------------------------|
| Jump-Box-Provisioner | Gateway         | 10.0.0.4   | 20.127.29.209 | Linux            | boring_nightingale | Ansible, SSH Gateway                      |
| Web-1                | Web Server      | 10.0.0.5   | N/A           | Linux            | dvwa               | Host D*mn Vulnerable Web App              |
| Web-2                | Web Server      | 10.0.0.6   | N/A           | Linux            | dvwa               | Host D*mn Vulnerable Web App              |
| ELK-Stack-Server     | Data Aggregator | 10.1.0.4   | 13.64.150.214 | Linux            | elk                | Host Elastic Search, Logstash, and Kibana |

A visual of the virtual machines in the Azure Portal.
![VMS](/Screenshots/Cloud VMs.PNG)

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Each of the virtual machines can accept connections from the Internet but only under specific circumstances. Access to these machines is only allowed from the following IP addresses:
```
        141.126.167.203
        76.101.174.173
```
![RedTeamNSG](/Screenshots/Red-Team-NSG.PNG)
![ELKNSG](/Screenshots/ELK-NSG-rules.PNG)

Machines within the network can only be accessed directly via SSH by the Jump-Box-Provisioner virtual machine.

A summary of the access policies in place can be found in the table below.


| Name                 | Pubicly Accessible | Allowed Source IP Addresses and Ports                         | Allowed Destinations |
|----------------------|--------------------|---------------------------------------------------------------|----------------------|
| Jump-Box-Provisioner | Yes                | 141.126.167.203:22, 76.101.174.173:22                         | Virtual Network      |
| Web-1                | No                 | 10.0.0.4:22, 141.126.167.203:80, 76.101.174.173:80            | Virtual Network      |
| Web-2                | No                 | 10.0.0.4:22, 141.126.167.203:80, 76.101.174.173:80            | Virtual Network      |
| Elk-Stack-Server     | Yes                | 10.0.0.4:22, 141.126.167.203:22,5601 76.101.174.173:22,5601,  | Virtual Network      |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because manual configurations can often run into issues where much time is spent configuring software to be used on a particular machine. Ansible avoids this issue by following a consistent, reliable, and simple "playbook", as it is referred to in the application, to setup configurations the same way everytime. In conjunction with Docker, a Platform as a Service (PaaS) software that uses OS-level virtualization in the form of "containers" to deliver software, Ansible playbooks were leveraged in this project to ensure consisent installation of the ELK machine, the D*mn Vulnerable Web Application machines, and the Metricbeat and Filebeat data collection software. 

The [playbook](Ansible/elkplaybook.yml) implements the following tasks:
- Installs Docker service
- Installs python 
- Installs Docker Module
- Increases the VMs vitual memory
- Configures the BM to use it's increased memory
- Downloads an image of an ELK container and launches it
- Ensures that docker is started on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![Elk Docker](/Screenshots/ELK-Container.PNG)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
```
10.0.0.5
10.0.0.6
```

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat collects system logs and services running on the systems on which it is installed. It can also track sudo commands, SSH logins and new users and groups on a system.
- Metricbeat collects statistics on the usage on the system, such as network, memrory, and cpu.

An Example of a combined beats dashboard:
![MetricFileBeatBoard](/Screenshots/Beats Dashboard.PNG)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Navigate to the /etc/ansible directory with:
```
cd /etc/ansible
```
- Create the roles directory, this will contain your metricbeat and filebeat playbooks.
```
mkdir roles
```
- Update the [hosts](/Linux/hosts) file to include the elk and webserver address. It should look similar to this:
```
[webservers]
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
```
- Copy the [Filebeat-Config](/Linux/filebeat-config.yml) and [Metricbeat-Config](/Linux/metricbeat-config.yml) to the /etc/ansible directory.
- Copy the [Filebeat-Playbook](/Ansible/filebeat-playbook.yml) and [Metricbeat-Playbook](/Ansible/metricbeat-playbook.yml) into the /etc/ansible/roles directory. 
- From with in the /etc/ansible/roles directory run each playbook with 
```
ansible-playbook filebeat-playbook.yml
ansible-playbook metricbeat-playbook.yml
```
- Navigate to http://[your.VM.IP]:5601/app/kibana. Use the public IP address of the ELK server that you created to check that the installation worked as expected.

### Troubleshooting and Lessons Learned
This virtual network is setup to allow access from your home computer on your home network. One disadvantage of this is that SSH keys generated outside of Azure will not travel from one machine to the next and the GUI interface of the Azure portal does not seem to allow for including multiple public keys on one VM. Thus, in this particular project, the SSH keys had to be updated on multiple occasions due to switching from one machine to another while traveling. 
This update is shown here:
![ssh](/Screenshots/SSH-Reset.PNG)
![ssh2](/Screenshots/SSH-Reset2.PNG)
Additonally, network security rules had to be updated to include multiple entries for which to SSH into the Jumpbox gateway, and to view the Elk websever. This may also occur if your ISP has your network IP set to be dynamic, in which case the Network Security Rules for which IP is able to connect must be updated accordingly.  

One major advantage of the Ansible playbook system is that it is highly versatile, and mistakes in writing the playbook can be easily corrected and re-ran for the new changes to take effect. An example of this is shown here:
![Filebeat](/Screenshots/Filebeat-setup error-on-Elk-Server.PNG)
![Filebeat error 2](/Screenshots/Filebeat-removal-part-1.PNG)

In this instance, Filebeat was mistakenly placed on the ELK server rather than the Web-1 and Web-2 webservers. The only necessary step to rectify this mistake was updating the filebeat-playbook.yml file changing the hosts section of the playbook to be webservers rather than elk seen here:
![hosts1](/Screenshots/filebeat-playbook-hosts.PNG)