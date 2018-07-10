# Ansible Playbooks for Dell VRTX Chassis setup

The following playbooks can be used to set up the CMC, Internal Switch and base config on the iDRAC cards in a [Dell VRTX Chassis](https://www.dell.com/en-us/work/learn/vrtx-server-solutions).
The tasks make extensive use of the "raw" ansible module, but still attempts to only make "changes" when they are needed.

## Setup

You MUST use the included ansible.cfg file.  There is a bug in the VRTX CMC that does not play well with the default Ansible options of ControlPersist.  In order to completely disable it, we need to pass in a blank ssh_args to ansible.  The easiest way to do this is to set the ANSIBLE_CONFIG environment variable to point to the config file BEFORE running the playbooks

``` bash
$ ANSIBLE_CONFIG=<path to cfg file>
$ export ANSIBLE_CONFIG
```

or run the included setup script

``` bash
$ source setup.sh
```

## Host file configuration

Use the included "inventory/hosts" file as an example for what needs to be defined.  At a minimum the following information is required:

One or more VRTX chassis deployments in the form of

```
[<vrtx group name>]
<chassis host name> ansible_host=<ip address of vrtx cmc> mgmt_netmask=<netmask for mgmt network> mgmt_gw=<gateway for mgmt network> chassis_name=<chassis name> chassis_location=<chassis location>
```

The following variables can be defined/used either in the all:vars section or over-ridden in the \<vrtx group name\>:vars section if you wish to override the all setting for a given group

| ansible variable            | required | description                                                                           |
| --------------------------- | -------- | ------------------------------------------------------------------------------------- |
| ansible_ssh_pass            | X        | password to connect to the vrtx cmc with                                              |
| ansible_ssh_user            | X        | username to connect to the vrtx cmc with                                              |
| dhcp_enable                 | X        | enable [1] or disable [0] DHCP for CMC                                                |
| ipv6_enable                 | X        | enable [1] or disable [0] IPv6 for CMC                                                |
| ntp_enable                  |          | enable [1] or disable [0] ntp for the chassis                                         |
| ntp_server1                 |          | if NTP enabled the first ntp server to connect to                                     |
| dns_from_dhcp_config        | X        | use DNS records from DHCP [1] or ignore dns records from DHCP [0]                     |
| dns_server_1                |          | if DNS from DHCP is disabled, DNS server 1 to use                                     |
| dns_server_2                |          | if DNS from DHCP is disabled, DNS server 2 to use                                     |
| dns_domain_name             |          | DNS domain name                                                                       |
| alerting_enable             | X        | enable [1] or disable [0] alerting for the chassis                                    |
| smtp_server                 |          | SMTP server to use for e-mail alerts                                                  |
| source_email_name           |          | email address alerts should come from                                                 |
| alert_email_target_1        |          | destination email address for alerts                                                  |
| alert_email_target_1_enable |          | enable [1] or disable [0] email alerts to email_target_1                              |
| snmp_enable                 |          | enable [1] or disable [0] snmp                                                        |
| snmp_community              |          | community string for the snmp agent                                                   |
| snmp_protocol               |          | configure default SNMP version ALL [0], or SNMPv3 [1]                                 |
| snmp_trap_format            |          | configure SNMP trap version SNMPv1 [0], SNMPv2 [1], or SNMPv3 [2]                     |
| snmp_trap_enable_1          |          | enable [1] or disable [0] sending of snmp traps 1                                     |
| snmp_trap_destination_1     |          | IP address of trap destination 1                                                      |
| snmp_trap_enable_2          |          | enable [1] or disable [0] sending of snmp traps 2                                     |
| snmp_trap_destination_2     |          | IP address of trap destination 2                                                      |
| switch1_ip                  |          | disabled DHCP for built in switch and assigns static IP to internal switch            |
| lcd_locale                  |          | set the locale for the chassis display. valid options are en, es, fr                  |
| lcd_orientation             |          | vertical [0] horizontal [1]                                                           |
| lcd_buttons_enable          |          | enable [1] or disable [0] ldc buttons on chassis                                      |
| slot_name_config            |          | manually set slot name [0], display host name from os [1], display iDRAC DNS name [2] |
| server1_name                |          | server name for slot 1, is slot_name_config is set to 0                               |
| server2_name                |          | server name for slot 2, is slot_name_config is set to 0                               |
| server3_name                |          | server name for slot 3, is slot_name_config is set to 0                               |
| server4_name                |          | server name for slot 4, is slot_name_config is set to 0                               |
| server1_idrac_ip            |          | IP address for slot1 iDRAC.  DHCP enabled if not defined                              |
| server2_idrac_ip            |          | IP address for slot2 iDRAC.  DHCP enabled if not defined                              |
| server3_idrac_ip            |          | IP address for slot3 iDRAC.  DHCP enabled if not defined                              |
| server4_idrac_ip            |          | IP address for slot4 iDRAC.  DHCP enabled if not defined                              |
| raid_1_config               |          | See RAID section below for more details                                               |


## Running the playbooks

Initial chassis setup can be done by editing the hosts file and then running

`ansible-playbook -i inventory/hosts playbooks/deploy_vrtx_chassis.yml`

Initial RAID setup can be done by running the following playbook.  See RAID section below before using this

`ansible-playbook -i inventory/hosts playbooks/configure_vrtx_raid.yml`

### R1_2104 Switch Configuration

Coming soon.


## RAID Configuration

The RAID configuration playbooks are **DANGEROUS**. Be sure you know what you are doing when you run them.  Every attempt has been made to ensure we dont over-write an existing RAID configuration.

RAID configuration takes a Dell racadm RAID config command.  see the References section below for the exact command options.

_Example_

```
raid_1_config="createvd:RAID.ChassisIntegrated.1-1 -rl r6 -pdkey:Disk.Bay.0:Enclosure.Internal.0-0:RAID.ChassisIntegrated.1-1,Disk.Bay.1:Enclosure.Internal.0-0:RAID.ChassisIntegrated.1-1,Disk.Bay.2:Enclosure.Internal.0-0:RAID.ChassisIntegrated.1-1,Disk.Bay.3:Enclosure.Internal.0-0:RAID.ChassisIntegrated.1-1,Disk.Bay.4:Enclosure.Internal.0-0:RAID.ChassisIntegrated.1-1"
```

will create a 5 disk RAID 6 virtual disk, using the first 5 disks in the system.

## References

[Dell CMC VRTX RACADM Guide](http://www.dell.com/support/manuals/us/en/04/poweredge-vrtx/cmcvrtx_3.0_racadm_guide/cfglcdinfo?guid=guid-22702c64-816f-46ba-9715-e71b1c79949d&lang=en-us)

## Copyright

Copyright 2018 Paychex Inc. All rights reserved.
Use of this source code is governed by the Apache 2.0
license that can be found in the LICENSE file.

## Author

This exporter was originally written by [Mark DeNeve](https://github.com/xphyr)