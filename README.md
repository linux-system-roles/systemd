# systemd

[![ansible-lint.yml](https://github.com/linux-system-roles/systemd/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/ansible-lint.yml) [![ansible-test.yml](https://github.com/linux-system-roles/systemd/actions/workflows/ansible-test.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/ansible-test.yml) [![codeql.yml](https://github.com/linux-system-roles/systemd/actions/workflows/codeql.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/codeql.yml) [![codespell.yml](https://github.com/linux-system-roles/systemd/actions/workflows/codespell.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/codespell.yml) [![markdownlint.yml](https://github.com/linux-system-roles/systemd/actions/workflows/markdownlint.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/markdownlint.yml) [![python-unit-test.yml](https://github.com/linux-system-roles/systemd/actions/workflows/python-unit-test.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/python-unit-test.yml) [![qemu-kvm-integration-tests.yml](https://github.com/linux-system-roles/systemd/actions/workflows/qemu-kvm-integration-tests.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/qemu-kvm-integration-tests.yml) [![shellcheck.yml](https://github.com/linux-system-roles/systemd/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/shellcheck.yml) [![tft.yml](https://github.com/linux-system-roles/systemd/actions/workflows/tft.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/tft.yml) [![tft_citest_bad.yml](https://github.com/linux-system-roles/systemd/actions/workflows/tft_citest_bad.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/tft_citest_bad.yml) [![woke.yml](https://github.com/linux-system-roles/systemd/actions/workflows/woke.yml/badge.svg)](https://github.com/linux-system-roles/systemd/actions/workflows/woke.yml)

![template](https://github.com/linux-system-roles/systemd/workflows/tox/badge.svg)

Ansible role that can be used to deploy unit files and manage systemd units. Role is a convenience
wrapper around systemd and template Ansible Core modules.

## Requirements

*NOTE:* Support for user units is not available in EL7 or earlier.  This feature
is only available in EL8 and later.

### Collection requirements

In order to manage `rpm-ostree` systems, the role requires modules from external
collections.  Use the following command to install them:

```bash
ansible-galaxy collection install -vv -r meta/collection-requirements.yml
```

## Role Variables

List of variables consumed by the role follows, note that none of them is mandatory.

Each of the variables can either be a list of strings, or a list of `dicts`.

The list of strings form assumes that the items to be managed are system units
owned by `root`, and for files, assumes that the files should be `present`.

The list of `dict` form looks like this:

```yaml
systemd_unit_files:
  - item: some.service
    user: my_user
    state: [present|absent]
```

Use the `dict` form to manage user units, and to remove unit files.  If using
user units, the role will manage lingering for those users.

*NOTE:* Support for user units is not available in EL7 or earlier.  This feature
is only available in EL8 and later.

### systemd_unit_files

List of systemd unit file names that should be deployed to managed nodes.

### systemd_unit_file_templates

List of systemd unit file names that should be deployed to managed nodes. Each name should
correspond to Jinja template file that will be templated out to managed nodes. If the local
file has a `.j2` suffix it will be stripped to form the service name.

### systemd_dropins

List of systemd drop in files that will be templated out to managed hosts and will extend
respective systemd unit files. Name of the unit file that given entry extends is encoded in
the name of the entry itself. For example, for entry `foo.service.conf` it is expected that
`foo.service.conf` Jinja template exists and resulting dropin file will extend `foo.service`
unit file. If the local file has a `.j2` suffix it will be stripped to form the service
name.

### systemd_started_units

List of unit names that shall be started via systemd.

### systemd_stopped_units

List of unit names that shall be stopped via systemd.

### systemd_restarted_units

List of unit names that shall be restarted via systemd.

### systemd_reloaded_units

List of unit names that shall be reloaded via systemd.

### systemd_enabled_units

List of unit files that shall be enabled via systemd.

### systemd_disabled_units

List of unit files that shall be disabled via systemd.

### systemd_masked_units

List of unit files that shall be masked via systemd.

### systemd_unmasked_units

List of unit files that shall be unmasked via systemd.

### systemd_transactional_update_reboot_ok

This variable is used to handle reboots required by transactional updates. If a transactional update requires a reboot, the role will proceed with the reboot if systemd_transactional_update_reboot_ok is set to true. If set to false, the role will notify the user that a reboot is required, allowing for custom handling of the reboot requirement. If this variable is not set, the role will fail to ensure the reboot requirement is not overlooked.

Example of setting the variables for the simple list of strings format:

```yaml
systemd_unit_files:
  - foo.service
  - bar.service
systemd_dropins:
  - cups.service.conf.j2
  - avahi-daemon.service.conf.j2
systemd_started_units:
  - foo.service
  - bar.service
systemd_enabled_units:
  - foo.service
  - bar.service
```

Example of setting the variables for the list of `dict` format:

```yaml
systemd_unit_files:
  - item: foo.service
    user: root
    state: present
  - item: bar.service
    user: my_user
    state: absent
systemd_dropins:
  - item: cups.service.conf.j2
    user: root
    state: present
  - item: avahi-daemon.service.conf.j2
    user: my_user
    state: absent
systemd_started_units:
  - item: foo.service
    user: root
  - item: bar.service
    user: my_user
systemd_enabled_units:
  - item: foo.service
    user: root
  - item: bar.service
    user: my_user
```

## Variables Exported by the Role

### `systemd_units`

The variable is a `dict`.  Each key is the name of a systemd unit.  Each value
is a dict with fields that describe the state of that systemd unit present on
the managed host for the system scope.

### `systemd_units_user`

Variable shall contain a dict.  Each key is the name of a user given in one of
the lists passed to the role, and `root` (even if `root` is not given).  Each
value is a dict of systemd units for that user, or system units for `root`, in
the format of `systemd_units` above.

## Example Playbook

```yaml
- name: Deploy and start systemd unit
  hosts: all
  vars:
    systemd_unit_file_templates:
      - foo.service.j2
    systemd_started_units:
      - item: foo.service
        user: root
      - item: bar.service
        user: my_user
    systemd_enabled_units:
      - foo.service
  roles:
    - linux-system-roles.systemd
```

## rpm-ostree

See README-ostree.md

## License

MIT

## Author

Michal Sekletar <msekleta@redhat.com>
Rich Megginson <rmeggins@redhat.com>
