# ansible-role-dhcpd

![GitHub](https://img.shields.io/github/license/jam82/ansible-role-dhcpd) ![GitHub last commit](https://img.shields.io/github/last-commit/jam82/ansible-role-dhcpd) ![GitHub issues](https://img.shields.io/github/issues-raw/jam82/ansible-role-dhcpd) ![Travis (.com) branch](https://img.shields.io/travis/com/jam82/ansible-role-dhcpd/main?label=travis) [![Molecule](https://github.com/jam82/ansible-role-dhcpd/actions/workflows/molecule.yml/badge.svg)](https://github.com/jam82/ansible-role-dhcpd/actions/workflows/molecule.yml)

**Ansible role for setting up ISC DHCP Server for IPv4 networks.**

> This role does not manage your firewall.

IPv6 functionality is currently disabled by this role.
It will be extended, when needed.

## Preface

A different approach for managing the dhcp daemon is used across distributions.

**Archlinux**, **RedHat** and **Suse** use two `systemd` unit files,
one for IPv4 called `dhcpd.service` and one for IPv6 called `dhcpd6.service`.
Although **Suse** uses a shared daemon configuration file located at
`/etc/sysconfig/dhcpd`.

**Debian** on the other hand has a legacy `init.d` script called
`isc-dhcp-server` which manages both and can itself be triggered via
`systemctl <status|start|...> isc-dhcp-server`. In **Debian** you enable/disable
the services for the different IP protocols by commenting
in/out `INTERFACESv4/INTERFACESv6` variables
in the daemon configuration file `/etc/default/isc-dhcp-server` and
not via enabling/disabling two differnt services.

And **Alpine** says: do it yourself! :)
Only for dual stack configurations, to be fair.

The configuration is divided in `dhcpd.conf` for IPv4
and `dhcpd6.conf` for IPv6 on almost all distributions,
except in **Alpine**. Here you find only one `dhcpd.conf` by default.

This role uses `dhcpd.conf` only and currently does not touch the `dhcpd6` service.

To keep the abstraction level low, and because I do not want to develop an
ansible-role-specific dsl for dhcpd.conf, the configuration options are piped in `as-is`.

## Default settings

By default this role will only enable DHCPv4 and create empty subnet
declarations for all subnets obtained via `{{ dhcpd_interfaces }}` that do not
have a `{{ dhcpd_subnets }}` declaration.

Empty subnet declarations effectively mean "*don't serve this subnet*".

Subnets must be explicitly configured via `dhcpd_subnets` dict.

## Supported Platforms

| OS Family | Distribution  | Latest | Supported Version(s) | Comment |
|-----------|---------------|--------|----------------------|---------|
| Alpine    | Alpine        | :heavy_check_mark: | 3.12, 3.13 | |
| Archlinux | Archlinux     | :heavy_check_mark: | - | |
|           | Manjaro       | :heavy_check_mark: | - | |
| Debian    | Debian        | :heavy_check_mark: | 10, 11 | |
|           | Ubuntu        | :heavy_check_mark: | 18.04, 20.04 | |
| RedHat    | Almalinux     | :heavy_check_mark: | 8 | |
|           | Amazonlinux   | :x: | - | not tested, image not working |
|           | Centos        | :heavy_check_mark: | 8 | |
|           | Fedora        | :heavy_check_mark: | 33, 34, Rawhide | |
|           | Oraclelinux   | :heavy_check_mark: | 8 | |
| Suse      | OpenSuse Leap | :x: | 15.1, 15.2, 15.3 | systemd-timesync: interface name too long (is 16) !?! |
|           | Tumbleweed    | :heavy_check_mark: | - | |

## Requirements

Ansible 2.9 or higher.

## Variables

Variables and defaults for this role.

### defaults/main.yml

```yaml
---
# role: ansible-role-dhcpd
# file: defaults/main.yml

# The role is disabled by default, so you do not get in trouble.
# Checked in tasks/main.yml which includes tasks/tasks.yml if enabled.
dhcpd_role_enabled: false

# list of ipv4 interfaces to listen on.
# this must always contain at least one interface. sort is for idempotence.
dhcpd_interfaces: "{{ ansible_interfaces | difference(['lo']) | sort }}"

# dhcpd options. put all global configuration here.
dhcpd_conf: |
  default-lease-time 600;
  max-lease-time 7200;
  ddns-update-style none;


# dhcpd subnet declarations. put all subnet specific configuration here.
dhcpd_subnets: []
# - subnet: 172.17.0.0
#   netmask: 255.255.255.0
#   conf: |
#     range 172.17.0.32 172.17.0.248;

#     option broadcast-address 172.17.0.255;
#     option routers 172.17.0.1;
#     option domain-name-servers 172.17.0.1;

#     # class "pxeclients" {
#     #     match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
#     #     next-server 172.17.0.10;

#     #     if exists user-class and option user-class = "iPXE" {
#     #         filename "http://172.17.0.10/ipxe";
#     #     }
#     #     elsif option pxe-system-type = 00:07 {
#     #         filename "ipxe.efi";
#     #     } else {
#     #         filename "undionly.kpxe";
#     #     }
#     # }
```

## Dependencies

None.

## Example Playbook

```yaml
---
# role: ansible-role-dhcpd
# play: dhcpd
# file: dhcpd.yml

- hosts: all
  become: true
  gather_facts: true
  vars:
    dhcpd_role_enabled: true
  roles:
    - role: ansible-role-dhcpd
```

## License and Author

- Author:: [jam82](https://github.com/jam82/)
- Copyright:: 2021, [jam82](https://github.com/jam82/)

Licensed under [MIT License](https://opensource.org/licenses/MIT).
See [LICENSE](https://github.com/jam82/ansible-role-dhcpd/blob/master/LICENSE) file in repository.

## References

- [ArchWiki](https://wiki.archlinux.org/)
