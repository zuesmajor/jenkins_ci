# Automation EC2 Testing for Ansible

_**What it's for**_
This automation framework is for an operator to fork and build their own playbooks to run against an EC2 instance for testing. Once you push your changes to github a build will automatically trigger and run your Ansible Plays against the EC2 instance.

_**How to use**_

The part that you want to pay attention to is within `ci_build.yml`:
```
########################
# Main Configuration Steps For New Instance

- name: Configuring new EC2 for Tests
  hosts: ec2-hosts
  user: ec2-user
  gather_facts: false
  tasks:

   - block:
       - name: Pause To bypass Connection Refused
         pause:
           seconds: 25

       - include: configure.yml

     rescue:
     # This is for access variables across hosts.
     # Sets a failure flag for when operators plays fail
       - debug: msg='Hit Rescue'

       - name: set Failure Fact
         command: echo "Failed"
         register: failed
```

This section of code is where you want to `- include` your playbooks that will be ran. You'll want to write your own plays in a separate file then include them within this section of code to make things easier to read and debug.

You want to `- include` your playbooks within the `- block` section of the code. This acts as a `try/catch` in which for 2 reasons:
1. When your playbook fails, it'll delete the EC2 instance. You can view the output on Jenkins
2. If it passes, it'll keep the instance or you can pass `-e delete_instance=true` into the command line to delete upon playbook passing.

```
tasks:

   - block:
       - name: Pause To bypass Connection Refused
         pause:
           seconds: 25

       - include: configure.yml

```
