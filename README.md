Ansible Role Prometheus Exporter
========

[![Build Status](https://travis-ci.com/Turgon37/ansible-prometheus-exporter.svg?branch=master)](https://travis-ci.com/Turgon37/ansible-prometheus-exporter)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-Turgon37.prometheus_exporter-blue.svg)](https://galaxy.ansible.com/Turgon37/prometheus_exporter/)

## Description

:grey_exclamation: Before using this role, please know that all my Ansible roles are fully written and accustomed to my IT infrastructure. So, even if they are as generic as possible they will not necessarily fill your needs, I advice you to carrefully analyse what they do and evaluate their capability to be installed securely on your servers.

This roles can install and configure most of the prometheus exporter.

## Requirements

Require Ansible >= 2.4

### Dependencies

## OS Family

This role is available for Debian and CentOS

## Features

At this day the role can be used to :

  * install exporter using corresponding host architecture
  * configure exporter
     * binary capabilities
     * custom files
     * service command line
  * configure a systemd service to launch the exporter
  * known exporters : node_exporter, blackbox_exporter
  * [local facts](#facts)

## Configuration

### Role

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below. To see default values please refer to this file.

| Name                                           | Type                            | Description                                                                                                                       |
| ---------------------------------------------- | --------------------------------| ----------------------------------------------------------------------------------------------------------------------------------|
| `prometheus_exporter__name`                    | String                          | The exporter name (use the official name here)                                                                                    |
| `prometheus_exporter__download_url`            | Url                             | If the exporter name is not known by the role, you must provide the download url (see example below)                              |
| `prometheus_exporter__checksum_url`            | Url                             | If the exporter name is not known by the role, you must provide the checksum url (see example below)                              |
| `prometheus_exporter__temporary_file_extension`| String                          | If the exporter name is not known by the role, you must fill here with downloaded file's extension (see example below)            |
| `prometheus_exporter__version`                 | String                          | This variable has default value on [name of the exporter]+__version variable. It can be used as a placeholder in url (see example)|
| `prometheus_exporter__service_user`            | String                          | Specify the system user with which the exporter will be run, this value is auto generated if not filled                           |
| `prometheus_exporter__service_group`           | String                          | Specify the system group with which the exporter will be run, this value is auto generated if not filled                          |
| `prometheus_exporter__service_enabled`         | Boolean                         | Enable or not the systemd service                                                                                                 |
| `prometheus_exporter__service_cmdline`         | String/List of Strings          | The command line arguments that will be given to executable                                                                       |
| `prometheus_exporter__create_directories`      | List of directories definitions | Will create the given list of directories before to run the exporter (see below)                                                  |
| `prometheus_exporter__create_files`            | List of files definitions       | Will create the given list of file before to run the exporter (see below)                                                         |
| `prometheus_exporter__capabilities`            | List of String                  | List of capabilities to apply on exporter executable                                                                              |


### Installation and configuration

By default some exporters are known, so you have not to fill the download url, checksum url and others options.
But if you want to install your own exporter, you have juste to fill theses settings. Then the installation procedure must be "the same" for all exporter

#### Custom directories

The prometheus_exporter__create_directories variable allow custom directories creation before to start the exporter.
It take a list of directories definition, each definition call the official "file" module.

#### Custom files

The prometheus_exporter__create_files variable allow custom files creation before to start the exporter.
It need a list of files definitions. Each file definition accept common file attributes plus 'content' or 'template'.
If 'template' is used, the 'template' module will be call to produce the file's content. So encure that the template file is reacheable.
If 'content' is used, the 'copy' module will be call to directly produce the file's content.


### Known Exporters

#### Node exporter


| Name                                                      | Type                | Description                                                             |
| --------------------------------------------------------- | --------------------|-------------------------------------------------------------------------|
| `node_exporter__web_listen_address`                       | String              | The ipaddress + port on which the exporter will listen                  |
| `node_exporter__web_telemetry_path`                       | String              | The path prefix on which to serve the metrics                           |
| `node_exporter__enabled_collectors_global/group/host`     | List of String/Dict | Define collectors to enable with optional options (see below)           |
| `node_exporter__disabled_collectors_global/group/host`    | List of String      | Disable theses collectors (theses names must not be in the enabled list)|
| `node_exporter__textfile_directory`                       | String              | The path where the local disk statistics collector will read metrics    |

Some node-exporter collectors' accepts options. In the variables node_exporter__enabled_collectors_* you can pass simple string and dict

If you use simple string, the collector will simply be enabled.

If you specify a dict, the collector will be enabled and all dict key-value will be passed as collector options (see example with filesystem below)


#### Blackbox exporter

| Name                                    | Type         | Description                                                                                                                |
| --------------------------------------- | -------------|----------------------------------------------------------------------------------------------------------------------------|
| `blackbox_exporter__web_listen_address` | String       | The ipaddress + port on which the exporter will listen                                                                     |
| `blackbox_exporter__config`             | String/Mixed | The raw blackbox yaml configuration file. If this variable is a dict, it will be yamlized before being written to the file |

## Facts

By default the local fact are installed and expose the following variables :


```
ansible_local.[exporter name]:
    version_full: "1.0.0"
    version_major: "1"
```

## Example

### Playbook

* Install and configure a built-in exporter type as follows:

```yaml
- hosts: all
  roles:
    - role: turgon37.prometheus_exporter
      vars:
        prometheus_exporter__name: node_exporter
```

* Install and configure a non built-in exporter type as follows :

```yaml
- hosts: all
  roles:
    - role: turgon37.prometheus_exporter
      vars:
        prometheus_exporter__name: mysqld_exporter
        prometheus_exporter__download_url: 
          "https://github.com/prometheus/mysqld_exporter/releases/download/\
          v{{ prometheus_exporter__version }}/\
          mysqld_exporter-{{ prometheus_exporter__version }}.linux\
          -{{ prometheus_exporter__go_arch_map[ansible_architecture]|d(ansible_architecture) }}.tar.gz"
        mysqld_exporter__version: 0.11.0
        prometheus_exporter__temporary_file_extension: .tar.gz
        prometheus_exporter__service_cmdline: >-
          --config.my-cnf /etc/mysql/monitoring.cnf
```

### Node exporter

```
node_exporter__version: 0.17.0

# using a global exporters port map defined in your inventory to prevent overlap
node_exporter__web_listen_address: 127.0.0.1:{{ _prometheus_exporter_ports_map['node'] }}"
node_exporter__enabled_collectors_global:
  - ntp
  - filesystem:
      ignored-mount-points: "^/(sys|proc|dev)($|/)"
      ignored-fs-types: "^(sys|pr-c|auto)fs$"
```

### Blackbox exporter

```
blackbox_exporter__version: 0.14.0

prometheus_exporter__capabilities:
  - cap_net_raw+ep

blackbox_exporter__config:
  modules:
    http_2xx:
      prober: http
      timeout: 5s
      http:
        method: GET
        valid_status_codes: []
        no_follow_redirects: true
        fail_if_ssl: false
        fail_if_not_ssl: false
        preferred_ip_protocol: ip4
    icmp:
      prober: icmp
      icmp:
        preferred_ip_protocol: ip4
        dont_fragment: true
        payload_size: 500
```
