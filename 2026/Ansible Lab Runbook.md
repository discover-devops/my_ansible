# Ansible Lab Runbook
## Terraform Environment, Control Node Setup, Ad-hoc Commands, Playbooks, Handlers, Loops and Variables

This runbook lists every step we did in class, in order. Follow it from top to bottom.

---

## Lab Architecture

| Server | Role | Inventory group |
|---|---|---|
| control-node | Ansible installed here. All Ansible commands run from here | none |
| slave-1 | Managed node | MyWebServer |
| slave-2 | Managed node | MyDBServer |

- 1 VPC (10.0.0.0/16) with 2 public subnets (10.0.1.0/24 and 10.0.2.0/24)
- 3 Ubuntu 24.04 EC2 instances, nothing pre-installed
- 1 key pair to connect from your laptop with PuTTY
- Security Group: SSH (22) and HTTP (80) from outside, all traffic between the lab machines

Note your own IPs from terraform output. In this runbook we use these placeholders:

| Placeholder | Meaning |
|---|---|
| CONTROL_PUBLIC_IP | control-node public IP |
| SLAVE1_PUBLIC_IP, SLAVE2_PUBLIC_IP | slave public IPs (PuTTY and browser) |
| SLAVE1_PRIVATE_IP, SLAVE2_PRIVATE_IP | slave private IPs (used by Ansible) |

---

## Prerequisites on Your Laptop

- VS Code with Git Bash terminal
- Terraform installed
- AWS CLI installed and configured (aws configure)
- PuTTY and PuTTYgen installed

---

# PART 1: Build the Environment with Terraform

## Step 1: Create the project folder

Create a folder ansible-lab-terraform and open it in VS Code. Create the files below.

### providers.tf

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
    tls = {
      source  = "hashicorp/tls"
      version = ">= 4.0"
    }
    local = {
      source  = "hashicorp/local"
      version = ">= 2.4"
    }
  }
}

provider "aws" {
  region = var.region

  default_tags {
    tags = {
      Project   = var.project
      ManagedBy = "Terraform"
    }
  }
}
```

### variables.tf

```hcl
variable "region" {
  type    = string
  default = "ap-south-1"
}

variable "project" {
  type    = string
  default = "ansible-lab"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  type    = list(string)
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "slave_count" {
  description = "Number of slave nodes. Change 2 to 3 to add one more slave."
  type        = number
  default     = 2
}

variable "ssh_allowed_cidr" {
  description = "Who can SSH into the instances. Use your own IP like 49.36.10.20/32 for better security."
  type        = string
  default     = "0.0.0.0/0"
}
```

### network.tf

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_vpc" "lab" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project}-vpc"
  }
}

resource "aws_internet_gateway" "lab" {
  vpc_id = aws_vpc.lab.id

  tags = {
    Name = "${var.project}-igw"
  }
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.lab.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project}-public-${count.index + 1}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.lab.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.lab.id
  }

  tags = {
    Name = "${var.project}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

### security.tf

```hcl
resource "aws_security_group" "lab" {
  name        = "${var.project}-sg"
  description = "SSH and HTTP from outside, all traffic between lab machines"
  vpc_id      = aws_vpc.lab.id

  ingress {
    description = "SSH from laptop"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_allowed_cidr]
  }

  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "All traffic between lab machines"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    self        = true
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project}-sg"
  }
}
```

### keys.tf

The private key is saved in your Downloads folder, never inside the project folder.

```hcl
resource "tls_private_key" "lab" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "lab" {
  key_name   = "${var.project}-key"
  public_key = tls_private_key.lab.public_key_openssh
}

resource "local_sensitive_file" "lab_pem" {
  content         = tls_private_key.lab.private_key_pem
  filename        = pathexpand("~/Downloads/${var.project}-key.pem")
  file_permission = "0400"
}
```

### ec2.tf

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "control" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public[0].id
  vpc_security_group_ids = [aws_security_group.lab.id]
  key_name               = aws_key_pair.lab.key_name

  tags = {
    Name = "${var.project}-control-node"
  }
}

resource "aws_instance" "slave" {
  count = var.slave_count

  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
  vpc_security_group_ids = [aws_security_group.lab.id]
  key_name               = aws_key_pair.lab.key_name

  tags = {
    Name = "${var.project}-slave-${count.index + 1}"
  }
}
```

### outputs.tf

