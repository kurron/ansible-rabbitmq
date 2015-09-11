#Overview
This is an [Ansible](http://www.ansible.com/) playbook designed to greatly simplify the installation 
of [RabbitMQ](https://www.rabbitmq.com/) into an [Ubuntu 14.04](http://www.ubuntu.com/) instance.  The playbook is:

* quick -- normally taking only a few seconds to execute
* configurable -- some of the more useful RabbitMQ settings are exposed as a single configuration file
* convenient -- the default settings enable remote access via the guest account, setli the memory high water mark to 
90% and increases the `ulimit` setting to allow for larger workloads.

#Prerequisites

* an Ubuntu 14.04 Server instance with [SSH](http://www.openssh.com/) enabled and working
* the instance must have a user that has `sudo` privledges
* a box with the most current Ansible installed -- all testing was done using Ansible 1.9.3 on an Ubuntu 14.04 desktop talking 
to an Ubuntu 14.04 server
 
#Building
Since this project is just a collection of configuration and data files for Ansible to consume, no building is necessary.

#Installation
The first step is to get the files onto your Ansible box.  A great way is to use [Git](https://git-scm.com/) and
simply clone this project via `git clone https://github.com/kurron/ansible-rabbitmq.git`.  Another option is to 
[download the zip](https://github.com/kurron/ansible-rabbitmq/archive/master.zip) directly from GitHub.

Once you have the files available to you, **you must edit the `ansible.cfg` file, specifically the 
`remote_user` property.**  Failure to do this will prevent Ansible from SSH'ing into the instance.

To install RabbitMQ all you have to do is issue `./playbook.yml` from the command line.  Ansible will ask you for the password 
of the SSH account being used as well as the password to use for `sudo` (normally, you can just hit `Enter` here). Ansible 
will also prompt you for any configuration settings, providing defaults for each.  In a few seconds your instance should have the 
most current RabbitMQ installed and running.

#Tips and Tricks

##Verifying The Setup
Before trying the actual installation, it is often helpful to verify that all of your Ansible and SSH ducks are in a row.  We 
have provided a convenience script that has Ansible execute a simple `ping` command against your instance.  Try running 
`bin/ping-server.sh` to verify things are correctly set up.  If things are proper, you should see something similar to this:

```bash
bin/ping-server.sh 
SSH password: 
SUDO password[defaults to SSH password]: 
targetserver | success >> {
    "changed": false, 
    "ping": "pong"
}
```

##Verifying The RabbitMQ Installation
The simplest way to validate the RabbitMQ installation is to verify that your web browser can connect to the RabbitMQ 
administration console.  The URL is something similar to `http://192.168.1.33:15672/`.  You should be able to authenicate 
with `guest\guest` as credentials.  In the `Overview` tab, examine the value of the `File descriptors` column and ensure it 
shows the correct value.  Low values will limit the throughput of the broker.  Development workloads should use 4096 and 
production workloads should use 65536.

##Check The Tuning Guidelines
The [RabbitMQ Installation document](https://www.rabbitmq.com/install-debian.html) has notes on OS and RabbitMQ configuration 
settings. Since the advice changes between releases, it is wise to read over the page to verify that no manual provisioning 
needs to take place. 

##Installing Ubuntu As A Virtual Machine
When installing Ubuntu, make sure to hit the `F4` key and select `Minimal virtual machine`.  This will improve the performance 
of the VM as well as enable some optimizations that RabbitMQ can take advantage of.

#Troubleshooting

##An Example Run
A normal installation should look something like this:

```bash
./playbook.yml 
SSH password: 
SUDO password[defaults to SSH password]: 

PLAY [Gather prerequisites] *************************************************** 

GATHERING FACTS *************************************************************** 
ok: [targetserver]

TASK: [create groups based on distribution] *********************************** 
changed: [targetserver]

TASK: [debug msg="vm_memory_high_watermark = {{ vm_memory_high_watermark }}"] *** 
ok: [targetserver] => {
    "msg": "vm_memory_high_watermark = 0.9"
}

TASK: [debug msg="ulimit = {{ ulimit }}"] ************************************* 
ok: [targetserver] => {
    "msg": "ulimit = 4096"
}

PLAY [Install RabbitMQ] ******************************************************* 

GATHERING FACTS *************************************************************** 
ok: [targetserver]

TASK: [Install Repository keys] *********************************************** 
changed: [targetserver]

TASK: [Install RabbitMQ repository] ******************************************* 
changed: [targetserver]

TASK: [Install RabbitMQ] ****************************************************** 
changed: [targetserver]

TASK: [Copy the custom configuration file into /etc/rabbitmq] ***************** 
changed: [targetserver]

TASK: [Copy the enabled plugins configuration file] *************************** 
changed: [targetserver]

TASK: [Copy the custom configuration file into /etc/default] ****************** 
changed: [targetserver]

TASK: [Restart RabbitMQ] ****************************************************** 
changed: [targetserver]

PLAY RECAP ******************************************************************** 
targetserver               : ok=12   changed=8    unreachable=0    failed=0   
```

##Only Works On Ubuntu
When Ansible runs, it gathers up information about the target machine and will refuse to provision machine if 
it isn't an Ubuntu box.

#License and Credits
This project is licensed under the [Apache License Version 2.0, January 2004](http://www.apache.org/licenses/).

#List of Changes
