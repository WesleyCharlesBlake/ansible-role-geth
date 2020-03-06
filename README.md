<p><img src="https://code.benco.io/icon-collection/logos/ansible.svg" alt="ansible logo" title="ansible" align="left" height="60" /></p>
<p><img src="https://retohercules.com/images/ethereum-png-11.png" alt="ethereum logo" title="ethereum" align="right" height="80" /></p>

Ansible Role :computer: :link: Geth
=========

**Table of Contents**
  - [Supported Platforms](#supported-platforms)
  - [Requirements](#requirements)
  - [Role Variables](#role-variables)
      - [Install](#install)
      - [Config](#config)
      - [Launch](#launch)
      - [Uninstall](#uninstall)
  - [Dependencies](#dependencies)
  - [Example Playbook](#example-playbook)
  - [License](#license)

Ansible role that installs, configures and runs [Geth](https://geth.ethereum.org/): a command-line interface and API server for operating an ethereum node.

##### Supported Platforms:
```
* Debian
* MacOS
* Redhat(CentOS/Fedora)
* Ubuntu
```

Requirements
------------

Requires the `unzip/gtar` utility to be installed on the target host. See ansible `unarchive` module [notes](https://docs.ansible.com/ansible/latest/modules/unarchive_module.html#notes) for details.

Role Variables
--------------
Variables are available and organized according to the following software & machine provisioning stages:
* _config_
* _launch_


#### Config

Configuration of the `geth` client can be expressed in a config file written in [TOML](https://github.com/toml-lang/toml), a minimal markup language, used as an alternative to passing command-line flags at runtime. To get an idea how the config should look you can use the `geth dumpconfig` subcommand to export a client's existing configuration.

_The following variables can be customized to manage the location and content of this TOML configuration:_

`config_dir: </path/to/configuration/dir>` (**default**: `/etc/geth`)
- path on target host where the `geth` TOML configuration should be stored

`geth_config: {"<config-section>": {"<section-setting>": "<setting-value>",..},..}` **default**: see `defaults/main.yml`

* Any configuration setting/value key-pair supported by `geth` should be expressible within the `geth_config` hash and properly rendered within the associated TOML config. Values can be expressed in typical _yaml/ansible_ form (e.g. Strings, numbers and true/false values should be written as is and without quotes).

  Furthermore, configuration is not constrained by hardcoded author defined defaults or limited by pre-baked templating. If the config section, setting and value are recognized by the `geth` tool, :thumbsup: to define within `geth_config`.

  A list of configurable settings can be found [here](https://gist.github.com/0x0I/5887dae3cdf4620ca670e3b194d82cba).

  Keys of the `geth_config` hash represent TOML config sections:
  ```yaml
  geth_config:
    # [TOML Section 'Shh']
    Shh: {}
  ```

  Values of `geth_config[<key>]` represent key,value pairs within an embedded hash expressing config settings:
  ```yaml
  geth_config:
    # TOML Section '[Shh]'
    Shh:
      # Section setting MaxMessageSize with value of 1048576
      MaxMessageSize: 1048576
  ```



##### Examples

  Connect to either the Ropsten PoW(proof-of-work) or Rinkeby PoA(proof-of-authory) pre-configured test network:
  ```
  extra_run_args:
    - '--testnet' # PoW
  # ...or...
  extra_run_args:
    - '--rinkeby' # PoA
  ```

  Enhance logging and debugging capabilities for troubleshooting issues:
  ```
  extra_run_args:
    - --debug
    - '--verbosity 5'
    - '--trace /tmp/geth.trace'
  ```

  Enable client and server profiling for analytics and testing purposes:
  ```
  extra_run_args:
    - --pprof
    - '--memprofilerate 1048576'
    - '--blockprofilerate 1'
    - '--cpuprofile /tmp/geth-cpu-profile'
  ```

`custom_unit_properties: <hash-of-systemd-service-settings>` (**default**: `[]`)
- hash of settings used to customize the `[Service]` unit configuration and execution environment of the *Geth* **systemd** service.

#### Uninstall

Support for uninstalling and removing artifacts necessary for provisioning allows for users/operators to return a target host to its configured state prior to application of this role. This can be useful for recycling nodes and roles and perhaps providing more graceful/managed transitions between tooling upgrades.

_The following variable(s) can be customized to manage this uninstall process:_

`perform_uninstall: <true | false>` (**default**: `false`)
- whether to uninstall and remove all artifacts and remnants of this `geth` installation on a target host (**see**: `handlers/main.yml` for details)

Dependencies
------------

- 0x0i.systemd

Example Playbook
----------------
Basic setup with defaults:
```
- hosts: all
  roles:
  - role: geth
```

Launch an Ethereum light client and connect it to the Rinkeby PoA (Proof of Authority) test network:
```
- hosts: light-client
  roles:
  - role: geth
    vars:
      geth_config:
        Eth:
          SyncMode: light
      extra_run_args:
        - --rinkeby
```

Run a full Ethereum node using "fast" sync-mode (only process most recent transactions), enabling both the RPC server interface and client miner and overriding the (block) data directory:
```
- hosts: full-node
  roles:
  - role: geth
    vars:
      geth_config:
        Eth:
          SyncMode: fast
        Node:
          DataDir: /mnt/geth
      extra_run_args:
        - --rpc
        - '--rpcaddr="0.0.0.0'
        - '--mine --miner.threads 16'
```