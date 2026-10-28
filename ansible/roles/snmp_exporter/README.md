# Ansible Role: snmp-exporter

Deploys [Prometheus snmp_exporter](https://github.com/prometheus/snmp_exporter) with Docker.

The role expects the generated, combined configuration at:

```text
<configdir>/snmp/snmp.yaml
```

This matches the existing `monconf\snmp` layout. The role also accepts the
same file under `<configdir>/snmp_exporter/`, plus `snmp.yml` variants.

The individual files such as `checkpoint.yaml`, `dell.yaml`, `f5.yaml`,
`hsm.yaml`, `network.yaml`, `paloalto.yaml`, and `sanswitch.yaml` are
device-family configurations. If `snmp.yaml` is present, it is used directly.
If it is not present, the role automatically merges all `.yaml` and `.yml`
files in the directory into one exporter configuration. The existing
`snmp.yaml` already contains these modules:

```text
firewall, firewall_tenant, dell_baremetal, luna_hsm,
network_switch, checkpoint_fw, paloalto_fw
```

Define physical devices in Prometheus file service discovery under:

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

Any file whose name matches `snmp*.yml`, `snmp*.yaml`, or `snmp*.json` is
automatically treated as an SNMP file SD source. Prometheus scrapes all listed
devices through the SNMP exporter using `/snmp`.

File SD stores the device address and the `module`/`auth` names. The actual
SNMP credentials and OID walk definitions remain in `snmp.yaml`, because
Prometheus file SD cannot define snmp_exporter modules or authentication
profiles.