```hcl
output "control_public_ip" {
  value = aws_instance.control.public_ip
}

output "control_private_ip" {
  value = aws_instance.control.private_ip
}

output "slave_public_ips" {
  value = aws_instance.slave[*].public_ip
}

output "slave_private_ips" {
  value = aws_instance.slave[*].private_ip
}

output "pem_file" {
  value = local_sensitive_file.lab_pem.filename
}

output "login_user" {
  value = "ubuntu"
}
```

### .gitignore

```text
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl
*.pem
*.ppk
crash.log
```

## Step 2: Create the infrastructure

In the VS Code Git Bash terminal:

```bash
terraform init
terraform plan
terraform apply -auto-approve
terraform output
```

- terraform init: downloads the aws, tls and local providers
- terraform plan: shows what will be created
- terraform apply: creates the VPC, subnets, security group, key pair and 3 EC2 instances
- terraform output: shows the public and private IPs and the pem file path

Verify the key file:

```bash
ls -l ~/Downloads/ansible-lab-key.pem
ssh-keygen -l -f ~/Downloads/ansible-lab-key.pem
```

The fingerprint must start with 4096. That is the key Terraform created.

## Step 3: Convert the pem key to ppk for PuTTY

PuTTY needs a .ppk file.

1. Open PuTTYgen and click Load
2. Change the file type to All Files, select Downloads/ansible-lab-key.pem
3. Confirm the Key fingerprint shows ssh-rsa 4096 and matches the ssh-keygen output
4. Click Save private key, click Yes (no passphrase), save as Downloads/ansible-lab-key.ppk

Note: if your Downloads folder has older keys from other labs, make sure you load ansible-lab-key.pem. Loading the wrong key gives the error "Server refused our key" in PuTTY.

## Step 4: Connect to all 3 servers with PuTTY

For each server:

1. Session, Host Name: ubuntu@PUBLIC_IP
2. Connection, SSH, Auth, Credentials, Private key file: Downloads/ansible-lab-key.ppk
3. Session, Saved Sessions: control-node, slave-1 or slave-2, click Save, click Open

The same .ppk works for all 3 servers. Only the IP changes.

Git Bash alternative:

```bash
ssh -i ~/Downloads/ansible-lab-key.pem ubuntu@CONTROL_PUBLIC_IP
```

Set a hostname in each window so you always know where you are:

```bash
sudo hostnamectl set-hostname control-node
exec bash
```

Use slave-1 and slave-2 in the other two windows.

---

# PART 2: Install Ansible and Connect Control Node to Slaves

## Step 5: Install Ansible on the control-node only

Run on control-node:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
ansible --version
```

Ansible is installed only on the control-node. The slaves need nothing extra because Ansible is agentless. It connects over SSH and uses the Python already on Ubuntu.

## Step 6: Generate an SSH key on the control-node

Run on control-node:

```bash
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
ls -l ~/.ssh
cat ~/.ssh/id_rsa.pub
```

| File | Purpose |
|---|---|
| ~/.ssh/id_rsa | Private key. Stays on the control-node |
| ~/.ssh/id_rsa.pub | Public key. Copy this to every slave |
| ~/.ssh/authorized_keys | Keys allowed to log in to this machine |
| ~/.ssh/known_hosts | Servers this machine has already connected to |

Two different keys are used in this lab:

| Key | Lives on | Used for |
|---|---|---|
| ansible-lab-key.ppk | Your laptop | You logging in to all servers with PuTTY |
| id_rsa | control-node | control-node logging in to the slaves |

Copy the full output of cat ~/.ssh/id_rsa.pub. It is one single line starting with ssh-rsa and ending with ubuntu@ followed by the hostname.

## Step 7: Add the control-node public key to each slave

Run on slave-1 AND on slave-2 (paste your copied line inside the quotes):

```bash
echo "PASTE_ID_RSA_PUB_LINE_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
wc -l ~/.ssh/authorized_keys
```

- wc -l must show 2: the laptop key from AWS and the control-node key
- Use >> (append). Using > overwrites the file, deletes the laptop key and locks you out of PuTTY
- The pasted key must stay on one single line

## Step 8: Test SSH from control-node to the slaves

Run on control-node:

```bash
ssh ubuntu@SLAVE1_PRIVATE_IP hostname
ssh ubuntu@SLAVE2_PRIVATE_IP hostname
```

Type yes the first time. Each command must print the slave hostname without asking for a password.

Troubleshooting: Permission denied (publickey) means the slave does not have the control-node public key. Repeat Step 7 on that slave and check wc -l shows 2.

## Step 9: Create the inventory

Run on control-node:

```bash
mkdir -p ~/ansible && cd ~/ansible
cat > inventory.ini <<EOF
[MyWebServer]
slave-1 ansible_host=SLAVE1_PRIVATE_IP

