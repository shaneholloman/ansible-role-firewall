# Ansible Role: `firewall`

[![Release](https://github.com/shaneholloman/ansible-role-firewall/actions/workflows/release.yml/badge.svg)](https://github.com/shaneholloman/ansible-role-firewall/actions/workflows/release.yml)

Installs an iptables-based firewall for Linux.  
Supports both IPv4 (`iptables`) and IPv6 (`ip6tables`).

This firewall aims for simplicity over complexity, and only opens a few specific ports for incoming traffic (configurable through Ansible variables). If you have a rudimentary knowledge of `iptables` and/or firewalls in general, this role should be a good starting point for a secure system firewall.

After the role is run, a `firewall` init service will be available on the server.  
You can use `service firewall [start|stop|restart|status]` to control the firewall.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yml
firewall_state: started
firewall_enabled_at_boot: true
```

Controls the state of the firewall service; whether it should be running (`firewall_state`) and/or enabled on system boot (`firewall_enabled_at_boot`).

```yml
firewall_flush_rules_and_chains: true
```

Whether to flush all rules and chains whenever the firewall is restarted. Set this to `false` if there are other processes managing iptables (e.g. Docker).

```yml
firewall_allowed_tcp_ports:
  - "22"
  - "80"
  ...
firewall_allowed_udp_ports: []
```

A list of TCP or UDP ports (respectively) to open to incoming traffic.

```yml
firewall_forwarded_tcp_ports:
  - { src: "22", dest: "2222" }
  - { src: "80", dest: "8080" }
firewall_forwarded_udp_ports: []
```

Forward `src` port to `dest` port, either TCP or UDP (respectively).

```yml
firewall_additional_rules: []
firewall_ip6_additional_rules: []
```

Any additional (custom) rules to be added to the firewall (in the same format you would add them via command line, e.g. `iptables [rule]`/`ip6tables [rule]`).

A couple of examples of how this could be used:

```yml
# Allow only the IP 167.89.89.18 to access port 4949 (Munin).
firewall_additional_rules:
  - "iptables -A INPUT -p tcp --dport 4949 -s 167.89.89.18 -j ACCEPT"
```

```yml
# Allow only the IP 214.192.48.21 to access port 3306 (MySQL).
firewall_additional_rules:
  - "iptables -A INPUT -p tcp --dport 3306 -s 214.192.48.21 -j ACCEPT"
```

See [Iptables Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands) for more examples.

```yml
firewall_log_dropped_packets: true
```

Whether to log dropped packets to syslog (messages will be prefixed with "Dropped by firewall: ").

```yml
firewall_disable_firewalld: false
firewall_disable_ufw: false
```

Set to `true` to disable firewalld (installed by default on RHEL/CentOS) or ufw (installed by default on Ubuntu), respectively.

```yml
firewall_enable_ipv6: true
```

Set to `false` to disable configuration of ip6tables (for example, if your `GRUB_CMDLINE_LINUX` contains `ipv6.disable=1`).

## Dependencies

None.

## Example Playbook

```yml
- hosts: server
  vars_files:
    - vars/main.yml
  roles:
    - { role: shaneholloman.firewall }
```

*Inside `vars/main.yml`*:

```yml
firewall_allowed_tcp_ports:
  - "22"
  - "25"
  - "80"
```

## TODO

  - [ ] Make outgoing ports more configurable.
  - [ ] Make other firewall features (like logging) configurable.

## License

Unlicense

## Author Information

Shane Holloman 2023
