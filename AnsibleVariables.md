Ansible variables are placeholders that store data that can be used across Ansible playbooks, roles, and tasks. They allow you to define values once and reuse them throughout your Ansible configuration, making your playbooks more dynamic and easier to maintain.

Variables in Ansible can be defined at different levels:

1. **Global Variables**: These are defined in the Ansible configuration files (`ansible.cfg`) or inventory files and are accessible to all playbooks and roles.

2. **Playbook Variables**: These are defined within a playbook and are accessible only within that playbook.

3. **Role Variables**: These are defined within an Ansible role and are accessible only within that role, unless explicitly overridden.

Variables can hold various types of data, including strings, numbers, lists, dictionaries, and even complex data structures.

Here's an example of how you can define and use variables in an Ansible playbook:

Let's say you have a playbook (`example_playbook.yml`) that installs a package and creates a file with some content. You want to use variables to define the package name, file path, and content.

```yaml
# example_playbook.yml

---
- name: Example Playbook
  hosts: servers
  become: yes
  vars:
    package_name: "nginx"
    file_path: "/etc/nginx/nginx.conf"
    file_content: |
      user www-data;
      worker_processes auto;
      ...
  tasks:
    - name: Install nginx package
      apt:
        name: "{{ package_name }}"
        state: present

    - name: Create nginx configuration file
      template:
        src: templates/nginx.conf.j2
        dest: "{{ file_path }}"
      vars:
        content: "{{ file_content }}"
```

In this example:

- `package_name`, `file_path`, and `file_content` are variables defined at the playbook level.
- The `apt` task installs the package specified by the `package_name` variable.
- The `template` task uses a Jinja2 template (`nginx.conf.j2`) to create the nginx configuration file at the path specified by the `file_path` variable, and the content of the file is determined by the `file_content` variable.

You can then run this playbook using the `ansible-playbook` command:

```
ansible-playbook example_playbook.yml
```

Here in this example demonstrates how variables can make your playbook more flexible and reusable by allowing you to easily customize the behavior of tasks without modifying the playbook itself.


Let's take examples of defining global variables in both the inventory file and the Ansible configuration file:

### Inventory File Example:

In your inventory file (e.g., `inventory.ini`), you can define global variables under a `[all:vars]` section. These variables will be available to all hosts in the inventory:

```ini
[all:vars]
ansible_user = my_user
ansible_ssh_private_key_file = /path/to/private/key.pem
```

In this example:

- `ansible_user` is a global variable that specifies the SSH user to use when connecting to hosts.
- `ansible_ssh_private_key_file` is a global variable that specifies the path to the private SSH key file to use for authentication.

### Ansible Configuration File Example (ansible.cfg):

In your Ansible configuration file (typically `ansible.cfg`), you can define global variables under the `[defaults]` section:

```ini
[defaults]
remote_user = my_user
private_key_file = /path/to/private/key.pem
```

In this example:

- `remote_user` is a global variable that specifies the SSH user to use when connecting to hosts.
- `private_key_file` is a global variable that specifies the path to the private SSH key file to use for authentication.

### How to Use These Variables:

Once defined, these variables can be referenced in your playbooks or roles. For example:

```yaml
---
- name: Example Playbook
  hosts: all
  tasks:
    - name: Print SSH user
      debug:
        msg: "SSH user is {{ ansible_user }}"

    - name: Print private key file path
      debug:
        msg: "Private key file path is {{ private_key_file }}"
```

In this playbook:

- `{{ ansible_user }}` references the `ansible_user` variable defined in either the inventory file or the Ansible configuration file.
- `{{ private_key_file }}` references the `private_key_file` variable defined in either the inventory file or the Ansible configuration file.

These variables will be substituted with their respective values when the playbook is executed.


Let's create a real-world example where we install Nginx on an Ubuntu server using Ansible. 
We'll define the variables in a separate YAML file and then use those variables in the playbook.

### Variable File (nginx_vars.yml):

Create a file named `nginx_vars.yml` to define the variables:

```yaml
# nginx_vars.yml

nginx_package: nginx
```

In this file:

- `nginx_package` is the variable that specifies the name of the Nginx package.

### Playbook (install_nginx.yml):

Create a playbook named `install_nginx.yml` to install Nginx using the variables defined in the `nginx_vars.yml` file:

```yaml
# install_nginx.yml

---
- name: Install Nginx on Ubuntu
  hosts: ubuntu_servers
  become: yes
  vars_files:
    - nginx_vars.yml
  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: "{{ nginx_package }}"
        state: present

    - name: Start Nginx service
      service:
        name: nginx
        state: started
```

In this playbook:

- We're specifying the hosts where we want to install Nginx (`ubuntu_servers`).
- We're using the `vars_files` directive to include the variables defined in the `nginx_vars.yml` file.
- We're using the `{{ nginx_package }}` variable to specify the Nginx package name during installation.

### Running the Playbook:

To run the playbook:

```
ansible-playbook -i inventory.ini install_nginx.yml
```

Replace `inventory.ini` with your inventory file containing the list of Ubuntu servers.

### Explanation:

- By separating variables into a separate file (`nginx_vars.yml`), we can easily manage and reuse them across multiple playbooks.
- If we need to change the Nginx package name or add more variables, we only need to update the `nginx_vars.yml` file, and the changes will be reflected in all playbooks that use it.
- This approach promotes modularity and maintainability in our Ansible configuration.


Let's create another example where we install MySQL on Ubuntu servers using Ansible. We'll follow a similar approach by defining variables in a separate YAML file and then using those variables in the playbook.

### Variable File (mysql_vars.yml):

Create a file named `mysql_vars.yml` to define the variables:

```yaml
# mysql_vars.yml

mysql_package: mysql-server
mysql_root_password: "your_mysql_root_password"
```

In this file:

- `mysql_package` is the variable that specifies the name of the MySQL package.
- `mysql_root_password` is the variable that specifies the root password for MySQL.

### Playbook (install_mysql.yml):

Create a playbook named `install_mysql.yml` to install MySQL using the variables defined in the `mysql_vars.yml` file:

```yaml
# install_mysql.yml

---
- name: Install MySQL on Ubuntu
  hosts: ubuntu_servers
  become: yes
  vars_files:
    - mysql_vars.yml
  tasks:
    - name: Install MySQL
      apt:
        name: "{{ mysql_package }}"
        state: present

    - name: Secure MySQL installation
      debconf:
        name: "{{ mysql_package }}"
        question: "{{ item.key }}"
        value: "{{ item.value }}"
        vtype: "{{ item.vtype | default('string') }}"
      with_dict:
        mysql-server-{{ ansible_distribution_release }}-passwords
```

In this playbook:

- We're specifying the hosts where we want to install MySQL (`ubuntu_servers`).
- We're using the `vars_files` directive to include the variables defined in the `mysql_vars.yml` file.
- We're using the `{{ mysql_package }}` variable to specify the MySQL package name during installation.
- We're using the `debconf` module to set the root password for MySQL. Note that this task is specific to Ubuntu and may need adjustment for other distributions.

### Running the Playbook:

To run the playbook:

```
ansible-playbook -i inventory.ini install_mysql.yml
```

Replace `inventory.ini` with your inventory file containing the list of Ubuntu servers.

### Explanation:

- Similar to the previous example, by separating variables into a separate file (`mysql_vars.yml`), we can easily manage and reuse them across multiple playbooks.
- If we need to change the MySQL package name or root password, we only need to update the `mysql_vars.yml` file, and the changes will be reflected in all playbooks that use it.
- This approach maintains modularity and simplifies maintenance in our Ansible configuration.



We can create a playbook to uninstall both Nginx and MySQL from the Ubuntu servers using the same pattern of variables defined in separate YAML files.

### Playbook (uninstall_packages.yml):

Create a playbook named `uninstall_packages.yml` to uninstall both Nginx and MySQL using the variables defined in their respective variable files:

```yaml
# uninstall_packages.yml

---
- name: Uninstall Nginx and MySQL from Ubuntu
  hosts: ubuntu_servers
  become: yes
  vars_files:
    - nginx_vars.yml
    - mysql_vars.yml
  tasks:
    - name: Remove Nginx
      apt:
        name: "{{ nginx_package }}"
        state: absent

    - name: Remove MySQL
      apt:
        name: "{{ mysql_package }}"
        state: absent
```

In this playbook:

- We're specifying the hosts where we want to uninstall Nginx and MySQL (`ubuntu_servers`).
- We're using the `vars_files` directive to include the variables defined in the `nginx_vars.yml` and `mysql_vars.yml` files.
- We're using the `{{ nginx_package }}` and `{{ mysql_package }}` variables to specify the package names of Nginx and MySQL respectively during uninstallation.

### Running the Playbook:

To run the playbook:

```
ansible-playbook -i inventory.ini uninstall_packages.yml
```

Replace `inventory.ini` with your inventory file containing the list of Ubuntu servers.

### Explanation:

- This playbook follows the same pattern of using separate variable files (`nginx_vars.yml` and `mysql_vars.yml`) to define variables for Nginx and MySQL respectively.
- By doing so, we can easily manage and reuse these variables across different playbooks.
- The tasks in the playbook utilize these variables to uninstall the specified packages (`nginx_package` and `mysql_package`) from the Ubuntu servers.
- This approach maintains consistency and promotes modularity in our Ansible configuration.
