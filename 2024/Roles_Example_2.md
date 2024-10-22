### **Step-by-Step Guide to Use Ansible Roles for Amazon Linux Slaves**

Ansible **roles** allow you to organize your playbooks into reusable components. This guide will walk you through the process of creating and using Ansible roles to manage Amazon Linux slaves, specifically installing and configuring NGINX.

---

### **Step 1: Set Up Directory Structure for Ansible Roles**

Ansible has a specific directory structure for roles. To set up this structure, create the necessary directories:

```bash
mkdir -p ~/ansible-project/roles/nginx/{tasks,templates,handlers,vars,defaults}
```

This will create the following structure:

```
ansible-project/
  ├── roles/
      └── nginx/
          ├── tasks/
          ├── templates/
          ├── handlers/
          ├── vars/
          └── defaults/
```

### **Step 2: Create Role Files**

#### 1. **Define Tasks in `tasks/main.yml`**

The `tasks` file defines the actions that your role will perform, such as installing and configuring NGINX.

```bash
nano ~/ansible-project/roles/nginx/tasks/main.yml
```

Add the following content to install and configure NGINX:

```yaml
---
- name: Install NGINX on Amazon Linux
  yum:
    name: nginx
    state: present

- name: Start and enable NGINX
  service:
    name: nginx
    state: started
    enabled: yes

- name: Deploy custom index.html
  template:
    src: index.html.j2
    dest: /usr/share/nginx/html/index.html
    owner: nginx
    group: nginx
    mode: '0644'
```

---

#### 2. **Create a Template in `templates/index.html.j2`**

Templates allow you to dynamically generate files based on variables. Create a template for NGINX's default web page:

```bash
nano ~/ansible-project/roles/nginx/templates/index.html.j2
```

Add the following content:

```html
<html>
  <head><title>Welcome to {{ site_title }}</title></head>
  <body><h1>Welcome to {{ site_title }}!</h1></body>
</html>
```

---

#### 3. **Create a Handler in `handlers/main.yml`**

Handlers are triggered when certain conditions are met, such as restarting a service if a configuration file changes:

```bash
nano ~/ansible-project/roles/nginx/handlers/main.yml
```

Add the following content:

```yaml
---
- name: Restart NGINX
  service:
    name: nginx
    state: restarted
```

---

#### 4. **Define Variables in `vars/main.yml`**

Variables are used to store values that may change between hosts or environments.

```bash
nano ~/ansible-project/roles/nginx/vars/main.yml
```

Add a variable for the website title:

```yaml
---
site_title: "NGINX on Amazon Linux"
```

---

### **Step 3: Create a Playbook that Uses the Role**

Now that you have created the role, you need to create a playbook that will use the role.

```bash
nano ~/ansible-project/nginx_playbook.yml
```

Add the following content to the playbook:

```yaml
---
- name: Deploy NGINX on Amazon Linux
  hosts: webservers
  become: yes
  roles:
    - nginx
```

This playbook will apply the NGINX role to all servers in the `webservers` group.

---

### **Step 4: Define Inventory File**

Next, define your inventory file with the servers you want to target.

```bash
nano ~/ansible-project/inventory
```

Add your Amazon Linux instances to the `webservers` group:

```ini
[webservers]
your-amazon-linux-instance-ip ansible_user=ec2-user ansible_ssh_private_key_file=/path/to/your/private_key.pem
```

---

### **Step 5: Run the Playbook**

Now that everything is set up, you can run the playbook using the following command:

```bash
cd ~/ansible-project
ansible-playbook -i inventory nginx_playbook.yml
```

This will:
- Install NGINX on all servers in the `webservers` group.
- Configure NGINX with a custom `index.html` page using the `site_title` variable.
- Start and enable the NGINX service.

---

### **Step 6: Verify the Installation**

After the playbook runs successfully, you can verify that NGINX is running by accessing the server's IP address in your browser:

```
http://your-amazon-linux-instance-ip
```

You should see a page that says "Welcome to NGINX on Amazon Linux!"

---

### **Conclusion**

You’ve now created and used an Ansible role to manage Amazon Linux slaves by installing and configuring NGINX. Using roles helps to modularize and simplify your Ansible playbooks, making them more reusable and maintainable.

