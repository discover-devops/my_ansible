# 05 Loops Lab: One Task, Many Values

## The real problem first

Your team lead sends this message:

"Every new server needs git, apache2, unzip and curl. Also create accounts for rohit, virat, ishan and abhishek."

Without loops, your playbook looks like this:

```yaml
- name: Install git
  package:
    name: git
    state: present

- name: Install apache2
  package:
    name: apache2
    state: present

- name: Install unzip
  package:
    name: unzip
    state: present
```

And it keeps going. 4 packages + 4 users = 8 tasks that are almost the same. Only one word changes in each.

Ask yourself: next month 10 more users join. Will you copy and paste 10 more tasks?

---

## Concept: What is a loop

A loop gives one task a list of values. Ansible runs the task once for each value.

- loop: the list of values
- item: the current value in each round
- {{ item }}: put the current value here

```yaml
- name: Install packages
  package:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - apache2
```

Round 1: item = git. Round 2: item = apache2. One task, many values.

---

## Prerequisites

Lab folder with inventory and ansible.cfg (see file 01):

```bash
cd ~/ansible-labs
ansible all -m ping
ansible all -m apt -a "update_cache=yes" -b
```

The last command refreshes the package list on both slaves so every package can be found.

---

## Step 1: Create the loop playbook

```bash
cd ~/ansible-labs
cat > playbook-loops.yaml <<'EOF'
---
- name: Install packages and create users using loops
  hosts: all
  become: true

  tasks:
    - name: Install packages in a loop
      package:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - apache2
        - unzip
        - curl

    - name: Create users in a loop
      user:
        name: "{{ item }}"
        shell: /bin/bash
        create_home: yes
      loop:
        - rohit
        - virat
        - ishan
        - abhishek
EOF
```

How the first loop runs:

| Round | item |
|---|---|
| 1 | git |
| 2 | apache2 |
| 3 | unzip |
| 4 | curl |

Note: Linux usernames must be lowercase. Ubuntu rejects capital letters in usernames by default, so we use rohit, not Rohit.

---

## Step 2: Run it

```bash
ansible-playbook playbook-loops.yaml --syntax-check
ansible-playbook playbook-loops.yaml
```

Look at the output. Each value gets its own line:

```text
TASK [Install packages in a loop]
ok: [slave-1] => (item=git)
ok: [slave-1] => (item=apache2)
changed: [slave-1] => (item=unzip)
...
TASK [Create users in a loop]
changed: [slave-1] => (item=rohit)
changed: [slave-1] => (item=virat)
```

- ok: the package was already installed. git and curl come with Ubuntu, apache2 may be there from earlier labs
- changed: Ansible installed or created it

---

## Step 3: Verify

```bash
ansible all -a "id rohit"
ansible all -a "git --version"
ansible all -a "which unzip"
```

All 4 users exist and all 4 packages are installed on both slaves.

---

## Step 4: Run it again

Guess before you run: how many items will show changed?

```bash
ansible-playbook playbook-loops.yaml
```

Every item shows ok. The loop is idempotent too, for every single item.

---

## Step 5: Add one more user

A new joiner, shubman, needs an account. Add one line to the list:

```bash
sed -i 's/        - abhishek/        - abhishek\n        - shubman/' playbook-loops.yaml
grep -A6 "Create users" playbook-loops.yaml
ansible-playbook playbook-loops.yaml
```

Look at the output: only item=shubman shows changed. Everything else is ok.

Concept: adding a user is one line, not one new task. This is why loops make playbooks shorter and easier to maintain.

---

## Step 6: Loops with variables

Loops and variables work together. Move the lists into vars:

```bash
cat > playbook-loops-vars.yaml <<'EOF'
---
- name: Loops with variables
  hosts: all
  become: true

  vars:
    packages:
      - git
      - apache2
      - unzip
      - curl
    team_users:
      - rohit
      - virat
      - ishan
      - abhishek
      - shubman

  tasks:
    - name: Install packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"

    - name: Create users
      user:
        name: "{{ item }}"
        shell: /bin/bash
        create_home: yes
      loop: "{{ team_users }}"
EOF
ansible-playbook playbook-loops-vars.yaml
```

- packages and team_users are variables that hold a list
- loop: "{{ team_users }}" loops over that list
- Now the tasks never change. You only edit the lists

---

## Clean up

Remove the users, using a loop again:

```bash
cat > remove-users.yaml <<'EOF'
---
- name: Remove lab users
  hosts: all
  become: true
  tasks:
    - name: Remove users
      user:
        name: "{{ item }}"
        state: absent
        remove: yes
      loop:
        - rohit
        - virat
        - ishan
        - abhishek
        - shubman
EOF
ansible-playbook remove-users.yaml
```

---

## Common mistakes

| Mistake | What happens |
|---|---|
| name: {{ item }} without quotes | YAML syntax error |
| loop: at the wrong indentation | loop is treated as a module argument and fails |
| Capital letters in usernames | useradd: invalid user name |
| Using item outside a loop | Error: item is undefined |

---

## Interview Questions

1. What is the difference between loop and item?
2. Why must "{{ item }}" be in quotes?
3. How do you loop over a list stored in a variable?
4. After adding one user to the list, why does only that user show changed?
5. Why does Ubuntu reject a username like Rohit?

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- LinkedIn: https://www.linkedin.com/in/chaubeyrahul/
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
