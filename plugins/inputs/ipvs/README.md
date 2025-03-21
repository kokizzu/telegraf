# IPVS Input Plugin

This plugin gathers metrics about the [IPVS virtual and real servers][ipvs]
using the netlink socket interface of the Linux kernel.

> [!IMPORTANT]
> The plugin requires `CAP_NET_ADMIN` and `CAP_NET_RAW` capabilities.
> Check the [permissions section](#permissions) for ways to grant them.

⭐ Telegraf v1.9.0
🏷️ network, system
💻 linux

[ipvs]: http://www.linuxvirtualserver.org/software/ipvs.html

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Collect virtual and real server stats from Linux IPVS
# This plugin ONLY supports Linux
[[inputs.ipvs]]
  # no configuration
```

### Permissions

Assuming you installed the Telegraf package via one of the published packages,
the process will be running as the `telegraf` user. However, in order for this
plugin to communicate over netlink sockets it needs the telegraf process to have
`CAP_NET_ADMIN` and `CAP_NET_RAW` capabilities.

This is the case when running Telegraf as `root` or some user with
`CAP_NET_ADMIN` and `CAP_NET_RAW`. Alternatively, you can add the capabilities
when starting Telegraf via systemd by running `systemctl edit telegraf.service`
and add the following:

```shell
[Service]
CapabilityBoundingSet=CAP_NET_RAW CAP_NET_ADMIN
AmbientCapabilities=CAP_NET_RAW CAP_NET_ADMIN
```

## Metrics

Server will contain tags identifying how it was configured, using one of
`address` + `port` + `protocol` *OR* `fwmark`. This is how one would normally
configure a virtual server using `ipvsadm`.

- ipvs_virtual_server
  - tags:
    - sched (the scheduler in use)
    - netmask (the mask used for determining affinity)
    - address_family (inet/inet6)
    - address
    - port
    - protocol
    - fwmark
  - fields:
    - connections
    - pkts_in
    - pkts_out
    - bytes_in
    - bytes_out
    - pps_in
    - pps_out
    - cps

- ipvs_real_server
  - tags:
    - address
    - port
    - address_family (inet/inet6)
    - virtual_address
    - virtual_port
    - virtual_protocol
    - virtual_fwmark
  - fields:
    - active_connections
    - inactive_connections
    - connections
    - pkts_in
    - pkts_out
    - bytes_in
    - bytes_out
    - pps_in
    - pps_out
    - cps

## Example Output

Virtual server is configured using `fwmark` and backed by 2 real servers:

```text
ipvs_virtual_server,address=172.18.64.234,address_family=inet,netmask=32,port=9000,protocol=tcp,sched=rr bytes_in=0i,bytes_out=0i,pps_in=0i,pps_out=0i,cps=0i,connections=0i,pkts_in=0i,pkts_out=0i 1541019340000000000
ipvs_real_server,address=172.18.64.220,address_family=inet,port=9000,virtual_address=172.18.64.234,virtual_port=9000,virtual_protocol=tcp active_connections=0i,inactive_connections=0i,pkts_in=0i,bytes_out=0i,pps_out=0i,connections=0i,pkts_out=0i,bytes_in=0i,pps_in=0i,cps=0i 1541019340000000000
ipvs_real_server,address=172.18.64.219,address_family=inet,port=9000,virtual_address=172.18.64.234,virtual_port=9000,virtual_protocol=tcp active_connections=0i,inactive_connections=0i,pps_in=0i,pps_out=0i,connections=0i,pkts_in=0i,pkts_out=0i,bytes_in=0i,bytes_out=0i,cps=0i 1541019340000000000
```

Virtual server is configured using `proto+addr+port` and backed by 2 real
servers:

```text
ipvs_virtual_server,address_family=inet,fwmark=47,netmask=32,sched=rr cps=0i,connections=0i,pkts_in=0i,pkts_out=0i,bytes_in=0i,bytes_out=0i,pps_in=0i,pps_out=0i 1541019340000000000
ipvs_real_server,address=172.18.64.220,address_family=inet,port=9000,virtual_fwmark=47 inactive_connections=0i,pkts_out=0i,bytes_out=0i,pps_in=0i,cps=0i,active_connections=0i,pkts_in=0i,bytes_in=0i,pps_out=0i,connections=0i 1541019340000000000
ipvs_real_server,address=172.18.64.219,address_family=inet,port=9000,virtual_fwmark=47 cps=0i,active_connections=0i,inactive_connections=0i,connections=0i,pkts_in=0i,bytes_out=0i,pkts_out=0i,bytes_in=0i,pps_in=0i,pps_out=0i 1541019340000000000
```
