# Ansible Role - Uncomplicated Firewall (UFW)

Ansible Role to deploy/configure the software firewall 'UFW' on a debian-based linux server.  
Forked from `ansibleguy/linux_ufw`

## Functionality

This ansible role will do:

* **Package installation**
  * UFW

* **Configuration**
  * Rules via using **one of two modes**
    * The **stateful** way (_default_)
      * keeps existing rules and adds/removes rules using a rule state
    * The **stateless** way
      * reset's the ufw state and rules every time
      * after that the new rules get applied

  * Verification that a ssh-rule is in place

## Info

* **Note:** Most of the role's functionality can be opted in or out.

  For all available options - see the default-config located in the main defaults-file!

* **Note:** this role currently only supports debian-based systems

* **Warning:** Not every setting/variable you provide will be checked for validity. Bad config might break the role!

## Usage

### Config

Define at least one of the following dictionaries as needed:

```yaml
ufw_global_rules:
  ruleShortName:
    rule: 'allow'  # default if empty
    port: 80
    proto: 'tcp'
    log: 'no'  # default if empty
    from_ip: 'any'  # default if empty
    to_ip: 'any'  # default if empty
    direction: 'in'  # default if empty
    present: true  # default if empty => will be used for stateful rule-check (alias = state: present)
    position: 2  # you can define the position of the rule in the ruleset (alias = insert)
    comment: 'You can overwrite the default comment'

ufw_group_rules:
  ruleShortName:
    rule: 'allow'  # default if empty
    port: 80
    proto: 'tcp'
    log: 'no'  # default if empty
    from_ip: 'any'  # default if empty
    to_ip: 'any'  # default if empty
    direction: 'in'  # default if empty
    present: true  # default if empty => will be used for stateful rule-check (alias = state: present)
    position: 2  # you can define the position of the rule in the ruleset (alias = insert)
    comment: 'You can overwrite the default comment'

ufw_host_rules:
  ruleShortName:
    rule: 'allow'  # default if empty
    port: 80
    proto: 'tcp'
    log: 'no'  # default if empty
    from_ip: 'any'  # default if empty
    to_ip: 'any'  # default if empty
    direction: 'in'  # default if empty
    present: true  # default if empty => will be used for stateful rule-check (alias = state: present)
    position: 2  # you can define the position of the rule in the ruleset (alias = insert)
    comment: 'You can overwrite the default comment'
```

If you want to support IPv6 as well, set the following variable:

```yaml
ufw_config:
  ipv6: 'yes'
```

### Execution

Run the playbook:

```bash
ansible-playbook -K -D -i inventory/hosts.yml playbook.yml
```

The ufw-task itself is '[community.general.ufw](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html)'

### Example

**State before:**

```bash
chris@ansible:~$ sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 7424/tcp                   ALLOW IN    Anywhere                   # Ansible managed - confusedService
[ 2] 7429/tcp                   ALLOW IN    Anywhere                   (log) # Ansible managed - nothingImportant
```

**Config**

```yaml
ufw_global_rules:
  # incoming traffic restrictions
  SSH:
    port: 22
    proto: 'tcp'
    log: true
    rule: 'limit'
  RandomWebServer:
    port: 8482
    proto: 'tcp'
  SecureLink:
    port: 54038:54085
    proto: 'udp'
    log: true
    from_ip: '192.168.194.0/28'
  ipsecESP:
    proto: 'esp'
    from_ip: '10.10.10.1'
    to_ip: '10.10.20.254'
  ipsecIKE:
    port: 500,4500
    proto: 'udp'
    from_ip: '10.10.10.1'
    to_ip: '10.10.20.254'

  # outgoing traffic restrictions
  denyNtpOutgoing:
    port: 123
    proto: 'udp'
    rule: 'deny'
    direction: 'out'

  # remove those rules:
  confusedService:
    port: 7424
    proto: 'tcp'
    state: 'absent'
  nothingImportant:
    port: 7429
    proto: 'tcp'
    log: true
    present: false
```

**Result:**

```bash
chris@ansible:~$ sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     LIMIT IN    Anywhere                   (log) # Ansible managed - SSH
[ 2] 8482/tcp                   ALLOW IN    Anywhere                   # Ansible managed - RandomWebServer
[ 3] 54038:54085/udp            ALLOW IN    192.168.194.0/28           (log) # Ansible managed - SecureLink
[ 4] 10.10.20.254/esp           ALLOW IN    10.10.10.1/esp             # Ansible managed - ipsecESP
[ 5] 10.10.20.254 500,4500/udp  ALLOW IN    10.10.10.1                 # Ansible managed - ipsecIKE
[ 6] 123/udp                    DENY OUT    Anywhere                   (out) # Ansible managed - denyNtpOutgoing
```
