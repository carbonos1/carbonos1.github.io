
## Weekend Side-Project: Ansible on Mikrotik routers

### Intro

Maintaining  a large amount of Mikrotik routers / switches is not unfamiliar to me.  Because logging into a large  amount of switches at a time is tedious, network automation is often a subject of discussion online and  in the workplace, and so adding these technologies to my skillset has been something of interest.

Whilst there are many of technologies that can offer these, I decided to try my hand at ansible for network configuration / deployment, training myself on Mikrotik Devices first as a point of familiarity.

**Note:** This post is simply me learning how to use ansible, and noting down my findings (as well as some basic building blocks for future inventories / scripts). If you wish to learn about this in a technical fashion, I would recommend Ansibles [Getting Started](https://docs.ansible.com/ansible/latest/getting_started/index.html) and [Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
[This Page](https://docs.ansible.com/ansible/latest/collections/community/routeros/api_module.html) in the ansible documentation, and for playbook / deployment examples I  would recommend [This Blog Post](https://yetiops.net/posts/ansible-for-networking-part-6-mikrotik-routeros/#why-routeros) on YetiOps.net , which provides some VERY detailed playbooks which can be used to see how playbooks and  inventories can be written

### Why Ansible?

Ansible is a modern, well supported Open Source Network Automation tool maintained by Red Hat written in Python. The tool is primarily used for server / application deployment, but it is more than capable of network automation, and has also found a home in this space as well. As such, Ansible supports modules for bigger players in the networking space (Cisco, Juniper,Arista... etc ) and also supports community modules as well, in this case, Mikrotik is one of those community supported modules!

For me specifically, when comparing ansible to other networking automation tools, Ansible was an attractive choice due to:

1. **Free / Open Source:** I can install this at home and get familiar with it before using it in a professional Environment
2. **Familiarity with Languages / Syntax used:** Python and YAML are my go to scripting language / variable storing languages, so I am more than happy to re-use these in this environment. This is a huge pull as well, and will influence my boilerplate code below as well.
3. **Modular:** Need to connect to an unsupported switch? You can build a module for that! Once of the other benefits of open source (and community support) is the ability to download community and add them if someone has run into the same issue. However, you can also build your own module if what you are trying to do is niche or unsupported, meaning with enough time and / or effort, you can connect to anything!
4. **Community Support / Free online Documentation:** Ansible is very welcomed in the DevOps / Server AND Networking space, and has spawned a HUGE amount of Documentation for every role it is capable of doing. This will be useful in the future.

### Installing in OCT  2023 (Debian / pip)

The Current recommended way to install ansible is either with pip (the included package manager with most installs of Python) OR with pipx. In my instance, I installed ansible with pip with the following command:

```bash
python3 -m pip install --user ansible
```

Once ansible was installed, I also needed to add the paramiko and ansible-pylibssh libraries:

```bash
pip install ansible-pylibssh paramiko
```

Then  with those installed, I then installed the  community.routeros library:

```bash
ansible-galaxy collection install community.routeros
```

Once this was installed, I had everything. So I went straight to testing my configs!


### Using Playbooks and Inventories

Ansible uses **two** files to enable network automation, these are:

1. **The Inventory:** This contains WHAT devices you want to configure, as well as the key variables (such as IP addresses, host ids, interfaces... etc) that will change between each network device.
2. **The Playbook:** this contains the HOW each device is to be configured, as well the requirements for each "action" (called a play) to happen on the device. 

####  Inventory Example:
```yaml
test:
    hosts:
        192.168.40.21:
          ansible_password: 'test_password' # host independent variable
        192.168.40.22:
        192.168.40.23:
        192.168.40.24:

mikrotik1:
    hosts:
      192.168.50.53:
    vars: # Variables for only the group mikrotik1
      ansible_connection: network_cli
      ansible_network_os: routeros
      ansible_user: 'admin' # Group Var
      ansible_password: 'password1234'
```
For more examples / a better explanation of the hosts file, see the Ansible docs page on it [here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html).

#### Playbook Example

The Playbook has a few more moving elements than the inventory, it is where we outline  the numerous plays / actions are part of our playbook. 

```yaml
---
- name: RouterOS test with network_cli connection
  hosts: mikrotik1
  gather_facts: false
  tasks:
  
  - name: Gather system resources
    community.routeros.command:
      commands:
        - /system resource print
    register: system_resource_print

  - name: Show system resources
    debug:
      var: system_resource_print.stdout_lines
      
  - name: Gather facts
    community.routeros.facts:
      gather_subset: all

  - name: Show a fact
    debug:
      msg: "First IP address: {{ ansible_net_all_ipv4_addresses[0] }}"
```

The Above Config file shows a number of actions that are to be performed.

Our first play runs the command "/system resource print" and saves it to a variable

Our second play prints the first plays result to the screen

Our third play uses the gather_facts command to gather as many variables about the switch as it can, the gather_subset tells it to grab as much as it can.

Our Fourth play grabs an IP address learnt about in our first play, and prints it to the screen as part of a message.

For more  documentation on building RouterOS Configs using SSH, see [This Guide](https://docs.ansible.com/ansible/latest/collections/community/routeros/docsite/ssh-guide.html) which is part of the ansible documentation.

#### Example output:
```
PLAY [RouterOS test with network_cli connection] ****************************************************************************************

TASK [Gather system resources] ****************************************************************************************
changed: [192.168.50.53]

TASK [Show system resources] ****************************************************************************************
ok: [192.168.50.53] => {
    "system_resource_print.stdout_lines": [
        [
            "uptime: 1d1h46m50s",
            "                  version: 7.11.2 (stable)",
            "               build-time: Aug/31/2023 13:55:47",
            "         factory-software: ",
            "              free-memory: 953.0MiB",
            "             total-memory: 1024.0MiB",
            "                      cpu: ARM",
            "                cpu-count: 2",
            "                 cpu-load: 0%",
            "           free-hdd-space: 1696.0KiB",
            "          total-hdd-space: 16.0MiB",
            "  write-sect-since-reboot: 436",
            "         write-sect-total: 7974",
            "        architecture-name: arm",
            "               board-name: CRS317-1G-16S+",
            "                 platform: MikroTik"
        ]
    ]
}

TASK [Gather facts] ****************************************************************************************
ok: [192.168.50.53]

TASK [Show a fact] ****************************************************************************************
ok: [192.168.50.53] => {
    "msg": "First IP address: 192.168.50.53"
}

TASK [Save config to file] ****************************************************************************************
changed: [192.168.50.53]

PLAY RECAP ****************************************************************************************
192.168.50.53              : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```



### Example: Simple Backup Script

This Example shows how easy it is to backup a mikrotik  device. we can write up a simple playbook + inventory, each of which (for this example) are only 8-10 lines each!

#### Inventory:
Note:  This is simply a smaller version of our example inventory,  we can use the same inventory for multiple playbooks.

```yaml
mikrotik1:
    hosts:
      192.168.50.53:
    vars:
      ansible_connection: network_cli
      ansible_network_os: routeros
      ansible_user: 'admin'
      ansible_password: 'admin'
```

####  Playbook:

```yaml
---
- name: Backup Mikrotik Configs
  hosts: mikrotik1  # Mikrotik Router / Switch name, this can be  changed to match the Hosts / Inventory list supplied
  gather_facts: false
  tasks:
  - name: Gather facts
    community.routeros.facts:
      gather_subset: config
  - name: Save config to file
    ansible.builtin.copy: content={{ansible_net_config_nonverbose}} dest=./{{inventory_hostname}}
```

**Note:**  In this example we use "ansible_net_config_nonverbose". There is a version of this variable called "ansible_net_config", although it produces a config file triple the size with no tangible benefits to the config itself.

####  Example Output:

##### CLI:
```bash
ansible-playbook -i ansible_mikrotik.yml backupconfig.yml 

PLAY [Backup Mikrotik Configs] ********************************************************************

TASK [Gather facts] ********************************************************************
ok: [192.168.50.53]

TASK [Save config to file] *************************************************************
changed: [192.168.50.53]

PLAY RECAP *****************************************************************************
192.168.50.53              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

This also saved my test switches config to a file in my current directory, as  can be seen in the example below,  this looks to have worked saved successfully!

##### Switch Config:
```routeros
# DATE by RouterOS 7.11.2
# software id = XXXXX
#
# model = CRS317-1G-16S+
# serial number = XXXXX

/interface bridge
add admin-mac= auto-mac=no comment=defconf name=bridge

/interface ethernet
set [ find default-name=ether1 ] comment=to-home.carbonos1.net

/interface vlan
add interface=bridge name=VLAN100 vlan-id=100

/interface lte apn
set [ find default=yes ] ip-type=ipv4 use-network-apn=no

/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik

/ip hotspot profile
set [ find default=yes ] html-directory=hotspot

/port
set 0 name=serial0

/interface bridge port
add bridge=bridge comment=to-home-network disabled=yes \
    ingress-filtering=no interface=ether1
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus1
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus2
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus3
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus4
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus5
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus6
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus7
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus8
add bridge=bridge comment=defconf ingress-filtering=no interface=sfp-sfpplus9
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus10
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus11
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus12
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus13
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus14
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus15
add bridge=bridge comment=defconf ingress-filtering=no interface=\
    sfp-sfpplus16
/ip settings
set max-neighbor-entries=8192
/ipv6 settings
set disable-ipv6=yes max-neighbor-entries=8192
/interface ovpn-server server
set auth=sha1,md5
/ip address
add address=192.168.50.53/24 comment=LAN-NETWORK interface=ether1 network=\
    192.168.50.0
/ip dns
set servers=8.8.8.8,4.4.4.4
/ip route
add disabled=no dst-address=0.0.0.0/0 gateway=192.168.50.1
/routing bfd configuration
add disabled=no interfaces=all min-rx=200ms min-tx=200ms multiplier=5
/system clock
set time-zone-name=
/system identity
set name=TOC-MEL-CRS317-1
/system note
set show-at-login=no
/system routerboard settings
set boot-os=router-os
```



### Conclusion:

From half a day messing around with a Mikrotik CRS317 and ansible, I can see this is a VERY Powerful tool. The ability to quickly combine together different playbooks / inventory groups  shows that this tool may  become an essential skill I may require moving forward, and hope to expand upon this set of skills in the future!
