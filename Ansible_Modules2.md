How to write Custome Modules:

You can write your own custom modules in Ansible to perform tasks that are specific to your environment or infrastructure, such as installing Nginx. Custom modules allow you to extend Ansible's functionality beyond what is provided by core and community modules.

To write a custom module to install Nginx, you would typically follow these steps:

1. **Choose a Programming Language**: Ansible custom modules can be written in any programming language that can output JSON. However, Python is the most common choice due to its integration with Ansible.

2. **Understand the Module Structure**: Custom modules need to follow a specific structure and adhere to Ansible's module development guidelines. The module needs to accept parameters, perform the required actions on the target hosts, and return JSON-formatted output.

3. **Write the Module Code**: You would write the code for your custom module, implementing the logic to install Nginx on the target hosts. This code would typically use subprocesses or libraries to interact with the underlying system and perform the installation.

4. **Test the Module**: It's essential to thoroughly test your custom module to ensure that it behaves as expected and handles different scenarios gracefully.

5. **Distribute the Module**: Once your custom module is ready, you can distribute it to your Ansible control node and make it available for use in your playbooks.

Here's a very basic example of what a custom module to install Nginx might look like in Python:

```python
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule
import subprocess

def install_nginx():
    result = {}
    cmd = ["apt", "install", "-y", "nginx"]
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    stdout, stderr = proc.communicate()
    result['stdout'] = stdout.decode('utf-8')
    result['stderr'] = stderr.decode('utf-8')
    result['rc'] = proc.returncode
    return result

def main():
    module = AnsibleModule(argument_spec={})
    result = install_nginx()
    if result['rc'] != 0:
        module.fail_json(msg="Failed to install Nginx", **result)
    else:
        module.exit_json(changed=True, msg="Nginx installed successfully", **result)

if __name__ == '__main__':
    main()
```

This example is quite basic and lacks error handling, but it demonstrates the general structure of a custom module. You would need to enhance it with error handling, parameter validation, and additional features as needed for your use case.

Remember to follow Ansible's module development guidelines and best practices when writing custom modules to ensure consistency and compatibility with Ansible. Additionally, consider contributing your custom modules to the Ansible community to benefit others.
