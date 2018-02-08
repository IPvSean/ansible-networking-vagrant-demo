# Exercise 04 - Configuring IP Addresses

For this exercise we are going to configure IP address between the leaf and spines by using the [vyos_l3_interface module](http://docs.ansible.com/ansible/latest/vyos_l3_interface_module.html) and the [aggregate resources feature](https://www.ansible.com/blog/accelerate-ansible-networking-aggregate-resources).  Here is the IP address schema we are using for this lab:

Here is the IP address diagram:
![diagram](ipaddress_diagram.png)

Device Left | Left IP | Right IP | Device Right
------------ | ------------- | ------------- | -------------
spine01 eth2 | 192.168.11.1/30 | 192.168.11.2/30 | eth6 leaf01
spine01 eth3 | 192.168.12.1/30 | 192.168.12.2/30 | eth6 leaf02
spine02 eth2 | 192.168.21.1/30 | 192.168.21.2/30 | eth7 leaf01
spine02 eth3 | 192.168.22.1/30 | 192.168.22.2/30 | eth7 leaf02


Look at the following playbook:

```yml
---
- hosts: network
  connection: network_cli
  vars:
    interface_data:
      spine01:
          - { name: eth2, ipv4: 192.168.11.1/30 }
          - { name: eth3, ipv4: 192.168.12.1/30 }
      spine02:
          - { name: eth2, ipv4: 192.168.21.1/30 }
          - { name: eth3, ipv4: 192.168.22.1/30 }
      leaf01:
          - { name: eth6, ipv4: 192.168.11.2/30 }
          - { name: eth7, ipv4: 192.168.21.2/30 }
      leaf02:
          - { name: eth6, ipv4: 192.168.12.2/30 }
          - { name: eth7, ipv4: 192.168.22.2/30 }
  tasks:
    - name: Set IP addresses on aggregate
      vyos_l3_interface:
        aggregate: "{{interface_data[inventory_hostname]}}"
```

To run the playbook use the `ansible-playbook` command.  The default password is vagrant for the vyos vagrant image.

```bash
ansible-playbook system.yml -u vagrant -k
```
Parameter | Explanation
------------ | -------------
ansible-playbook | Ansible executable for running playbooks
system.yml | the name of the playbook
-u vagrant | specifies user vagrant
-k | prompts us for password

# Looking at the results

Login to a device:
```
ssh vagrant@leaf01
```

To see what Ansible configured:



## Complete
You have completed exercise 04.

[Return to training-course](../README.md)
