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

This example demonstrates how variables can make your playbook more flexible and reusable by allowing you to easily customize the behavior of tasks without modifying the playbook itself.
