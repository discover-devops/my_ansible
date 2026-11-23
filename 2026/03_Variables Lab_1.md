# 03 Variables Lab 1: Variables Inside the Playbook

## The real problem first

Look at this playbook and count how many times apache2 appears:

```yaml
- name: Install apache2
  package:
    name: apache2
    state: present

- name: Restart apache2
  service:
    name: apache2
    state: restarted
```

apache2 appears 4 times. Now your team says: "From today the web page goes to a different folder" or "use a different package".

How many places will you edit? What if you miss one?

This is called hardcoding. The value is typed directly in many places. The bigger the playbook, the bigger the risk.

---

## Concept: What is a variable

A variable is a name that holds a value. You define it once and use the name everywhere.

```yaml
vars:
  pkg_name: apache2
```

Then anywhere in the playbook:

```yaml
name: "{{ pkg_name }}"
```

The double curly braces {{ }} mean: put the value of this variable here. Ansible replaces {{ pkg_name }} with apache2 before running the task. This syntax comes from Jinja2, the template engine Ansible uses.

Change the value in one place, and every task that uses it follows.

---

## Prerequisites

Lab folder with inventory and ansible.cfg (see file 01):

```bash
cd ~/ansible-labs
ansible all -m ping
```

Port 80 must be free on both slaves. If nginx is running on a slave, remove it first, because Apache and nginx cannot both use port 80:

```bash
ansible all -m apt -a "name=nginx state=absent purge=yes update_cache=yes" -b
```

---

## Step 1: Create the web page on the control node

```bash
cd ~/ansible-labs
cat > index.html <<'EOF'
<h1>Hello world</h1>
<p>This is my Ansible website</p>
EOF
pwd
```

The file is at /home/ubuntu/ansible-labs/index.html. Ansible will copy it from the control node to the slaves.

---

## Step 2: Create the playbook with variables

```bash
cat > playbook-vars.yaml <<'EOF'
---
- name: Deploy Apache application with variables
  hosts: all
  become: true

  vars:
    pkg_name: apache2
    pkg_state: present
    file_source: /home/ubuntu/ansible-labs/index.html
    file_destination: /var/www/html/index.html

  tasks:
    - name: Install {{ pkg_name }}
      package:
        name: "{{ pkg_name }}"
        state: "{{ pkg_state }}"

    - name: Copy index.html
      copy:
        src: "{{ file_source }}"
        dest: "{{ file_destination }}"
      notify: Restart {{ pkg_name }}

  handlers:
    - name: Restart {{ pkg_name }}
      service:
        name: "{{ pkg_name }}"
        state: restarted
EOF
```

The variables:

| Variable | Value | Where it is used |
|---|---|---|
| pkg_name | apache2 | task name, package, handler name, service |
| pkg_state | present | package state |
| file_source | /home/ubuntu/ansible-labs/index.html | copy src, on the control node |
| file_destination | /var/www/html/index.html | copy dest, on the slaves |

Two rules to remember:

- When a value STARTS with {{ it must be inside quotes: name: "{{ pkg_name }}"
- Inside a longer text, quotes are not needed: name: Install {{ pkg_name }}

---

## Step 3: Run it

```bash
ansible-playbook playbook-vars.yaml --syntax-check
ansible-playbook playbook-vars.yaml
```

Look at the output carefully:

```text
TASK [Install apache2]
TASK [Copy index.html]
RUNNING HANDLER [Restart apache2]
```

We wrote Install {{ pkg_name }} in the file, but the output shows Install apache2. Ansible filled in the variable even inside the task name.

Verify:

```bash
curl http://SLAVE1_PRIVATE_IP
curl http://SLAVE2_PRIVATE_IP
```

---

## Step 4: Change a value in ONE place

Before you run this, guess: we will change only one line at the top. Which tasks will be affected?

```bash
sed -i 's#/var/www/html/index.html#/var/www/html/about.html#' playbook-vars.yaml
grep file_destination playbook-vars.yaml
ansible-playbook playbook-vars.yaml
curl http://SLAVE1_PRIVATE_IP/about.html
```

- sed changes file_destination in the vars section only
- grep shows the line now ends with about.html
- The copy task, which we did not touch, now creates about.html
- curl shows the page at the new address

Concept: one change at the top, the whole playbook follows. This is why we use variables.

---

## Step 5: Override a variable from the command line

```bash
ansible-playbook playbook-vars.yaml -e "file_destination=/var/www/html/contact.html"
curl http://SLAVE1_PRIVATE_IP/contact.html
grep file_destination playbook-vars.yaml
```

- -e means extra vars. It gives a value at run time
- The file still says about.html, but this run used contact.html
- Extra vars always win over values written in the playbook

Think: why is this useful in a CI/CD pipeline like Jenkins? The pipeline can pass a different value for dev, test and prod without editing the playbook.

---

## Step 6: Put it back

```bash
sed -i 's#/var/www/html/about.html#/var/www/html/index.html#' playbook-vars.yaml
grep file_destination playbook-vars.yaml
```

---

## A word of caution

What if we change pkg_name from apache2 to nginx? The package and the handler would follow, but nginx would fight Apache for port 80. A variable makes a change easy. It does not make every change correct. Do not run that change on these servers.

---

## Common mistakes

| Mistake | What happens |
|---|---|
| name: {{ pkg_name }} without quotes | YAML syntax error |
| Typo in variable name, for example pkg_nme | Error: variable is undefined |
| Variable defined under tasks instead of vars | Variable is undefined |

---

## Interview Questions

1. What is hardcoding and why is it a problem?
2. Why must "{{ pkg_name }}" be in quotes when it starts the value?
3. What does -e do, and which wins: a value in vars or a value given with -e?
4. Which template engine gives Ansible the {{ }} syntax?
5. Give one example where changing a variable could break the playbook.

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- LinkedIn: https://www.linkedin.com/in/chaubeyrahul/
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
