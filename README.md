[![ansible-lint](https://github.com/sscheib/ansible-role-satellite_prepare_installation/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/sscheib/ansible-role-satellite_prepare_installation/actions/workflows/ansible-lint.yml) [![Publish latest release to Ansible Galaxy](https://github.com/sscheib/ansible-role-satellite_prepare_installation/actions/workflows/ansible-galaxy.yml/badge.svg)](https://github.com/sscheib/ansible-role-satellite_prepare_installation/actions/workflows/ansible-galaxy.yml)

satellite_prepare_installation
=========

This role performs the prerequisite steps to install a Red Hat Satellite server:
1. Open ports/services in the firewall
2. Install and enable chronyd
3. Optionally install the sos package
4. Enable the Satellite dnf module
5. Update all packages
6. Install the Satellite RPM
7. Optionally reboot the server

The steps are taken from the [Satellite documentation for installing Satellite](https://access.redhat.com/documentation/de-de/red_hat_satellite/6.14/html/installing_satellite_server_in_a_connected_network_environment/installing_server_connected_satellite).

After executing this role you can run the role [`redhat.satellite_operations.installer`](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/satellite_operations/content/role/installer/).

To use the certified collection `redhat.satellite` (for the role `redhat.satellite_operations.installer`) you need to be a Red Hat subscriber. If you don't own any
subscriptions, you can make use of [Red Hat's Developer Subscription](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux) which
is provided at no cost by Red Hat.

**Prior to running this role**, the system needs to be registered to the Red Hat Customer Portal and the appropriate repositories have to be enabled. For this task you can use the Red Hat system role
`redhat.rhel_system_roles.rhc`.

To get you started you can use the following *example variables definition* for a Red Hat Satellite 6.14 when using the role `redhat.rhel_system_roles.rhc`:

```
rhc_state: 'present'
rhc_auth:
  login:
    username: <RHSM_USERNAME>
    password: <RHSM_PASSWORD>

# Repositories to enable on the Satellite server (the Linux host, not the application)
rhc_repositories:
  - name: 'rhel-8-for-x86_64-baseos-rpms'
    state: 'enabled'

  - name: 'rhel-8-for-x86_64-appstream-rpms'
    state: 'enabled'

  - name: 'satellite-maintenance-6.14-for-rhel-8-x86_64-rpms'
    state: 'enabled'

  - name: 'satellite-6.14-for-rhel-8-x86_64-rpms'
    state: 'enabled'

# RHEL release to set in subscription-manager
rhc_release: 8

# We don't want to register to Red Hat Insights
rhc_insights:
  state: 'absent'
  autoupdate: false
  remediation: 'absent'
```

Requirements
------------

This role needs the system to be registered to the Red Hat Customer Portal with enabled Red Hat Satellite repositories in order to to install necessary packages.

Role Variables
--------------
| variable                                     | default                      | required | description                                                                    |
| :---------------------------------           | :--------------------------- | :------- | :----------------------------------------------------------------------------- |
| `sat_firewalld_ports`                        | check `defaults/main.yml`    | false    | `firewalld` ports to open on the Satellite                                     |
| `sat_firewalld_services`                     | check `defaults/main.yml`    | false    | `firewalld` services to open on the Satellite                                  |
| `sat_chrony_package_name`                    | `chrony`                     | false    | Name of the chrony package                                                     |
| `sat_chrony_systemd_service_name`            | `chronyd`                    | false    | Name of the chrony systemd service                                             |
| `sat_install_sos_package`                    | `true`                       | false    | Whether to install the sos package                                             |
| `sat_sos_package_name`                       | `sos`                        | false    | Name of the sos package                                                        |
| `sat_dnf_module_regexp`                      | `^satellite$`                | false    | Regular expression to search for the Satellite dnf module                      |
| `sat_dnf_module_name`                        | `satellite:el8`              | false    | Satellite dnf module name                                                      |
| `sat_update_all_packages`                    | `true`                       | false    | Whether to update all packages prior to installing the Satellite package       |
| `sat_package_name`                           | `satellite`                  | false    | Name of the Satellite package to install                                       |
| `sat_reboot_satellite_after_update`          | `true`                       | false    | Whether to reboot the Satellite after packages where updated                   |
| `sat_reboot_timeout`                         | `1200`                       | false    | Maximum seconds to wait for the host to come up again after rebooting          |
| `sat_post_reboot_delay`                      | `30`                         | false    | Seconds to wait after the reboot command before validating the system reboot   |
| `sat_quiet_assert`                           | `false`                      | false    | Whether to quiet assert                                                        |

## Variable `sat_firewalld_ports`

An example of the `sat_firewalld_ports` variable is illustrated down below (which is in fact the default value):
```
sat_firewalld_ports:
  - port: '1883/tcp'
    zone: 'public'

  - port: '5646-5647/tcp'
    zone: 'public'

  - port: '5910-5930/tcp'
    zone: 'public'

  - port: '8000/tcp'
    zone: 'public'

  - port: '8140/tcp'
    zone: 'public'

  - port: '9090/tcp'
    zone: 'public'
```

Please note:
- The ports are taken from the [Satellite documentation](https://access.redhat.com/documentation/de-de/red_hat_satellite/6.14/html/installing_satellite_server_in_a_connected_network_environment/preparing_your_environment_for_installation_satellite#supported-browsers_satellite)
- Both `port` and `zone` are required attributes for each list element
- The format the ports can be specified in is either PORT/PROTOCOL or for a range of ports PORT-PORT/PROTOCOL
- The ports are passed to the `ansible.posix.firewalld` module

## Variable `sat_firewalld_services`

An example of the `sat_firewalld_services` variable is illustrated down below (which is in fact the default value):
```
sat_firewalld_services:
  - name: 'dns'
    zone: 'public'

  - name: 'dhcp'
    zone: 'public'

  - name: 'tftp'
    zone: 'public'

  - name: 'http'
    zone: 'public'

  - name: 'https'
    zone: 'public'
```

Please note:
- The services are derived from the ports specified in the [Satellite documentation](https://access.redhat.com/documentation/de-de/red_hat_satellite/6.14/html/installing_satellite_server_in_a_connected_network_environment/preparing_your_environment_for_installation_satellite#supported-browsers_satellite)
- Both `name` and `zone` are required attributes for each list element
- The services are passed to the `ansible.posix.firewalld` module, so the services *need to exist* prior of using them (the above services exist already by default)

Dependencies
------------

This role makes use of the Ansible collection [`ansible.posix`](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html), which is specified via
`collections/requirements.yml`.

Example Playbook
----------------

```
---
- name: 'Ensure prerequisites are met for Satellite to be installed'
  hosts: 'all'
  gather_facts: false
  roles:
    - role: 'satellite_prepare_installation'
      vars:
        sat_firewalld_ports:
          - port: '1883/tcp'
            zone: 'public'

          - port: '5646-5647/tcp'
            zone: 'public'

          - port: '5910-5930/tcp'
            zone: 'public'

          - port: '8000/tcp'
            zone: 'public'

          - port: '8140/tcp'
            zone: 'public'

          - port: '9090/tcp'
            zone: 'public'

        sat_firewalld_services:
          - name: 'dns'
            zone: 'public'

          - name: 'dhcp'
            zone: 'public'

          - name: 'tftp'
            zone: 'public'

          - name: 'http'
            zone: 'public'

          - name: 'https'
            zone: 'public'

        sat_chrony_package_name: 'chrony'
        sat_chrony_systemd_service_name: 'chronyd'
        sat_install_sos_package: true
        sat_sos_package_name: 'sos'
        sat_dnf_module_regexp: '^satellite$'
        sat_dnf_module_name: 'satellite:el8'
        sat_update_all_packages: true
        sat_package_name: 'satellite'
        sat_reboot_satellite_after_update: true
        sat_reboot_timeout: 1200
        sat_quiet_assert: false
...
```

License
-------

GPL-2.0-or-later
