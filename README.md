Ansible Role Prometheus Exporter
========

[![Build Status](https://travis-ci.org/Turgon37/ansible-prometheus-exporter.svg?branch=master)](https://travis-ci.org/Turgon37/ansible-prometheus-exporter)
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

## Configuration

### Role

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below. To see default values please refer to this file.

| Name                          | Description       |
| ----------------------------- | ------------------|
| `prometheus_exporter__name`   | The exporter name |


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