[MyDBServer]
slave-2 ansible_host=SLAVE2_PRIVATE_IP
EOF
cat inventory.ini
```

Ansible also creates a built-in group called all, which contains every host.

## Step 10: Test the connection with Ansible ping

```bash
ansible all -i inventory.ini -m ping
ansible all -i inventory.ini --list-hosts
ansible MyWebServer -i inventory.ini --list-hosts
ansible MyDBServer -i inventory.ini --list-hosts
```

Both slaves must return "ping": "pong" in green. The Ansible ping module logs in over SSH and runs Python on the slave. It is not a network ping.

---

# PART 3: Ad-hoc Commands

An ad-hoc command is a single Ansible command run directly from the terminal for a quick, one-time task. No playbook file is needed.

Format:

```text
ansible <host pattern> -i <inventory> -m <module> -a "<arguments>" -b
```

| Part | Meaning |
|---|---|
| ansible | The ad-hoc command tool |
| host pattern | all, a group name, or a host name |
| -i inventory.ini | Inventory file to use |
| -m | Module to use |
| -a | Arguments for the module |
| -b | Become root (sudo). Needed for admin tasks |

Host patterns:

| Pattern | Runs on |
|---|---|
| all | slave-1 and slave-2 |
| slave-1 | slave-1 only |
| MyWebServer | slave-1 only |
| MyDBServer | slave-2 only |
| 'MyWebServer:MyDBServer' | both groups |

## Step 11: Create a user on all slaves

```bash
ansible all -i inventory.ini -m user -a "name=devops shell=/bin/bash create_home=yes" -b
ansible all -i inventory.ini -a "id devops"
```

Run the first command again. The output changes from changed (yellow) to ok (green). The user module checks the current state first and only acts if something must change. This is called idempotency.

## Step 12: Target a single host or group

```bash
ansible slave-1 -i inventory.ini -m user -a "name=webadmin shell=/bin/bash create_home=yes" -b
ansible MyDBServer -i inventory.ini -m user -a "name=dbadmin shell=/bin/bash create_home=yes" -b
ansible all -i inventory.ini -a "id webadmin"
ansible all -i inventory.ini -a "id dbadmin"
```

webadmin exists only on slave-1 and dbadmin only on slave-2. The failed id on the other server is expected and proves the targeting worked.

## Step 13: More ad-hoc examples

```bash
ansible all -i inventory.ini -a "uptime"
ansible all -i inventory.ini -m apt -a "name=tree state=present update_cache=yes" -b
ansible MyWebServer -i inventory.ini -m copy -a "content='Hello from Ansible' dest=/tmp/hello.txt"
```

If no -m is given, Ansible uses the command module. The command module always reports changed, because Ansible cannot know what an arbitrary command did.

Remove a user:

```bash
ansible all -i inventory.ini -m user -a "name=devops state=absent remove=yes" -b
```

---

# PART 4: Playbooks

A playbook is a YAML file that stores many tasks in order, so the setup is saved, repeatable and can be kept in Git.

Structure:

| Level | Meaning | Example |
|---|---|---|
| Playbook | The YAML file | site.yml |
| Play | Which hosts to work on | hosts: MyWebServer |
| Task | One step | Install nginx |
| Module | The tool that does the step | apt, copy, service, user |

YAML rules: indentation matters, use spaces only, never tabs. Lines starting with - are list items.

## Step 14: First playbook

Run on control-node:

```bash
cd ~/ansible
cat > site.yml <<'EOF'
---
- name: Configure Web Server
  hosts: MyWebServer
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create home page
      copy:
        content: "Hello from Ansible Lab - Web Server\n"
        dest: /var/www/html/index.html

    - name: Start nginx and enable on boot
      service:
        name: nginx
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

    - name: Install MySQL server
      apt:
        name: mysql-server
        state: present
        update_cache: yes
EOF
```

Check, dry run, then run:

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
ansible-playbook -i inventory.ini site.yml --check
ansible-playbook -i inventory.ini site.yml
curl http://SLAVE1_PRIVATE_IP
```

