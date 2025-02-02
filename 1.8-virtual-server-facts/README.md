# Exercise 1.8: Gather and display virtual server facts.

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Playbook Output](#playbook-output)
- [Solution](#solution)

# Objective

Demonstrate use of the [BIG-IP config module](https://docs.ansible.com/ansible/latest/modules/bigip_device_facts.html) to save the running configuration to disk

# Guide

## Step 1:

Using your text editor of choice create a new file called `bigip-virtual-server-facts.yml`.

```
[student1@ansible ~]$ nano bigip-virtual-server-facts.yml
```

>`vim` and `nano` are available on the control node, as well as Visual Studio and Atom via RDP

## Step 2:

Ansible playbooks are **YAML** files. YAML is a structured encoding format that is also extremely human readable (unlike it's subset - the JSON format).

Enter the following play definition into `bigip-virtual-server-facts.yml`:

``` yaml
---
- name: GET F5 FACTS
  hosts: lb
  connection: local
  gather_facts: false
```

- The `---` at the top of the file indicates that this is a YAML file.
- The `hosts: f5`,  indicates the play is run only on the F5 BIG-IP device
- `connection: local` tells the Playbook to run locally (rather than SSHing to itself)
- `gather_facts: no` disables facts gathering.  We are not using any fact variables for this playbook.

## Step 3

Next, add the `task`. This task will set the 'provider' facts and use `bigip_device_facts` to collect virtual server object information in a variable.

``` yaml
---
- name: BIG-IP SETUP
  hosts: lb
  connection: local
  gather_facts: false

  vars:
    default_pool: "/Common/http-pool"
    vip_port: 8088

  tasks:

    - set_fact:
       provider:
        server: "{{private_ip}}"
        user: "{{ansible_user}}"
        password: "{{ansible_ssh_pass}}"
        server_port: 8443
        validate_certs: no

    - name: Collect information of all virtual servers
      bigip_device_facts:
        gather_subset:
         - virtual-servers
        provider: "{{provider}}"
      register: facts_result

    - name: Display the results
      debug:
        var: facts_result
        
```


>A play is a list of tasks. Tasks and modules have a 1:1 correlation.  Ansible modules are reusable, standalone scripts that can be used by the Ansible API, or by the ansible or ansible-playbook programs. They return information to ansible by printing a JSON string to stdout before exiting.

- `vars:` defines the variables which will be used elsewhere in this defined playbook.
- `default_pool:` is a user defined variable which uses /Common/http-pool as the default pool.
- `vip_port:` is a user defined variable which sets the vip service port to listen on port 8888.
- `set_fact:` this Ansible module allows setting variables which will be available for subsequent plays throughout a playbook run.
- The `provider:` creates a reusable object containing connection details.
- The `server: "{{private_ip}}"` parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable `private_ip` in inventory
- The `user: "{{ansible_user}}"` parameter tells the module the username to login to the F5 BIG-IP device with
- The`password: "{{ansible_ssh_pass}}"` parameter tells the module the password to login to the F5 BIG-IP device with
- The `server_port: 8443` parameter tells the module the port to connect to the F5 BIG-IP device with
- The `validate_certs: "no"` parameter tells the module to not validate SSL certificates.  This is just used for demonstration purposes   since this is a lab.
- `name: "Collect information of all virtual servers" ` is a user defined description that will display in the terminal output.
- `bigip_device_facts:` tells the task which module to use.  Everything except `register` is a module parameter defined on the module documentation page.
- The `gather_subset: virtual-servers` parameter tells the module only to grab virtual system information.
- `register: facts_result` tells the task to save the output to a variable facts_result.
- The `name: Display the results` is a user defined description that will display in the terminal output.
- `debug:` tells the task to use the debug module.
- The `var: facts_result` parameter tells the module to display the variable facts_result.

## Step 4

Run the playbook - exit back into the command line of the control host and execute the following:

```
[student1@ansible ~]$ ansible-playbook bigip-virtual-server-facts.yml
```

# Playbook Output

```yaml
[student1@ansible ~]$ ansible-playbook bigip-virtual-server-facts.yml

PLAY [GET F5 FACTS] **************************************************************

TASK [set_fact] ******************************************************************
ok: [f5]

TASK [Collect information of all virtual servers] ********************************
changed: [f5]

TASK [Display the results] *******************************************************
ok: [f5] =>
  facts_result:
    ansible_facts:
      discovered_interpreter_python: /usr/bin/python
    changed: true
    failed: false
    virtual_servers:
    - auto_lasthop: default
      availability_status: available
      client_side_bits_in: 430288
      client_side_bits_out: 983960
      client_side_current_connections: 0
      client_side_evicted_connections: 0
      client_side_max_connections: 5
      client_side_pkts_in: 454
      client_side_pkts_out: 523
      client_side_slow_killed: 0
      client_side_total_connections: 60
      cmp_enabled: 'yes'
      cmp_mode: all-cpus
      connection_limit: 0
      connection_mirror_enabled: 'no'
      cpu_usage_ratio_last_1_min: 0
      cpu_usage_ratio_last_5_min: 0
      cpu_usage_ratio_last_5_sec: 0
      current_syn_cache: 0
      default_pool: /Common/http_pool
      destination: /Common/172.16.82.208:443
      destination_address: 172.16.82.208
      destination_port: 443
      enabled: 'yes'
      ephemeral_bits_in: 0
      ephemeral_bits_out: 0
      ephemeral_current_connections: 0
      ephemeral_evicted_connections: 0
      ephemeral_max_connections: 0
      ephemeral_pkts_in: 0
      ephemeral_pkts_out: 0
      ephemeral_slow_killed: 0
      ephemeral_total_connections: 0
      full_path: /Common/vip
      gtm_score: 0
      hardware_syn_cookie_instances: 0
      irules:
      - /Common/irule1
      - /Common/irule2
      max_conn_duration: 679290
      mean_conn_duration: 19248
      min_conn_duration: 36
      name: vip
      nat64_enabled: 'no'
      profiles:
      - context: client-side
        full_path: /Common/clientssl
        name: clientssl
      - context: all
        full_path: /Common/http
        name: http
      - context: all
        full_path: /Common/oneconnect
        name: oneconnect
      - context: all
        full_path: /Common/tcp
        name: tcp
      protocol: tcp
      rate_limit: -1
      rate_limit_destination_mask: 0
      rate_limit_mode: object
      rate_limit_source_mask: 0
      snat_type: automap
      software_syn_cookie_instances: 0
      source_address: 0.0.0.0/0
      source_port_behavior: preserve
      status_reason: The virtual server is available
      syn_cache_overflow: 0
      syn_cookies_status: not-activated
      total_hardware_accepted_syn_cookies: 0
      total_hardware_syn_cookies: 0
      total_requests: 113
      total_software_accepted_syn_cookies: 0
      total_software_rejected_syn_cookies: 0
      total_software_syn_cookies: 0
      translate_address: 'yes'
      translate_port: 'yes'
      type: standard

PLAY RECAP ***********************************************************************
f5                         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
## Step 5
The result of the variable facts_result is shown in the playbook output. This output will get used to find the more specific information we want to obtain.

Next, add underneath yaml file to the bottom of the bigip-virtual-server-facts.yml playbook.

``` yaml

    - name: Display all VIP's available
      debug: "msg={{item.name}}"
      loop: "{{facts_result.virtual_servers}}"
      loop_control:
        label: "{{item.name}}"

    - name: Display VIP's that has a specific destination port
      debug: "msg={{item.name}}"
      when: item.destination_port == vip_port
      loop: "{{facts_result.virtual_servers}}"
      loop_control:
        label:
        - "{{item.name}}"
        - "{{item.destination_port}}"

    - name: Display VIP's that have a specific default pool
      debug: "msg={{item.name}}"
      when: item.default_pool == default_pool
      loop: "{{facts_result.virtual_servers}}"
      loop_control:
        label:
        - "{{item.name}}"
        - "{{item.default_pool}}"

    - name: Store the first vip name in a variable
      set_fact:
        first_vip_name: "{{facts_result.virtual_servers[0].name}}"

    - name: Display all profiles attached to a VIP "name={{first_vip_name}}"
      debug: "msg={{item}}"
      loop: "{{facts_result.virtual_servers | json_query(query_string)}}"
      vars:
       query_string: "[?name=='{{first_vip_name}}'].profiles[*].name"
        
```

Explanation  of the used functions:
- `name: "Display all VIP's available" ` is a user defined description that will display in the terminal output.
- `debug:` This module prints statements during execution where msg is pulling the name from the facts_result output
- `loop:` tells the task to loop over the provided list. The list of virtual servers is taken from the facts_result output.
- `loop_control:` to limit the output a label of name is used for each item.
For the next sections we see a returning partern where 'loop_control' is used to limit the outcome of the loop function by using the labels name, destination_port, default_pool.
- `when:` is used to compare results from the facts_result variable output and only when this is true go through the loop and output the results.
- `vars:` the query string is a variable which is used to get the result from the JSON filter in the previous loop. It requests to search for the 'first vip name' and deliver the configured profiles as the output.

# Step 6

Run the playbook - exit back into the command line of the control host and execute the following:

```
[student1@ansible ~]$ ansible-playbook bigip-virtual-server-facts.yml
```

# Playbook Output

```yaml
PLAY [GET F5 FACTS] **************************************************************

TASK [set_fact] ******************************************************************
ok: [f5]

TASK [Collect information of all virtual servers] ********************************
changed: [f5]

TASK [Display the results] *******************************************************
ok: [f5] =>
  facts_result:
    ansible_facts:
      discovered_interpreter_python: /usr/bin/python
    changed: true
    failed: false
    virtual_servers:
    - auto_lasthop: default
      availability_status: available
      client_side_bits_in: 431248
      client_side_bits_out: 984280
      client_side_current_connections: 0
      client_side_evicted_connections: 0
      client_side_max_connections: 5
      client_side_pkts_in: 457
      client_side_pkts_out: 524
      client_side_slow_killed: 0
      client_side_total_connections: 61
      cmp_enabled: 'yes'
      cmp_mode: all-cpus
      connection_limit: 0
      connection_mirror_enabled: 'no'
      cpu_usage_ratio_last_1_min: 0
      cpu_usage_ratio_last_5_min: 0
      cpu_usage_ratio_last_5_sec: 0
      current_syn_cache: 0
      default_pool: /Common/http_pool
      destination: /Common/172.16.82.208:443
      destination_address: 172.16.82.208
      destination_port: 443
      enabled: 'yes'
      ephemeral_bits_in: 0
      ephemeral_bits_out: 0
      ephemeral_current_connections: 0
      ephemeral_evicted_connections: 0
      ephemeral_max_connections: 0
      ephemeral_pkts_in: 0
      ephemeral_pkts_out: 0
      ephemeral_slow_killed: 0
      ephemeral_total_connections: 0
      full_path: /Common/vip
      gtm_score: 0
      hardware_syn_cookie_instances: 0
      irules:
      - /Common/irule1
      - /Common/irule2
      max_conn_duration: 679290
      mean_conn_duration: 19044
      min_conn_duration: 36
      name: vip
      nat64_enabled: 'no'
      profiles:
      - context: client-side
        full_path: /Common/clientssl
        name: clientssl
      - context: all
        full_path: /Common/http
        name: http
      - context: all
        full_path: /Common/oneconnect
        name: oneconnect
      - context: all
        full_path: /Common/tcp
        name: tcp
      protocol: tcp
      rate_limit: -1
      rate_limit_destination_mask: 0
      rate_limit_mode: object
      rate_limit_source_mask: 0
      snat_type: automap
      software_syn_cookie_instances: 0
      source_address: 0.0.0.0/0
      source_port_behavior: preserve
      status_reason: The virtual server is available
      syn_cache_overflow: 0
      syn_cookies_status: not-activated
      total_hardware_accepted_syn_cookies: 0
      total_hardware_syn_cookies: 0
      total_requests: 113
      total_software_accepted_syn_cookies: 0
      total_software_rejected_syn_cookies: 0
      total_software_syn_cookies: 0
      translate_address: 'yes'
      translate_port: 'yes'
      type: standard

TASK [Display all VIP's available] ***********************************************
ok: [f5] => (item=vip) =>
  msg: vip

TASK [Display VIP's that has a specific destination port] ************************
skipping: [f5] => (item=[u'vip', u'443'])
skipping: [f5]

TASK [Display VIP's that have a specific default pool] ***************************
skipping: [f5] => (item=[u'vip', u'/Common/http_pool'])
skipping: [f5]

TASK [Store the first vip name in a variable] ************************************
ok: [f5]

TASK [Display all profiles attached to a VIP "name=vip"] *************************
ok: [f5] => (item=[u'clientssl', u'http', u'oneconnect', u'tcp']) =>
  msg:
  - clientssl
  - http
  - oneconnect
  - tcp

PLAY RECAP ***********************************************************************
f5                         : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```



# Solution

The finished Ansible Playbook is provided here for an Answer key.  Click here: [bigip-virtual-server-facts.yml](../1.8-virtual-server-facts/bigip-virtual-server-facts.yml).

You have finished this exercise.  [Click here to return to the lab guide](../README.md)
