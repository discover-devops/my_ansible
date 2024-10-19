**Ansible facts** are pieces of information that Ansible automatically collects about managed nodes (target machines) when a playbook or command is executed. These facts provide details about the system's configuration, such as operating system type, IP addresses, CPU architecture, memory, and more. Facts are stored as variables that you can reference in your playbooks to make them more dynamic and adaptable.

### **How Ansible Gathers Facts**
Ansible collects facts by running the **`setup`** module on managed nodes at the beginning of each playbook run (unless fact gathering is explicitly disabled). These facts are collected from the system's hardware, operating system, network configuration, and services.

### **Types of Facts Collected by Ansible**

Ansible facts can be grouped into categories, including:

1. **Operating System Information**:
   - `ansible_os_family`: The family of the operating system (e.g., `RedHat`, `Debian`, `Windows`).
   - `ansible_distribution`: The name of the OS distribution (e.g., `Ubuntu`, `CentOS`, `Fedora`).
   - `ansible_distribution_version`: The version of the operating system (e.g., `20.04` for Ubuntu).

2. **Network Information**:
   - `ansible_default_ipv4`: Default IPv4 address and other network details.
   - `ansible_all_ipv4_addresses`: A list of all IPv4 addresses on the system.
   - `ansible_fqdn`: Fully Qualified Domain Name of the system.

3. **Hardware Information**:
   - `ansible_processor`: Information about the CPU.
   - `ansible_memtotal_mb`: Total memory (RAM) in megabytes.
   - `ansible_architecture`: The system's CPU architecture (e.g., `x86_64`).

4. **File System Information**:
   - `ansible_mounts`: A list of mounted file systems and their properties.
   - `ansible_disk_size`: Information about disk size and usage.

5. **User and Host Information**:
   - `ansible_hostname`: The short hostname of the managed node.
   - `ansible_user_id`: The ID of the user running the playbook.

6. **Package and Service Information**:
   - `ansible_pkg_mgr`: The package manager used on the system (`apt` for Ubuntu/Debian, `yum` for RedHat/CentOS).
   - `ansible_services`: A list of services and their status.

### **How to View Ansible Facts**

You can view all the facts collected by Ansible from a managed node by running the following command:

```bash
ansible <host> -m setup
```

For example, to view facts about all hosts in your inventory, you can run:

```bash
ansible all -m setup
```

### **Using Facts in Playbooks**

You can use these facts in your playbooks to make them more flexible and dynamic. For example, you can conditionally execute tasks based on the value of a fact or customize tasks based on system properties.

#### Example: Using Facts in a Playbook

```yaml
---
- name: Use Ansible facts in a playbook
  hosts: all
  tasks:
    - name: Display the operating system family
      debug:
        msg: "This system is running {{ ansible_os_family }}."

    - name: Install the appropriate web server based on the OS
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"

    - name: Install the appropriate web server for RedHat-based systems
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

### **Disabling Fact Gathering**

If you do not want to collect facts for performance reasons or if facts are not needed for your playbook, you can disable fact gathering by setting `gather_facts: no` in your playbook:

```yaml
---
- name: Run without fact gathering
  hosts: all
  gather_facts: no
  tasks:
    - name: Print a message
      debug:
        msg: "Skipping fact gathering."
```

### **Custom Facts**
In addition to the default facts gathered by Ansible, you can define your own custom facts by placing scripts or static fact files in `/etc/ansible/facts.d` on the managed node. These custom facts can be written in JSON, INI, or executable formats (such as shell scripts) and will be collected along with standard facts.

---

### **Key Advantages of Ansible Facts**:
1. **Dynamic Playbooks**: Use facts to adapt playbooks to the specific properties of the managed nodes, such as installing different packages based on the operating system.
2. **Simplified Inventory Management**: Instead of hardcoding host properties, you can use facts to dynamically reference details about the host.
3. **Powerful Automation**: Facts allow for more flexible and context-aware automation.

---

Ansible facts are extremely powerful in making your playbooks adaptable to different environments. Let me know if you'd like further examples or explanations!