- --syntax-check: finds YAML and indentation errors
- --check: dry run, shows what would change without changing anything
- PLAY RECAP must show failed=0
- Run the playbook a second time: all tasks show ok and changed=0

Ad-hoc vs playbook:

| | Ad-hoc | Playbook |
|---|---|---|
| Command | ansible | ansible-playbook |
| Size | One task | Many tasks and plays |
| Saved | No | Yes, a .yml file |
| Use for | Quick checks and fixes | Repeatable setup |

---

# PART 5: Handlers, Loops and Variables

## Step 15: Prepare the lab folder

Run on control-node:

```bash
mkdir -p ~/ansible-labs && cd ~/ansible-labs
cp ~/ansible/inventory.ini .
printf "[defaults]\ninventory = inventory.ini\ninterpreter_python = auto_silent\n" > ansible.cfg
ansible all -m ping
```

- ansible.cfg in the current folder sets the inventory, so -i is no longer needed
- interpreter_python = auto_silent hides the Python interpreter warning

## Step 16: Remove nginx from slave-1

nginx from Step 14 is listening on port 80 on slave-1. Apache cannot start while port 80 is in use.

```bash
ansible all -m apt -a "update_cache=yes" -b
ansible slave-1 -m apt -a "name=nginx state=absent purge=yes" -b
ansible slave-1 -a "ss -tlnp" -b
```

The last command must not show anything listening on port 80.

If you skip this step, the Apache restart fails on slave-1 with:

```text
(98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:80
```

To find which process holds a port, use ansible slave-1 -a "ss -tlnp" -b.

---

## LAB 1: Website Deployment Using the Copy Module

Goal: install Apache on both slaves and copy an HTML file from the control-node.

```bash
cd ~/ansible-labs
cat > index.html <<'EOF'
<h1>Hello world</h1>
<p>This is my Ansible website</p>
EOF
```

```bash
cat > playbook.yaml <<'EOF'
---
- name: Deploy Apache application
  hosts: all
  become: true

  tasks:
    - name: Install Apache
      package:
        name: apache2
        state: present

    - name: Restart Apache
      service:
        name: apache2
        state: restarted

    - name: Copy index.html from Control Node to Managed Nodes
      copy:
        src: /home/ubuntu/ansible-labs/index.html
        dest: /var/www/html/index.html
EOF
ansible-playbook playbook.yaml --syntax-check
ansible-playbook playbook.yaml
```

| Module | What it does |
|---|---|
| package | Installs a package. Works on Ubuntu (apt) and RedHat (yum or dnf) |
| service | Starts, stops or restarts a service |
| copy | src is on the control-node, dest is on the managed node |

Verify:

```bash
curl http://SLAVE1_PRIVATE_IP
curl http://SLAVE2_PRIVATE_IP
```

Browser: http://SLAVE1_PUBLIC_IP and http://SLAVE2_PUBLIC_IP

Run the playbook again and compare:

| Task | First run | Second run |
|---|---|---|
| Install Apache | changed | ok |
| Restart Apache | changed | changed |
| Copy index.html | changed | ok |

Restart Apache always shows changed because state: restarted restarts the service on every run.

---

## LAB 2: Handlers

Goal: restart Apache only when index.html changes.

A handler is a task that runs only when another task notifies it and that task reports changed.

```bash
cat > playbook.yaml <<'EOF'
---
- name: Deploy Apache application
  hosts: all
  become: true

  tasks:
    - name: Install Apache
      package:
        name: apache2
        state: present

    - name: Copy index.html from Control Node to Managed Nodes
      copy:
        src: /home/ubuntu/ansible-labs/index.html
        dest: /var/www/html/index.html
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
EOF
ansible-playbook playbook.yaml
```

Result: Install Apache ok, Copy index.html ok, no RUNNING HANDLER line. The file did not change, so the handler was not notified.

Change the page and run again:

```bash
echo "<p>Hello from Ansible Handler</p>" >> index.html
ansible-playbook playbook.yaml
curl http://SLAVE1_PRIVATE_IP
```

Result: Copy index.html changed, RUNNING HANDLER [Restart Apache] changed.

Handler rules:

- The notify value must match the handler name exactly
- Handlers run at the end of the play, after all tasks
- A handler runs only once, even if several tasks notify it

---

## LAB 3: Loops

Goal: install several packages and create several users with one task each.

