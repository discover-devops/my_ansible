Below are detailed explanations and step-by-step labs for each topic in Module 3: Ansible Modules and Playbooks.

### Commonly Used Modules

#### File Module
The `file` module is used to manage file properties. You can create, delete, or modify files and directories.

**Example:**
Create a directory on a target machine.

**Playbook:**
```yaml
---
- name: Create a directory
  hosts: all
  tasks:
    - name: Create directory /tmp/test_directory
      file:
        path: /tmp/test_directory
        state: directory
        mode: '0755'
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano create_directory.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory create_directory.yml
    ```

#### Package Module
The `package` module handles package installation and management using the system’s package manager.

**Example:**
Install the `nginx` package.

**Playbook:**
```yaml
---
- name: Install nginx
  hosts: all
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano install_nginx.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory install_nginx.yml
    ```

#### Service Module
The `service` module is used to manage services on remote hosts.

**Example:**
Start and enable the `nginx` service.

**Playbook:**
```yaml
---
- name: Start and enable nginx
  hosts: all
  tasks:
    - name: Start and enable nginx service
      service:
        name: nginx
        state: started
        enabled: true
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano start_nginx.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory start_nginx.yml
    ```

#### Command Module
The `command` module allows you to run arbitrary commands on remote hosts.

**Example:**
Run a command to list the contents of a directory.

**Playbook:**
```yaml
---
- name: List directory contents
  hosts: all
  tasks:
    - name: List contents of /tmp
      command: ls /tmp
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano list_directory.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory list_directory.yml
    ```

#### Shell Module
The `shell` module executes shell commands on remote hosts.

**Example:**
Run a shell command to create a file with specific content.

**Playbook:**
```yaml
---
- name: Create a file with content
  hosts: all
  tasks:
    - name: Create file /tmp/test_file.txt with content
      shell: echo "Hello World" > /tmp/test_file.txt
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano create_file_with_content.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory create_file_with_content.yml
    ```

#### User Module
The `user` module manages user accounts.

**Example:**
Create a new user account.

**Playbook:**
```yaml
---
- name: Create a new user
  hosts: all
  tasks:
    - name: Create user johndoe
      user:
        name: johndoe
        state: present
        groups: sudo
        shell: /bin/bash
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano create_user.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory create_user.yml
    ```

### Advanced Playbook Features

#### Variables and Facts
Variables and facts allow for dynamic content in playbooks. Facts are gathered from remote systems, and variables are defined within playbooks or inventory files.

**Example:**
Use a variable to create a directory.

**Playbook:**
```yaml
---
- name: Create a directory using a variable
  hosts: all
  vars:
    my_directory: /tmp/my_directory
  tasks:
    - name: Create the directory
      file:
        path: "{{ my_directory }}"
        state: directory
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano create_directory_with_variable.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory create_directory_with_variable.yml
    ```

#### Conditionals and Loops
Conditionals and loops allow tasks to be executed based on certain conditions or repeated multiple times.

**Example:**
Create multiple directories using a loop.

**Playbook:**
```yaml
---
- name: Create multiple directories
  hosts: all
  tasks:
    - name: Create directories
      file:
        path: "{{ item }}"
        state: directory
      loop:
        - /tmp/dir1
        - /tmp/dir2
        - /tmp/dir3
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano create_multiple_directories.yml
    ```
2. Copy and paste the playbook example into the file.
3. Run the playbook:
    ```sh
    ansible-playbook -i inventory create_multiple_directories.yml
    ```

#### Handlers and Notifications
Handlers are tasks that are triggered by other tasks, often used to restart services after configuration changes.

**Example:**
Restart `nginx` service after a configuration change.

**Playbook:**
```yaml
---
- name: Change nginx config and restart service
  hosts: all
  tasks:
    - name: Copy nginx config
      copy:
        src: /path/to/local/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify:
        - Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**Step-by-Step Lab:**
1. Create a playbook file:
    ```sh
    nano change_nginx_config.yml
    ```
2. Copy and paste the playbook example into the file.
3. Ensure you have the `nginx.conf` file at the specified source path.
4. Run the playbook:
    ```sh
    ansible-playbook -i inventory change_nginx_config.yml
    ```

These labs provide practical examples for each topic, helping you understand how to use Ansible modules and advanced playbook features effectively.
