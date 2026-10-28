# SNMP exporter

Role `snmp_exporter` deploys `prom/snmp-exporter` on the hosts in the
`snmp_exporter` inventory group. It is enabled independently with
`enable_snmp_exporter: "yes"`.

## Configuration

The existing `C:\Users\dungbh8\monconf\snmp` directory contains both
device-family source files and the generated combined file. Copy the whole
directory into the Ansitheus custom configuration tree, preserving this path:

```text
<configdir>/snmp/snmp.yaml
```

The role uses `snmp.yaml` as the exporter configuration because it already
contains the merged `auths` and `modules` from Check Point, Dell, F5, HSM,
network switches, Palo Alto, and SAN switches. If the combined file is
omitted, the role automatically merges all `.yaml` and `.yml` files in the
`snmp` directory. The role also accepts a combined file under
`snmp_exporter/` and supports both `.yml` and `.yaml` extensions.

## Physical targets

Store all physical-device targets in Prometheus file service discovery. For
example, create:

```text
<configdir>/prometheus/file_sd/snmp_devices.yml
```

The `module` and `auth` labels must exist in the SNMP exporter configuration:

```yaml
- targets:
    - 192.0.2.10
  labels:
    module: checkpoint_fw
    auth: checkpoint_v2
- targets:
    - 192.0.2.20
  labels:
    module: dell_baremetal
    auth: dell_v2
- targets:
    - 192.0.2.30
  labels:
    module: network_switch
    auth: cisco_snmp_v2c
```

Any file named `snmp*.yml`, `snmp*.yaml`, or `snmp*.json` is automatically
loaded as an SNMP file SD source. Prometheus creates one `snmp` scrape job,
rewrites each physical target to the exporter endpoint, and passes `target`,
`module`, and `auth` as SNMP exporter parameters.

The file SD contains device addresses and references to `module`/`auth`.
SNMP credentials and OID walk definitions remain in the exporter `snmp.yaml`,
because file SD cannot define snmp_exporter authentication profiles or
modules.

The default role template contains a small `if_mib` example. It is only a
fallback; a real deployment should provide the generated configuration from
the device-specific SNMP generator.
