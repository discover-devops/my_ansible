# 04 Variables Lab 2: Variables From a Separate File and as a Prompt

## The real problem first

In Variables Lab 1 we wrote the variables at the top of the playbook. That works for 4 variables.

Now think about a real company:

- 40 variables for one application
- Different values for dev, test and prod
- The same steps for every environment

If all values live inside the playbook, you need 3 copies of the playbook, one per environment. When a step changes, you must fix it in 3 places.

What if the steps and the values lived in separate files?

---

## Concept: Separate the WHAT from the WITH WHICH VALUES

- The playbook says WHAT to do: install, copy, restart
- The vars file says WITH WHICH VALUES: which package, which file, which service

One playbook. Many vars files. Pick the vars file you need.

Ansible loads a vars file with the keyword vars_files.

---

## Prerequisites

Complete Variables Lab 1 (file 03). You need:

```bash
cd ~/ansible-labs
ls index.html
ansible all -m ping
```

---

# Part A: Variables From a Separate File

## Step 1: Create the variable file

```bash
cd ~/ansible-labs
cat > vars.yaml <<'EOF'
pkg_name: apache2
pkg_state: present
file_source: /home/ubuntu/ansible-labs/index.html
file_destination: /var/www/html/index.html
pkg_start: apache2
EOF
cat vars.yaml
```

Notice: there is no --- and no vars: keyword here. The file contains only key: value pairs.

There is a new variable pkg_start. Keep it in mind, it will make sense in Step 4.

---

## Step 2: Create the playbook that loads the file

```bash
cat > playbook-external-vars.yaml <<'EOF'
---
- name: Deploy application with external variables
  hosts: all
  become: true

  vars_files:
    - vars.yaml

  tasks:
    - name: Install {{ pkg_name }}
      package:
        name: "{{ pkg_name }}"
        state: "{{ pkg_state }}"

    - name: Copy index.html
      copy:
        src: "{{ file_source }}"
        dest: "{{ file_destination }}"
      notify: Restart {{ pkg_start }}

  handlers:
    - name: Restart {{ pkg_start }}
      service:
        name: "{{ pkg_start }}"
        state: restarted
EOF
```

What changed compared to Lab 1:

| Lab 1 | Lab 2 |
|---|---|
| vars: section inside the playbook | vars_files: points to vars.yaml |
| Values in the playbook | Values in a separate file |
| Service name = pkg_name | Service name = pkg_start |

---

## Step 3: Run it

```bash
ansible-playbook playbook-external-vars.yaml
curl http://SLAVE1_PRIVATE_IP
```

The output looks exactly like Lab 1. The playbook got the same values, just from a different place.

---

## Step 4: Change only the vars file

Now install Docker. We will NOT touch the playbook. Only vars.yaml changes.

```bash
cat > vars.yaml <<'EOF'
pkg_name: docker.io
pkg_state: present
file_source: /home/ubuntu/ansible-labs/index.html
file_destination: /home/ubuntu/index.html
pkg_start: docker
EOF
ansible-playbook playbook-external-vars.yaml
ansible all -a "systemctl is-active docker"
```

- systemctl is-active docker shows active on both slaves
- The playbook file was not changed at all

Now the aha moment. Look at these two lines:

```text
pkg_name: docker.io
pkg_start: docker
```

The package is called docker.io, but the service is called docker. If we used one variable for both, the restart would fail with "service docker.io not found".

That is why we keep the package name and the service name as two separate variables.

| Tool | Package name | Service name |
|---|---|---|
| Apache | apache2 | apache2 |
| Docker | docker.io | docker |

---

## Step 5: Put Apache back

```bash
cat > vars.yaml <<'EOF'
pkg_name: apache2
pkg_state: present
file_source: /home/ubuntu/ansible-labs/index.html
file_destination: /var/www/html/index.html
pkg_start: apache2
EOF
ansible-playbook playbook-external-vars.yaml
```

---

# Part B: Variables as a Prompt

## Concept: Ask for values when the playbook starts

Sometimes you want the playbook to ask you questions when it starts. This is done with vars_prompt.

Think before reading on: is this a good idea for a Jenkins pipeline that runs at 2 AM?

---

## Step 6: Create the prompt playbook

```bash
cat > playbook-prompt.yaml <<'EOF'
---
- name: Deploy application with prompted variables
  hosts: all
  become: true

  vars_prompt:
    - name: pkg_name
      prompt: "Enter the package you want to install"
      private: no

    - name: pkg_state
      prompt: "Enter package state (present or absent)"
      private: no

    - name: file_source
      prompt: "Enter source file path"
      private: no

    - name: file_destination
      prompt: "Enter destination file path"
      private: no

    - name: pkg_start
      prompt: "Enter service name to restart"
      private: no

  tasks:
    - name: Install {{ pkg_name }}
      package:
        name: "{{ pkg_name }}"
        state: "{{ pkg_state }}"

    - name: Copy index.html
      copy:
        src: "{{ file_source }}"
        dest: "{{ file_destination }}"
      notify: Restart {{ pkg_start }}

  handlers:
    - name: Restart {{ pkg_start }}
      service:
        name: "{{ pkg_start }}"
        state: restarted
EOF
```

| Keyword | Meaning |
|---|---|
| vars_prompt | List of questions asked when the playbook starts |
| name | The variable that stores your answer |
| prompt | The question shown on screen |
| private: no | You can see what you type. Use private: yes for passwords |

---

## Step 7: Run it and answer the questions

```bash
ansible-playbook playbook-prompt.yaml
```

Type these answers:

```text
Enter the package you want to install: apache2
Enter package state (present or absent): present
Enter source file path: /home/ubuntu/ansible-labs/index.html
Enter destination file path: /var/www/html/prompt.html
Enter service name to restart: apache2
```

Verify:

```bash
curl http://SLAVE1_PRIVATE_IP/prompt.html
```

---

## Concept: Where each method is used

Back to the question: a pipeline at 2 AM has nobody to type answers. So vars_prompt is for manual, one-time jobs. A pipeline passes values with -e instead.

| Method | Where the value lives | Best for |
|---|---|---|
| vars | Inside the playbook | Small playbooks, fixed values |
| vars_files | Separate YAML file | Real projects, one file per environment |
| vars_prompt | Typed when the playbook starts | Manual one-time jobs |
| -e | Command line | Pipelines, quick overrides. Always wins |

---

## Interview Questions

1. What is the benefit of keeping variables in a separate file?
2. How would you use one playbook for dev, test and prod?
3. Why do we keep pkg_name and pkg_start as separate variables?
4. Why is vars_prompt not used in CI/CD pipelines? What is used instead?
5. When would you set private: yes?

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- LinkedIn: https://www.linkedin.com/in/chaubeyrahul/
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
