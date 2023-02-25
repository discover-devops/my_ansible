##### IT-Infra-Tasks

##### **-   Upgrade Docker version, app upgrade*

##### **-   Repetitive tasks: Updates, Backups, Weekly system reboots**

##### **-  Creating users, Assigning Permissions and Assign Groups**  etc etc...





<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526132311667.png" alt="image-20220526132311667" style="zoom:33%;" />



##### Challenges with traditional approach:

\- Time consuming

\- Next time when you have to perform the job, again you have start from the step #1, remember last what you did, prepare the notes etc etc...

\- There are chances of Human error



**What is the solution ???**

Solution is Configuration Management tools.

**What is Config management** 
Configuration Management: As per my understanding, when we talk about the Config Management tools, we typically mean writing some kind of state description for our servers, then using a tool to enforce that the servers are, indeed, in that state.

**What does that means?**
Basically, the right packages are installed, configuration files have the expected values and have the expected permissions, the right services are running, and so on.



Why Ansible:

There are many config management tools are available Like Puppet, Chef, and Salt. Then why Ansible ??

Ansible it little different from other configuration management tools, Ansible exposes a domain-specific language (DSL) that you use to describe the state of your servers.

It is Simple, as it uses YAML which is easy to write and readable. You don't need any special coding skill.
You can execute tasks from your machine with a single command over multiple machines . 
Ansible support all OS and infrastructure.
Ansible is Powerful, you can use ansible to orchestrate a single application to entire infrastructure.
Ansible uses SSH, no agent is require, so it is fast and lite. Agentless, meaning no deployment effort and no upgrade effort.

Configuration Management: As per my understanding, when we talk about the Config Management tools, we typically mean writing some kind of state description for our servers, then using a tool to enforce that the servers are, indeed, in that state.

What does that means?
Basically, the right packages are installed, configuration files have the expected values and have the expected permissions, the right services are running, and so on. 

There are many config management tools are available Like Puppet, Chef, and Salt. Then why Ansible ??

Ansible it little different from other configuration management tools, Ansible exposes a domain-specific language (DSL) that you use to describe the state of your servers.

| Chef and  Puppet                                             | Ansible                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Chef and Puppet are configuration management systems that use  agents. | In  contrast, Ansible is push-based by default. Making a change looks like this: |
| They are pull-based by default. Agents installed on the servers  periodically check in with a central service and download configuration  information from the service. | A  sizable part of Ansible’s functionality comes from the Ansible Plugin System,  of which the Lookup and Filter plugins are most used. |
| Making configuration management changes to servers goes  something like this: | Advocates  of the pull-based approach claim that it is superior for scaling to large  numbers of servers and for dealing with new servers that can come online  anytime. |
| Step#1: make a change to a configuration management script.  | Step#1:  make a change to a playbook.                        |
| Step#2: push the change up to a configuration management central  service. | Step#2:  run the new playbook.                               |
| Step#3: Agent on server: wakes up after periodic timer fires. | Step#3:Ansible:  connects to servers and executes modules that change the state of the  servers. |
| Step#4: Agent on server: connects to configuration management  central service. |                                                              |
| Step#4: Agent on server: downloads new configuration management  scripts. |                                                              |
| Step#5: Agent on server: executes configuration management  scripts locally that change server state. |                                                              |

#### **<u>Architecture of Ansible</u>**



Ansible works with modules.
Modules are basically small programs that do the actual work.
Modules are send from the control machine to the target Servers, meaning they get pushed out there, they do there work there and deleted.
Modules, can install an application, made some configurational changes, etc etc , and when they are done they get removed.



What are modules?
Modules are basically Python packages, they are very granular, one module does one specific tasks

Module = Copy a file
Module= Install Nginx Server
Module = Stat Nginx server
Module = Create an Cloud instance

So modules are specifically designed for one tasks and Ansible has tons of modules available.

List of modules official document:

https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html



**Ansible Uses YAML**

YAML is very simple and intuitive, no need to learn any special programing skills to learn.



As mentioned that modules are very granular and very specific, designed for specific task. So in-order to make any complex configuration or application installation, we need multiple of such modules. Also, these modules are need to be run in a certain sequence. 

Say I've to install Nginx so, at high level these are going to be the steps:



<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526133931914.png" alt="image-20220526133931914" style="zoom: 33%;" />

<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526134247230.png" alt="image-20220526134247230" style="zoom:33%;" />



So, the sequential modules are grouped in Tasks.
Tasks are describe with some and it make sure that modules are executed in certain arguments.
You can also execute multiple modules in sequence.
In the playbook, we have to describe the target machine using HOSTS and specified the user who is going to execute the tasks.
REMOTE_USER.

ANSIBLE Inventory File







====================



**Ansible Modules**



Ansible Modules
-----------------

Here we are going to discuss about the Ansible Modules. 
Ansible modules are categorized into various groups based on their functionality some of them are listed here:

**System modules** are actions to be performed at a system level such as modifying the users and groups on the system, 
modifying iptables,firewall configurations on the system, working with logical volume groups,mounting operations, or working with services.
For example, starting, stopping, or restarting services in a system.

**Command modules** are used to execute commands or scripts on a host. These could be simple commands using the command module or an interactive execution using the expect module by responding to prompts.

You could also run a script on the host using the **script module**.

**File modules** help work with files. For example, use the ACL module to set and retrieve ACL information on files. Use the archive and unarchive modules to compress and unpack files. Use find, line in file,and replace modules to modify the contents of an existing file.

**Database modules** help in working with databases such as MongoDB, MySQL, MSSQL,or PostgreSQL to add or remove databases or modify database configurations.

The **cloud section** has a vast collection of modules for various different cloud providers like Amazon, Azure, Docker, Google,
OpenStack, and VMware being just a few of them.

There are a number of modules available for each of these that allow you to perform various tasks such as creating and destroying instances,
performing configuration changes in networking & security,managing containers, datacenters, clusters, virtual networking, VSAN,
and a lot more.

Then there are **Windows modules** that help use Ansible in a Windows environment. Some ofthese are **win_copy** to copy files,
**win_command** to execute a command on a Windows machine and there are a bunch of other modules available to work with files on Windows,
create a IIS website, install a software using MSI installer,make changes to registry using regedit and manage services and users in Windows.

These are just a few modules in a few categories.

There are a lot more and a comprehensive list can be found at docs.ansible.com along with detailed instructions on each of them.https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html

Let’s look at few of these modules in detail to understand, how you can use them and how to read the documentation page.
We will start with the command module.

The command module is used to execute a command on a remote node.The parameters of the command module as listed
in the documentation page is shown here:
https://docs.ansible.com/ansible/2.9/modules/command_module.html#command-module

<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526194114987.png" alt="image-20220526194114987" style="zoom: 50%;" />



Another module to look at is the **script module**. The script module executes a script which is located locally on the Ansible
controller machine on one or more remote nodes after transferring it over. 

To run a script on one or hundreds of servers, you really don’t have to copy it over to all the servers, Ansible takes care of automatically copying the script to the remote node and then executing it on the remote systems.

<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526194511570.png" alt="image-20220526194511570" style="zoom:50%;" />



<img src="C:\Users\rahchaub\AppData\Roaming\Typora\typora-user-images\image-20220526194757656.png" alt="image-20220526194757656" style="zoom:50%;" />

