# Ansible Variables and Templates

#### Understanding Variables

**Theory and Concept:**
Variables in Ansible are used to store values that can be reused throughout playbooks and roles. They enable dynamic assignment of values and can be defined in various places such as inventory files, playbooks, roles, and more.

**Variable Precedence:**
Ansible uses a specific order to determine which variable value to use when multiple variables with the same name are defined. The order of precedence (from highest to lowest) is:
1. Extra vars (`ansible-playbook -e "var=value"`)
2. Task vars (with `include_vars`)
3. Block vars
4. Role and include vars
5. Play vars
6. Inventory group vars
7. Inventory host vars
8. Host facts
9. Playbook group_vars/all
10. Playbook group_vars/*
11. Playbook host_vars/*
12. Role defaults

**Types of Variables:**
- **Simple Variables:** Basic key-value pairs.
- **Lists:** Arrays of values.
- **Dictionaries:** Key-value pairs where values can be other variables, lists, or dictionaries.

**Step-by-Step Lab: Understanding Variables**

1. **Create a Playbook with Simple Variables:**
    ```sh
    nano variables_example.yml
    ```

    ```yaml
    ---
    - name: Example of Variables
      hosts: all
      vars:
        my_variable: "Hello World"
      tasks:
        - name: Print the variable
          debug:
            msg: "{{ my_variable }}"
    ```

2. **Run the Playbook:**
    ```sh
    ansible-playbook -i inventory variables_example.yml
    ```

#### Jinja2 Templating

**Theory and Concept:**
Jinja2 is a templating engine used in Ansible to generate files dynamically. Templates can include variables, loops, and conditional statements, allowing for dynamic and complex file generation.

**Creating and Using Templates:**
Templates are typically written in `.j2` files and can be used to create configuration files, scripts, or any text files.

**Step-by-Step Lab: Creating and Using Templates**

1. **Create a Template File:**
    ```sh
    mkdir templates
    nano templates/example.j2
    ```

    ```jinja
    # This is a Jinja2 template
    Hello, my name is {{ name }}
    ```

2. **Create a Playbook to Use the Template:**
    ```sh
    nano use_template.yml
    ```

    ```yaml
    ---
    - name: Use Jinja2 Template
      hosts: all
      vars:
        name: "Ansible User"
      tasks:
        - name: Generate file from template
          template:
            src: templates/example.j2
            dest: /tmp/example.txt
    ```

3. **Run the Playbook:**
    ```sh
    ansible-playbook -i inventory use_template.yml
    ```

4. **Verify the Output:**
    ```sh
    cat /tmp/example.txt
    ```

#### Looping and Conditional Statements in Templates

**Theory and Concept:**
Jinja2 templates support loops and conditionals, enabling dynamic content generation based on conditions and iterations over lists or dictionaries.

**Step-by-Step Lab: Looping and Conditional Statements in Templates**

1. **Update the Template File:**
    ```sh
    nano templates/example_with_loops.j2
    ```

    ```jinja
    # Users List
    {% for user in users %}
    User: {{ user.name }}
    Role: {{ user.role }}
    {% if user.admin %}
    Admin: Yes
    {% else %}
    Admin: No
    {% endif %}
    {% endfor %}
    ```

2. **Create a Playbook to Use the Updated Template:**
    ```sh
    nano use_template_with_loops.yml
    ```

    ```yaml
    ---
    - name: Use Jinja2 Template with Loops and Conditionals
      hosts: all
      vars:
        users:
          - name: "Alice"
            role: "Developer"
            admin: true
          - name: "Bob"
            role: "Tester"
            admin: false
      tasks:
        - name: Generate file from template with loops and conditionals
          template:
            src: templates/example_with_loops.j2
            dest: /tmp/example_with_loops.txt
    ```

3. **Run the Playbook:**
    ```sh
    ansible-playbook -i inventory use_template_with_loops.yml
    ```

4. **Verify the Output:**
    ```sh
    cat /tmp/example_with_loops.txt
    ```

#### Real-World Example in DevOps

**Scenario:**
You need to dynamically generate an Nginx configuration file based on the environment (e.g., development, staging, production) and deploy it to multiple servers.

**Steps:**
1. **Create a Jinja2 Template for Nginx Configuration:**
    ```sh
    mkdir -p templates/nginx
    nano templates/nginx/nginx.conf.j2
    ```

    ```jinja
    server {
        listen 80;
        server_name {{ server_name }};
        
        location / {
            proxy_pass http://{{ app_server }};
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    ```

2. **Create a Playbook to Deploy the Configuration:**
    ```sh
    nano deploy_nginx_config.yml
    ```

    ```yaml
    ---
    - name: Deploy Nginx Configuration
      hosts: webservers
      vars:
        server_name: "example.com"
        app_server: "localhost:5000"
      tasks:
        - name: Generate Nginx config from template
          template:
            src: templates/nginx/nginx.conf.j2
            dest: /etc/nginx/sites-available/default

        - name: Restart Nginx
          service:
            name: nginx
            state: restarted
    ```

3. **Run the Playbook:**
    ```sh
    ansible-playbook -i inventory deploy_nginx_config.yml
    ```

By understanding variables and templates in Ansible, you can create more dynamic, reusable, and efficient playbooks, enabling better management of configurations and deployments in a DevOps environment.
