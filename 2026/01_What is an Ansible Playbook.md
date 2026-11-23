# 01 What is an Ansible Playbook and Why Do We Need It

## The real problem first

Imagine your manager says: "We have 2 new web servers. Install Apache, put our home page on them, and make sure Apache starts on boot."

With ad-hoc commands you can do it like this:

```bash
ansible MyWebServer -m apt -a "name=apache2 state=present update_cache=yes" -b
ansible MyWebServer -m copy -a "content='Welcome to our site' dest=/var/www/html/index.html" -b
ansible MyWebServer -m service -a "name=apache2 state=started enabled=yes" -b
```

It works. Now think about these questions before reading further:

- Next month 5 more servers arrive. Will you remember all 3 commands, in the same order, with the same options?
- Your teammate wants to know exactly how the servers were set up. Where will they look?
- The real setup is not 3 steps, it is 30 steps. Will you type 30 commands every time?

The commands lived only in your terminal history. That is the problem.

---

## Concept: What is a playbook

A playbook is a YAML file where you write down all the steps once, in order, and run them with one command.

Think of it like a recipe:

- Ad-hoc command = you cook from memory, one step at a time
- Playbook = the recipe is written down. Anyone can follow it, and the result is the same every time

Because a playbook is a file, you can:

- Save it in Git
- Review it with your team before running it
- Run it again on 1 server or 100 servers
- See the full history of what changed and who changed it

This is called Infrastructure as Code. The server setup itself becomes code.

---

## Concept: The structure of a playbook

```text
Playbook          one .yml file
  Play 1          WHERE to run (which group of hosts)
    Task 1        WHAT to do (one step)
    Task 2
  Play 2          another group of hosts
    Task 1
```

| Level | Answers | Example |
|---|---|---|
| Playbook | The whole file | site.yml |
| Play | Where? | hosts: MyWebServer |
| Task | What? | Install Apache |
| Module | How? | apt, copy, service, user |

YAML rules you must follow:

- Indentation matters. Use spaces only, never tabs
- A line starting with a dash ( - ) is one item in a list
- key: value pairs need a space after the colon

---

## Prerequisites

- Control node with Ansible installed
- Passwordless SSH from control node to slave-1 and slave-2
- Lab folder with inventory and ansible.cfg

```bash
mkdir -p ~/ansible-labs && cd ~/ansible-labs
cat > inventory.ini <<EOF
[MyWebServer]
slave-1 ansible_host=SLAVE1_PRIVATE_IP

[MyDBServer]
slave-2 ansible_host=SLAVE2_PRIVATE_IP
EOF
printf "[defaults]\ninventory = inventory.ini\ninterpreter_python = auto_silent\n" > ansible.cfg
ansible all -m ping
```

Replace SLAVE1_PRIVATE_IP and SLAVE2_PRIVATE_IP with your own values. Both slaves must reply with pong.

---

## Step 1: Write your first playbook

```bash
cd ~/ansible-labs
cat > first-playbook.yml <<'EOF'
---
- name: Configure Web Server
  hosts: MyWebServer
  become: yes
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Create home page
      copy:
        content: "Welcome to our site\n"
        dest: /var/www/html/index.html

    - name: Start Apache and enable on boot
      service:
        name: apache2
        state: started
        enabled: yes

- name: Configure DB Server
  hosts: MyDBServer
  become: yes
  tasks:
    - name: Create dbadmin user
      user:
        name: dbadmin
        shell: /bin/bash
        create_home: yes
EOF
```

Read the file line by line:

| Line | Meaning |
|---|---|
| --- | Start of a YAML document |
| - name: Configure Web Server | Start of Play 1 with a readable name |
| hosts: MyWebServer | Play 1 runs only on the web group (slave-1) |
| become: yes | Run tasks as root, same as -b in ad-hoc |
| tasks: | The list of steps begins |
| - name: Install Apache | One task. This name appears in the output |
| apt: | The module that does the work |
| name: apache2, state: present | Module arguments, same as -a in ad-hoc |
| - name: Configure DB Server | Play 2, a different group of hosts |

Notice: the 3 ad-hoc commands from the top of this page are now 3 tasks in one file.

---

## Step 2: Check, dry run, then run

```bash
ansible-playbook first-playbook.yml --syntax-check
ansible-playbook first-playbook.yml --check
ansible-playbook first-playbook.yml
```

- --syntax-check: finds YAML and indentation mistakes before anything runs
- --check: dry run. Shows what WOULD change, changes nothing
- Last command: runs it for real

---

## Step 3: Verify

```bash
curl http://SLAVE1_PRIVATE_IP
ansible MyDBServer -a "id dbadmin"
```

- curl shows Welcome to our site from slave-1
- id shows the dbadmin user on slave-2

---

## Step 4: Run it again and watch the aha moment

```bash
ansible-playbook first-playbook.yml
```

Before you run it, guess: will Ansible install Apache again?

Look at the PLAY RECAP:

| Run | What you see |
|---|---|
| First run | changed (yellow) for most tasks |
| Second run | ok (green) for every task, changed=0 |

Concept: Ansible checks the current state first and only acts when something is different. Apache is already installed, the page is already there, so nothing is done. This is called idempotency. You can safely run the same playbook again and again.

---

## Common mistakes

| Mistake | What happens |
|---|---|
| Tab instead of spaces | Syntax error |
| apt: not aligned under - name: | Syntax error or wrong task |
| Missing become: yes | Permission denied when installing packages |
| Wrong group name in hosts: | Skipping: no hosts matched. Group names are case-sensitive |

---

## Interview Questions

1. What is the difference between a play and a task?
2. Why is a playbook called Infrastructure as Code?
3. What does --check do, and when would you use it?
4. What is idempotency? How do you prove it with a playbook?
5. Why should a playbook be stored in Git?

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- LinkedIn: https://www.linkedin.com/in/chaubeyrahul/
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