```bash
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
        create_home: yes
        shell: /bin/bash
      loop:
        - rohit
        - virat
        - ishan
        - abhishek
EOF
ansible-playbook playbook-loops.yaml
ansible all -a "id rohit"
```

- loop: the list of values
- item: the current value in each iteration
- "{{ item }}": inserts the current value. Quotes are required when a value starts with {{
- Linux usernames must be lowercase. Ubuntu rejects capital letters by default

---

## LAB 4: Variables Inside the Playbook

Goal: define values once in vars and reuse them.

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
ansible-playbook playbook-vars.yaml
```

| Variable | Value |
|---|---|
| pkg_name | apache2 |
| pkg_state | present |
| file_source | /home/ubuntu/ansible-labs/index.html (control-node) |
| file_destination | /var/www/html/index.html (managed node) |

Change a value in one place:

```bash
sed -i 's#/var/www/html/index.html#/var/www/html/about.html#' playbook-vars.yaml
ansible-playbook playbook-vars.yaml
curl http://SLAVE1_PRIVATE_IP/about.html
```

Override a variable from the command line with -e (extra vars). Extra vars take priority over values in the playbook:

```bash
ansible-playbook playbook-vars.yaml -e "file_destination=/var/www/html/contact.html"
curl http://SLAVE1_PRIVATE_IP/contact.html
```

Note: changing pkg_name to nginx would make nginx and Apache compete for port 80. Do not run that change on these servers.

---

## LAB 5 Part A: Variables From a Separate File

Goal: keep variables in vars.yaml and load them with vars_files.

```bash
cat > vars.yaml <<'EOF'
pkg_name: apache2
pkg_state: present
file_source: /home/ubuntu/ansible-labs/index.html
file_destination: /var/www/html/index.html
pkg_start: apache2
EOF
```

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
ansible-playbook playbook-external-vars.yaml
```

Change only the vars file to install Docker:

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

The package is docker.io but the service is docker. That is why package name and service name use separate variables (pkg_name and pkg_start). The playbook was not changed, only vars.yaml.

Put Apache back:

```bash
cat > vars.yaml <<'EOF'
pkg_name: apache2
pkg_state: present
file_source: /home/ubuntu/ansible-labs/index.html
file_destination: /var/www/html/index.html
pkg_start: apache2
EOF
```

---

## LAB 5 Part B: Variables as a Prompt

Goal: enter variable values when the playbook starts.

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
ansible-playbook playbook-prompt.yaml
```

Enter:

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

| Keyword | Meaning |
|---|---|
| vars_prompt | List of questions asked when the playbook starts |
| name | Variable that stores the answer |
| prompt | Question text shown on screen |
| private: no | Input is visible. Use private: yes for passwords |

Prompted variables are for manual, one-time runs. Pipelines use -e instead because nobody is there to type answers.

---

# Adding One More Slave

In variables.tf change slave_count from 2 to 3, then on your laptop:

```bash
terraform apply -auto-approve
terraform output slave_private_ips
```

Then repeat Step 7 on slave-3, add slave-3 to inventory.ini and run ansible all -m ping.

---

# Troubleshooting Summary

| Problem | Cause | Fix |
|---|---|---|
| PuTTY: Server refused our key | .ppk made from the wrong or an old .pem | Load ansible-lab-key.pem in PuTTYgen, confirm 4096 fingerprint, save a new .ppk |
| SSH from control-node: Permission denied (publickey) | Slave does not have the control-node public key | Repeat Step 7, wc -l must show 2 |
| cat known_hosts: No such file | Wrong path | Use ~/.ssh/known_hosts |
| Apache fails: Address already in use | nginx holds port 80 | Remove nginx (Step 16) |
| Browser times out | Port 80 not open in Security Group | Check the HTTP rule in security.tf |
| Python interpreter WARNING | Ansible auto-detected Python | Harmless. Add interpreter_python = auto_silent to ansible.cfg |
| No hosts matched | Group name typo | Group names are case-sensitive |

---

# Clean Up

On your laptop:

```bash
terraform destroy -auto-approve
```

---

## Keep learning with me

- YouTube: https://www.youtube.com/@BuildAutomateArchitect
- System Design playlist: https://www.youtube.com/playlist?list=PLeAHTLMwFYQ0
- Telegram: https://t.me/BuildAutomateArchitect
- Website: https://rahulchaubeyconsulting.com/

Rahul Chaubey
