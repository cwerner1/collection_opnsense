# OPNSense - DNS host override module

**STATE**: stable

**TESTS**: [Playbook](https://github.com/ansibleguy/collection_opnsense/blob/stable/tests/unbound_host.yml)

**API DOCS**: [Core - Unbound](https://docs.opnsense.org/development/api/core/unbound.html)

**BASE DOCS**: [Unbound](https://docs.opnsense.org/manual/unbound.html)

## Definition

For basic parameters see: [Basics](https://github.com/ansibleguy/collection_opnsense/blob/stable/docs/use_basic.md#definition)

| Parameter  | Type   | Required | Default value | Aliases         | Comment                                                                                                                                                                                                                                            |
|:-----------|:-------|:---------|:--------------|:----------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| match_fields     | string | false    | ['hostname', 'domain', 'record_type', 'value', 'prio']              | -               | Fields that are used to match configured host-overrides with the running config - if any of those fields are changed, the module will think it's a new entry. At least one of: 'hostname', 'domain', 'record_type', 'value', 'prio', 'description' |
| hostname     | string | true     | -             | host, h         | Hostname of the record                                                                                                                                                                                                                             |
| domain     | string | true     | -             | dom, d          | Domain of the record                                                                                                                                                                                                                               |
| record_type   | string | true     | 'A'           | type, rr, rt    | Record type. One of: 'A', 'AAAA', 'MX'                                                                                                                                                                                                             |
| value   | string | false for state changes, else true     | -             | server, srv, mx | Value the record should hold                                                                                                                                                                                                                       |
| prio | int    | false    | 10            | mxprio          | Priority that is only used for MX record types                                                                                                                                                                                                     |
| description | string | false    | -             | desc            | Optional description for the host-override. Could be used as unique-identifier when set as only 'match_field'.                                                                                                                                     |
| reload       | boolean | false    | true                 | -               | If the running config should be reloaded on change - this will take some time. For mass-managing items you might want to reload it 'manually' after all changes are done => using the [reload module](https://github.com/ansibleguy/collection_opnsense/blob/stable/docs/use_reload.md). |

## Info

This module manages DNS host-overrides configuration that can be found in the WEB-UI menu: 'Services - Unbound DNS - Overrides - Host overrides'

Entries like these override individual results from the forwarders.

Use these for changing DNS results or for adding custom DNS records.

Keep in mind that all resource record types (i.e. A, AAAA, MX, etc. records) of a specified host below are being overwritten.

## Usage

First you will have to know about **host-matching**.

The module somehow needs to link the configured and existing host-overrides to manage them.

You can to set how this matching is done by setting the 'match_fields' parameter!

The default behaviour is that a host-override is matched by its 'hostname', 'domain', 'record_type', 'value' and 'prio' fields.

However - it is **recommended** to use/set 'description' as **unique identifier** if many overrides are used.

### Mass manage

If you are mass-managing DNS records or using DNS-Blacklists - you might want to disable ```reload: false``` on single module-calls!

This takes a long time, as the service gets reloaded every time!

You might want to reload it 'manually' after all changes are done => using the [reload module](https://github.com/ansibleguy/collection_opnsense/blob/stable/docs/use_reload.md)


## Examples

```yaml
- hosts: localhost
  gather_facts: no
  module_defaults:
    ansibleguy.opnsense.unbound_host:
      firewall: 'opnsense.template.ansibleguy.net'
      api_credential_file: '/home/guy/.secret/opn.key'
      match_fields: ['description']

    ansibleguy.opnsense.list:
      firewall: 'opnsense.template.ansibleguy.net'
      api_credential_file: '/home/guy/.secret/opn.key'
      target: 'unbound_host'

  tasks:
    - name: Example
      ansibleguy.opnsense.unbound_host:
        hostname: 'host'
        domain: 'opnsense.template.ansibleguy.net'
        value: '192.168.0.1'
        # match_fields: ['description']
        # record_type: 'A'
        # prio: 10
        # description: 'example'
        # state: 'present'
        # enabled: true
        # debug: false

    - name: Adding
      ansibleguy.opnsense.unbound_host:
        hostname: 'host'
        domain: 'opnsense.template.ansibleguy.net'
        value: '192.168.0.1'
        description: 'test1'
        # match_fields: ['description']

    - name: Removing
      ansibleguy.opnsense.unbound_host:
        hostname: 'host'
        domain: 'opnsense.template.ansibleguy.net'
        value: '192.168.0.1'
        state: 'absent'
        description: 'test1'
        # match_fields: ['description']

    - name: Adding MX record
      ansibleguy.opnsense.unbound_host:
        hostname: 'mx'
        domain: 'opnsense.template.ansibleguy.net'
        value: 'host.opnsense.template.ansibleguy.net'
        record_type: 'MX'
        description: 'test2'
        # match_fields: ['description']

    - name: Listing hosts
      ansibleguy.opnsense.list:
      #  target: 'unbound_host'
      register: existing_entries

    - name: Printing entries
      ansible.builtin.debug:
        var: existing_entries.data
```
