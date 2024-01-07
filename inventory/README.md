
# About our inventory

- Differences from usual Ansible projects
- How bbb-configs uses Ansible
- How hosts are constructed
  - base_inventory
  - keyed groups
  - merge_vars
- Working with the inventory




---

### .. magic inside

this inventory consists of three inventories.


# base_inventory
the first one prints some ansible-compatible hostlist based on all the folders inside host_vars.


# keyed_groups_stage_1.config
this is a configuration for the "constructed" inventory plugin. it dynamically constructs group memberships based on the host_vars. as the first inventory doesnt output any host & group vars, the constructed inventory has to fetchit on its own. This pretty new feature is available since ansible 2.11 . Its controlled by the "use_vars_plugins" key. Unfortunately its searching for the host&group vars in the folder of the first inventory plugin. As its located here, we need to symlink 'em here in order to allow it accomplishing its job.

.. pretty hacky, but less hacky than before :) ..lets hope ansible  will continue improving 


# keyed_groups_stage_2.config
this is yet another configuration for the "constructed" inventory plugin. in contrast to the first stage its purpose is to dynamically construct even more group memberships based on inherited vars from the previous stage.

---

Ansible inventory script for bbb-configs location files

By default, Ansible isn't well suited for how we organize hosts and locations.
We want one complete file for every location, which consists of one or more
hosts with various roles as well as shared and individual config data.
Ansible on the other hand starts at the "host vars" and then
uses the "keyed groups" plugin to merge multiple stages of "group vars".
For our case, this results in an amount of files that can feel overwhelming.

This inventory script allows for complete single location files while
at the same time being compatible with Ansible's host vars and group vars.
Note that certain shared and common config data is still loaded using
keyed groups and group vars where it makes sense (e.g. role and model).

Ansible expects this script to respond to `--host <hostname>` and `--list`.
For more information on inventory scripts, see:
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/script_inventory.html

With `--host <hostname>`, if no matching host is found in `locations/*.yml`,
we then look in Ansible's default `host_vars/` directory.

With `--list`, we scan both locations/ and host_vars/.

Data for hosts in host_vars/ is constructed by Ansible:
1. Combine the data from all files in host_vars/<hostname>/
2. Add keyed_groups as specified in inventory/
3. Merge the data from all related groups in group_vars/<hostname>/

Data for a specific host in locations/ is constructed as follows:
1. Copy the location data into a new object, ignore the `.hosts` array
2. Merge the array entry from `.hosts` with a matching `.hostname` value,
   with shallow merge only at the moment
3. Add keyed_groups as specified in inventory/
4. Merge the data from all related groups in group_vars/<hostname>/

For example:

```yml
location: pktpls
hosts:
  - hostname: pktpls-core
    role: corerouter
    string: host-var-has-precedence
    object: { two: 456 }
    array: [ bar ]
string: will-be-overridden
object: { one: 123 }
array: [ foo ]
```

results in:

```json
{
  "location": "pktpls",
  "hostname": "pktpls-core",
  "role": "corerouter",
  "string": "host-var-has-precedence",
  "object": {
    "two": 456
  },
  "array": [
    "bar"
  ]
}
```
