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
          - { name: lo, ipv4: 10.0.0.1/32 }
          - { name: eth2, ipv4: 192.168.11.1/30 }
          - { name: eth3, ipv4: 192.168.12.1/30 }
      spine02:
          - { name: lo, ipv4: 10.0.0.2/32 }
          - { name: eth2, ipv4: 192.168.21.1/30 }
          - { name: eth3, ipv4: 192.168.22.1/30 }
      leaf01:
          - { name: lo, ipv4: 10.0.0.11/32 }
          - { name: eth6, ipv4: 192.168.11.2/30 }
          - { name: eth7, ipv4: 192.168.21.2/30 }
      leaf02:
          - { name: lo, ipv4: 10.0.0.12/32 }
          - { name: eth6, ipv4: 192.168.12.2/30 }
          - { name: eth7, ipv4: 192.168.22.2/30 }
  tasks:
    - name: Set IP addresses on aggregate
      vyos_l3_interface:
        aggregate: "{{interface_data[inventory_hostname]}}"
```

To run the playbook use the `ansible-playbook` command.  The default password is vagrant for the vyos vagrant image.

```bash
ansible-playbook ipaddr.yml -u vagrant -k
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
```
vagrant@leaf01:~$ show int
Codes: S - State, L - Link, u - Up, D - Down, A - Admin Down
Interface        IP Address                        S/L  Description
---------        ----------                        ---  -----------
eth0             10.0.2.15/24                      u/u
eth1             172.16.10.11/24                   u/u
eth2             -                                 u/u
eth3             -                                 u/u
eth4             -                                 u/u
eth5             -                                 u/u
eth6             192.168.11.2/30                   u/u
eth7             192.168.21.2/30                   u/u
lo               127.0.0.1/8                       u/u
                 ::1/128
```
We can make sure both sides are configured by issuing an ICMP ping
```
vagrant@leaf01:~$ ping 192.168.11.1
PING 192.168.11.1 (192.168.11.1) 56(84) bytes of data.
64 bytes from 192.168.11.1: icmp_req=1 ttl=64 time=0.288 ms
64 bytes from 192.168.11.1: icmp_req=2 ttl=64 time=0.305 ms
^C
--- 192.168.11.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.288/0.296/0.305/0.019 ms
```

We can also automate checking all of the connections in a playbook:

```bash
ansible-playbook system.yml -u vagrant -k
```

To look at the playbook [click here](check.yml)


## Complete
You have completed exercise 04.

[Return to training-course](../README.md)
