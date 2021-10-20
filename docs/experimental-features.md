# Experimental Features

Experimental features are those features that work, but haven't been thoroughly tested enough to feel confident to use in production.

## Rollback Configuration

Starting in version 2.0.2, a featured called rollback configuraiton was introduced. The rollback configuration is exactly what it sounds like. It renders a rollback configuration in the event that a remediation causes a hiccup when being deployed. The rollback configuration does the inverse on a remediation. Instead of a remediation being renedered based upon the generated config, a rollback remediation is rendered from the generated config based upon the running configuration.

A rollback configuration can be rendered once the running and generated configurations are loaded. Below is an example.

```bash
>>> from hier_config import Host
>>> host = Host(hostname="aggr-example.rtr", os="ios")
>>> host.load_running_config_from_file("./tests/fixtures/running_config.conf")
>>> host.load_generated_config_from_file("./tests/fixtures/generated_config.conf")
>>> rollback = host.rollback_config()
>>> for line in rollback.all_children_sorted():
...     print(line.cisco_style_text())
... 
no vlan 4
no interface Vlan4
vlan 3
  name switch_mgmt_10.0.4.0/24
interface Vlan2
  no mtu 9000
  no ip access-group TEST in
  shutdown
interface Vlan3
  description switch_mgmt_10.0.4.0/24
  ip address 10.0.4.1 255.255.0.0
>>> 
```


## Unified diff

Starting in version 2.1.0, a featured called unified diff was introduced. It provides a similar output to difflib.unified_diff() but is aware of out of order lines and the parent child relationships present in the hier_config model of the configurations being diffed.  

This feature is useful in cases where you need to compare the differences of two network device configurations. Such as comparing the configs of redundant device pairs. Or, comparing running and intended configs. 

In its current state, this algorithm does not consider duplicate child differences. e.g. two instances `endif` in an IOS-XR route-policy. It also does not respect the order of commands where it may count, such as in ACLs. In the case of ACLs, they should contain sequence numbers if order is important.

```bash
In [1]: list(running_config.unified_diff(generated_config))
Out[1]:
['vlan 3',
 '  - name switch_mgmt_10.0.4.0/24',
 '  + name switch_mgmt_10.0.3.0/24',
 'interface Vlan2',
 '  - shutdown',
 '  + mtu 9000',
 '  + ip access-group TEST in',
 '  + no shutdown',
 'interface Vlan3',
 '  - description switch_mgmt_10.0.4.0/24',
 '  - ip address 10.0.4.1 255.255.0.0',
 '  + description switch_mgmt_10.0.3.0/24',
 '  + ip address 10.0.3.1 255.255.0.0',
 '+ vlan 4',
 '  + name switch_mgmt_10.0.4.0/24',
 '+ interface Vlan4',
 '  + mtu 9000',
 '  + description switch_mgmt_10.0.4.0/24',
 '  + ip address 10.0.4.1 255.255.0.0',
 '  + ip access-group TEST in',
 '  + no shutdown']
```
        