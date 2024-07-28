# Ansible Vault and Security

#### Ansible Vault

**Theory and Concept:**
Ansible Vault is a feature that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than plain text in your playbooks or roles. This ensures that sensitive information is not exposed.

**Encrypting and Decrypting Secrets:**

**Encrypting a File:**
You can encrypt an existing file or create a new encrypted file using Ansible Vault.

**Step-by-Step Lab: Encrypting a File**

1. **Encrypt an Existing File:**
    ```sh
    ansible-vault encrypt secrets.yml
    ```
    You'll be prompted to enter a password to encrypt the file.

2. **Create a New Encrypted File:**
    ```sh
    ansible-vault create encrypted_secrets.yml
    ```
    You'll be prompted to enter a password to encrypt the file, then the file will be opened in the default editor for you to add content.

3. **Edit an Encrypted File:**
    ```sh
    ansible-vault edit encrypted_secrets.yml
    ```
    You'll be prompted to enter the password to decrypt the file for editing.

4. **Decrypt an Encrypted File:**
    ```sh
    ansible-vault decrypt encrypted_secrets.yml
    ```
    You'll be prompted to enter the password to decrypt the file.

**Using Vault in Playbooks:**

**Step-by-Step Lab: Using Vault in Playbooks**

1. **Create an Encrypted Variables File:**
    ```sh
    ansible-vault create secrets.yml
    ```
    Add the following content to the `secrets.yml` file:
    ```yaml
    db_password: "SuperSecretPassword"
    ```

2. **Create a Playbook that Uses the Encrypted Variables:**
    ```sh
    nano use_vault.yml
    ```
    ```yaml
    ---
    - name: Use Ansible Vault
      hosts: all
      vars_files:
        - secrets.yml
      tasks:
        - name: Print the encrypted variable
          debug:
            msg: "Database password is {{ db_password }}"
    ```

3. **Run the Playbook with the Vault Password:**
    ```sh
    ansible-playbook -i inventory use_vault.yml --ask-vault-pass
    ```
    You'll be prompted to enter the password to decrypt the `secrets.yml` file.

#### Security Best Practices

**Secure Communications:**
Ensure all communications between Ansible and the managed nodes are encrypted. This can be achieved by using SSH for secure connections and ensuring SSL/TLS for web traffic.

**Managing Sensitive Data:**
1. **Use Ansible Vault to store sensitive data.**
2. **Restrict access to the vault password.**
3. **Rotate passwords and keys regularly.**

**Step-by-Step Lab: Secure Communications**

1. **Ensure SSH Keys Are Used for Ansible Communication:**
    Generate SSH keys if you don't have them already.
    ```sh
    ssh-keygen
    ```
    Copy the public key to the target machines.
    ```sh
    ssh-copy-id user@target_machine
    ```

2. **Use SSH Configurations for Enhanced Security:**
    Edit the SSH configuration file to enforce strong security practices.
    ```sh
    nano ~/.ssh/config
    ```
    Add the following content:
    ```plaintext
    Host target_machine
      HostName target_machine
      User user
      IdentityFile ~/.ssh/id_rsa
      StrictHostKeyChecking yes
    ```

**Managing Sensitive Data Example in DevOps:**

**Scenario:**
You need to deploy an application that requires a database password to be securely stored and retrieved during deployment.

**Steps:**
1. **Encrypt the Database Password Using Ansible Vault:**
    ```sh
    ansible-vault create db_secrets.yml
    ```
    Add the following content to the `db_secrets.yml` file:
    ```yaml
    db_password: "SuperSecretPassword"
    ```

2. **Create a Playbook to Deploy the Application Using the Encrypted Password:**
    ```sh
    nano deploy_application.yml
    ```
    ```yaml
    ---
    - name: Deploy Application
      hosts: app_servers
      vars_files:
        - db_secrets.yml
      tasks:
        - name: Install application dependencies
          apt:
            name: "{{ item }}"
            state: present
          loop:
            - python3-pip
            - libpq-dev

        - name: Set up database configuration
          template:
            src: templates/db_config.j2
            dest: /etc/app/db_config.ini

        - name: Start application
          service:
            name: app_service
            state: started
    ```

3. **Create the Template for Database Configuration:**
    ```sh
    mkdir templates
    nano templates/db_config.j2
    ```
    ```jinja
    [database]
    user = app_user
    password = {{ db_password }}
    ```

4. **Run the Playbook with the Vault Password:**
    ```sh
    ansible-playbook -i inventory deploy_application.yml --ask-vault-pass
    ```
    You'll be prompted to enter the password to decrypt the `db_secrets.yml` file.

By using Ansible Vault and following security best practices, you can ensure that sensitive information is protected and that communications between Ansible and managed nodes are secure. This helps maintain the integrity and confidentiality of your data in a DevOps environment.
