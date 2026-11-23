# 02 Ansible Ad-hoc Commands vs Playbooks

## The real question

You already know two ways to talk to your servers with Ansible:

- Type one command and it runs immediately (ad-hoc)
- Write a YAML file and run it (playbook)

Both use the same modules. Both connect over SSH. So why do we need two ways? And when should you use which one?

Keep that question in mind. By the end of this lab you will do the same job both ways and decide for yourself.

---

## Concept: Ad-hoc command

An ad-hoc command is one line, run directly from the terminal, for a quick one-time task.

```text
ansible <host pattern> -m <module> -a "<arguments>" -b
```

| Part | Meaning |
|---|---|
| ansible | The ad-hoc tool |
| host pattern | all, a group, or a host name |
| -m | Module to use |
| -a | Arguments for the module |
| -b | Become root (sudo) |

If you do not give -m, Ansible uses the command module and simply runs a shell command.

## Concept: Playbook

A playbook is a YAML file that stores many tasks in order. You run it with ansible-playbook.

The same module arguments you pass with -a become key: value lines in the playbook.

---

## Prerequisites

Lab folder with inventory and ansible.cfg (see file 01):

```bash
cd ~/ansible-labs
cat inventory.ini
ansible all -m ping
```

---

## Part A: Ad-hoc commands for quick work

### Step 1: Quick checks on all servers

```bash
ansible all -a "uptime"
ansible all -a "df -h /"
ansible all -m setup -a "filter=ansible_distribution*"
```

- uptime: how long each server has been running
- df -h /: disk space on each server
- setup with a filter: Ansible facts about the OS. Useful to check versions quickly

Notice: nothing was installed or changed. This is what ad-hoc is perfect for, asking questions.

### Step 2: A quick one-time fix

```bash
ansible all -m user -a "name=deploy shell=/bin/bash create_home=yes" -b
ansible all -a "id deploy"
```

One user, needed right now, on all servers. One command. Done.

### Step 3: Target one host or one group

```bash
ansible slave-1 -a "hostname"
ansible MyDBServer -a "hostname"
ansible 'MyWebServer:MyDBServer' -a "hostname"
```

| Pattern | Runs on |
|---|---|
| slave-1 | one host |
| MyDBServer | one group |
| 'MyWebServer:MyDBServer' | both groups. Quotes are needed because of the colon |

---

## Part B: The same job, two ways

Job: on the web servers, install the tree package, create a file /tmp/team.txt, and create a user webadmin.

### Step 4: Ad-hoc way

```bash
ansible MyWebServer -m apt -a "name=tree state=present update_cache=yes" -b
ansible MyWebServer -m copy -a "content='Managed by Ansible' dest=/tmp/team.txt" -b
ansible MyWebServer -m user -a "name=webadmin shell=/bin/bash create_home=yes" -b
```

3 commands. Now close your terminal. Next week, can you repeat this exactly? Can your teammate?

### Step 5: Playbook way

```bash
cat > web-setup.yml <<'EOF'
---
- name: Web server setup
  hosts: MyWebServer
  become: yes
  tasks:
    - name: Install tree
      apt:
        name: tree
        state: present
        update_cache: yes

    - name: Create team file
      copy:
        content: "Managed by Ansible\n"
        dest: /tmp/team.txt

    - name: Create webadmin user
      user:
        name: webadmin
        shell: /bin/bash
        create_home: yes
EOF
ansible-playbook web-setup.yml
```

Compare them side by side:

| Ad-hoc | Playbook |
|---|---|
| -m apt -a "name=tree state=present" | apt: then name: tree, state: present |
| -m copy -a "content=... dest=..." | copy: then content:, dest: |
| -m user -a "name=webadmin ..." | user: then name: webadmin |

Same modules. Same arguments. Only the format is different. The playbook is saved in a file.

### Step 6: Run the playbook again

```bash
ansible-playbook web-setup.yml
```

Everything shows ok and changed=0. The job is already done, so Ansible does nothing. Both ad-hoc modules and playbooks are idempotent, but only the playbook is easy to run again.

---

## Concept: Which one to use

| | Ad-hoc command | Playbook |
|---|---|---|
| Command | ansible | ansible-playbook |
| Size | One task | Many tasks and plays |
| Saved | No, only in terminal history | Yes, a .yml file |
| Version control | Not possible | Stored in Git |
| Repeatable | You must retype it | Run the same file again |
| Review by team | Not possible | Review the file before running |
| Best for | Quick checks, one-time fixes, troubleshooting | Server setup, deployments, anything repeated |

Simple rule: if you will do it more than once, or someone else needs to know how it was done, write a playbook.

Real-world examples:

| Situation | Use |
|---|---|
| Check disk space on 50 servers right now | Ad-hoc |
| Restart nginx on all web servers during an incident | Ad-hoc |
| Set up a new web server with 20 steps | Playbook |
| Deploy a new version of the app every week | Playbook |

---

## Clean up

```bash
ansible all -m user -a "name=deploy state=absent remove=yes" -b
ansible MyWebServer -m user -a "name=webadmin state=absent remove=yes" -b
```

---

## Interview Questions

1. What is the difference between the ansible and ansible-playbook commands?
2. Which module does Ansible use when you do not give -m?
3. Give two situations where an ad-hoc command is the better choice.
4. Why is a playbook better than ad-hoc commands for a production server setup?
5. Why do we put quotes around 'MyWebServer:MyDBServer'?

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- LinkedIn: https://www.linkedin.com/in/chaubeyrahul/
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
