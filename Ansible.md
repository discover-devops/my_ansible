## Ansible Overview

![image](https://github.com/discover-devops/my_ansible/assets/53135263/fb157626-26fb-4cfb-bea6-c275a80b832a)


**What is Ansible?**
Ansible is an open-source automation tool that simplifies configuration management, application deployment, and task automation. It provides a simple, human-readable language (YAML) to describe automation tasks, making it easy to understand and write automation scripts. Ansible is agentless, meaning it doesn't require any software to be installed on managed nodes, making it lightweight and efficient.

### Ansible Architecture

![image](https://github.com/discover-devops/my_ansible/assets/53135263/f05eb9a6-e777-45b6-969e-6cb27547a208)


#### 1. **Control Node:**
The machine where Ansible is installed and from which automation tasks are run. It contains the inventory, playbooks, and roles necessary for defining and executing tasks.

#### 2. **Managed Nodes:**
The servers or systems that Ansible manages. Ansible connects to these nodes via SSH (Secure Shell) for Linux systems or WinRM (Windows Remote Management) for Windows systems.

#### 3. **Inventory:**
A file or set of files that specify the managed nodes and their connection details. It defines the target systems where Ansible will execute tasks.

#### 4. **Playbooks:**
Ansible uses playbooks to describe automation jobs. Playbooks are written in YAML and define a series of tasks to be executed on managed nodes.

#### 5. **Modules:**
The building blocks of Ansible automation. Modules are small programs that carry out specific tasks, such as managing packages, files, services, etc. Ansible includes a broad set of built-in modules.

#### 6. **Plugins:**
Extend the functionality of Ansible by adding features like custom connection types, inventory scripts, and callbacks. Plugins can be used to integrate Ansible with other tools and systems.

### How Ansible Works

![image](https://github.com/discover-devops/my_ansible/assets/53135263/b4ca5b84-1adf-48c0-b9db-5add8409360d)



1. **Inventory Processing:**
   - Ansible reads the inventory to determine the list of managed nodes and their connection details.

2. **SSH/WinRM Connection:**
   - Ansible connects to managed nodes using SSH for Linux systems or WinRM for Windows systems. It establishes a secure connection to execute tasks.

3. **Playbook Execution:**
   - Playbooks describe a set of plays, each consisting of tasks. Ansible executes tasks sequentially on managed nodes.

4. **Modules Execution:**
   - Modules are used to perform specific tasks on managed nodes. Ansible ships with a wide range of modules for various purposes.

5. **Facts Gathering:**
   - Ansible collects system information, known as facts, from managed nodes. These facts can be used in playbooks for conditional execution.

6. **Idempotence:**
   - Ansible ensures idempotence, meaning if a task is executed multiple times, it has the same effect as running it once. This reduces the risk of unintended changes.

7. **Reporting and Logging:**
   - Ansible provides detailed output, including success or failure status for each task. This aids in troubleshooting and monitoring.

8. **Handlers:**
   - Handlers are tasks triggered by specific events in playbooks. For example, restarting a service only if a configuration file is changed.

9. **Roles:**
   - Roles are a way to organize and structure playbooks. They provide a convenient method for reusing and sharing Ansible code.



![image](https://github.com/discover-devops/my_ansible/assets/53135263/a5ffda62-61d5-4134-8ced-e8917735d136)



### Dive Deep into Ansible (Markdown Format)

**1. Control Node Setup:**
   - Ensure Ansible is installed on the control node. Use package managers or download the source from the [official Ansible website](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

```bash
# Example for installing Ansible on Ubuntu
sudo apt update
sudo apt install ansible
```

**2. Inventory Configuration:**
   - Create an inventory file (`inventory.ini`) listing the IP addresses or hostnames of managed nodes.

```ini
# inventory.ini
[web_servers]
192.168.1.10
192.168.1.11
```

**3. Playbook Creation:**
   - Write a playbook (`deploy_web_app.yml`) to install a web server on managed nodes.

```yaml
# deploy_web_app.yml
---
- name: Deploy Web App
  hosts: web_servers
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache service
      service:
        name: apache2
        state: started
```

**4. Execute Playbook:**
   - Run the playbook using the `ansible-playbook` command.

```bash
ansible-playbook -i inventory.ini deploy_web_app.yml
```

**5. Explore Modules:**
   - Ansible provides numerous modules. For example, use the `file` module to manage files.

```yaml
- name: Create a file
  hosts: web_servers
  tasks:
    - name: Create index.html
      file:
        path: /var/www/html/index.html
        state: touch
```

**6. Utilize Facts:**
   - Gather system information using facts.

```yaml
- name: Display facts
  hosts: web_servers
  tasks:
    - name: Show facts
      debug:
        var: ansible_facts
```

**7. Handlers for Restart:**
   - Use handlers to restart services.

```yaml
- name: Restart Apache if config changes
  hosts: web_servers
  tasks:
    - name: Copy Apache config
      copy:
        src: /path/to/new_config.conf
        dest: /etc/apache2/config.conf
        notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```

**8. Roles for Organization:**
   - Structure playbooks using roles.

```yaml
# site.yml
---
- name: Deploy Web App
  hosts: web_servers
  roles:
    - web_app_role
```

   - Organize tasks into a role directory structure.

```plaintext
web_app_role/
|-- tasks/
|   |-- main.yml
|-- handlers/
|   |-- main.yml
|-- files/
|-- templates/
|-- vars/
|   |-- main.yml
|-- defaults/
|   |-- main.yml
|-- meta/
|   |-- main.yml
```

This markdown dive deep guide provides a comprehensive overview of Ansible, its architecture, and a step-by-step walkthrough of key concepts and functionalities. Explore further by diving into Ansible documentation and trying out more complex scenarios and use cases.
