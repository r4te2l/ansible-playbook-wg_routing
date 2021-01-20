Wireguard Mesh Routing
=========

Manages Wireguard Configuration and Services

Role Variables
--------------

**Generic Variables**

| Variable                      | Purpose                                             | Default             |
| ----------------------------- | --------------------------------------------------- | ------------------- |
| `wg_routing_mesh_group`       | Ansible Group defining mesh members                 | 'wg_routing_mesh'   |
| `wg_routing_hide_private_key` | Hide the wireguard Key from ansible output          | true                |
| `bastion_key_path`            | Full path on bastion host containing wireguard keys | "$HOME/wg_routing"  |
| `sysctl_params`               | Defines Linux sysctl settings                       | See vars/Ubuntu.yml |
| `wg_routing_packages`         | Additional packages to install                      | [ 'bridge-utils' ]  |

**Netplan Variable**

| Variable            | Purpose                                   | Default |
| ------------------- | ----------------------------------------- | ------- |
| netplan_etc_configs | Dictionary defining netplan Configuration | {}      |

**example bridge definition**
```yaml
netplan_etc_configs:
  50-br1:
    network:
      version: 2
      bridges:
        br1:
          addresses:
            - 10.0.0.1/32
```

**Playbook Variables**

| Variable                                                | Purpose                                                         | Type       | Default     | Note                                                                   |
| ------------------------------------------------------- | --------------------------------------------------------------- | ---------- | ----------- | ---------------------------------------------------------------------- |
| `wgr_params`                                            | Dictionary containing Wireguard Mesh Routing configuration      | dictionary | `undefined` |                                                                        |
| `wgr_params.INTERFACE`                                  | Defines the Wireguard Interface                                 | dictionary | `undefined` |                                                                        |
| `wgr_params.INTERFACE.Address`                          | IP Address of the Wireguard Interface INTERFACE (CIDR notation) | string     | `undefined` | **required**                                                           |
| `wgr_params.INTERFACE.ListenPort`                       | Port used by the Wireguard Interface INTERFACE                  | integer    | `undefined` | **required**                                                           |
| `wgr_params.INTERFACE.EIGRPRouterID`                    | EIGRP Router ID, doubles as VXLAN ID                            | integer    | `undefined` | **required**                                                           |
| `wgr_params.INTERFACE.RouterAddress`                    | IP Address used on the VXLAN to exchange routes (CIDR notation) | string     | `undefined` | **required**                                                           |
| `wgr_params.INTERFACE.EIGRPRouterAuto`                  | Allow this playbook to find networks and passive-interfaces     | bool       | true        |                                                                        |
| `wgr_params.INTERFACE.EIGRPRouterBlackListedInterfaces` | Interfaces to always ignore                                     | list       | []          |                                                                        |
| `wgr_params.INTERFACE.EIGRPRouterNetworks`              | List of EIGRP networks (CIDR notation)                          | list       | []          | ignored when `EIGRPRouterBlackListedInterfaces` == true                |
| `wgr_params.INTERFACE.EIGRPRouterPassiveInterfaces`     | List of EIGRP passive interfaces                                | list       | []          | ignored when `EIGRPRouterBlackListedInterfaces` == true                |
| `wgr_params.INTERFACE.PublicEndpoint`                   | Specifies if this host is accessible over a public address      | bool       | false       |                                                                        |
| `wgr_params.INTERFACE.DNS`                              | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.FwMark`                           | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.MTU`                              | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PersistentKeepalive`              | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PostDown`                         | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PostUp`                           | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PreDown`                          | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PresharedKey`                     | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.PreUp`                            | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |
| `wgr_params.INTERFACE.Table`                            | Wireguard Setting                                               |            |             | See [Wireguard Role](https://github.com/r4te2l/ansible-role-wireguard) |


Dependencies
------------

See requirements.txt

Example Inventory
----------------

```yaml
---
all:
  children:
    wg_routing_mesh:
      vars:
        wb_bastion_host: 'wgr-bastion'
      hosts:
        wgr-bastion:
        wgr-1:
          wgr_params:
            wg0:
              Address: 198.51.100.1/24
              ListenPort: 51820
              EIGRPRouterID: 64513
              RouterAddress: 203.0.113.1/25
        wgr-2:
          wgr_params:
            wg0:
              Address: 198.51.100.2/24
              ListenPort: 51820
              EIGRPRouterID: 64513
              RouterAddress: 203.0.113.2/25
        wgr-3:
          wgr_params:
            wg0:
              Address: 198.51.100.3/24
              ListenPort: 51820
              EIGRPRouterID: 64513
              RouterAddress: 203.0.113.3/25
```

License
-------

BSD
