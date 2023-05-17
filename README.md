
# Ansible role for Restic

[[_TOC_]]

Installs and configure Restic. 

This role offers two ways to install Restic:

* From Binary (default, you can specify which version you want to grab)
* From Repo (Debian/Ubuntu and RHEL/Rocky only)

It also provides two methods to schedule backups :

* Crons (default)
* Systemd timers

## Example playbook

```yml
- name: Converge
  hosts: all
  vars:
    restic_backup_keep:
      hourly: 1
      daily: 1
      weekly: 7
      monthly: 30
      yearly: 60
    restic_backup_filestobackup:
      - "/root/"
      - "/etc/"
    restic_backup_config:
      - RESTIC_REPOSITORY: "/backup"
      - RESTIC_PASSWORD: 'superStrongPassword'
  roles:
    - ansible-role-restic
```

You can use the `restic_repo_init` tag to init the repository.

## Role Variables

### Required variables

* To enable cron (and disable systemd timers), specify : `restic_backup_cron: true`
* To install restic from binary instead of repo, specify : `restic_install_binary`

| Variable                               | Type   | Description                                                                                                                                                                                                                                        |
|----------------------------------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| restic_backup_keep                     | dictionary | Snapshot removal policy as described in the [Restic documentation](https://restic.readthedocs.io/en/stable/060_forget.html#removing-snapshots-according-to-a-policy). `--keep-daily=1` becomes a dictionary entry with the key being `daily` and the value `1` (see example above).|
| restic_backup_filestobackup            | list   | List of path to files/dirs you want to backup                                                                                                                                                                                                      |
| restic_backup_config                   | list   | List containing env variables which should be used for restic. For example, if you are using S3, you should specify related vars here. [Restic documentation here](https://restic.readthedocs.io/en/latest/040_backup.html#environment-variables). |
| restic_backup_config.RESTIC_REPOSITORY | string | Restic repository                                                                                                                                                                                                                                  |
| restic_backup_config.RESTIC_PASSWORD   | string | Resitc password                                                                                                                                                                                                                                    |

### Defaults variables

#### Install variables

| Variable                                | Type   | Default        | Description                                                  |
|-----------------------------------------|--------|----------------|--------------------------------------------------------------|
| restic_install_repo                     | bool   | true           | Install restic from repo (Debian/Ubuntu and RHEL/Rocky only) |
| restic_install_binary                   | bool   | false          | Install Restic from a binary instead of the repo             |
| restic_install_binary_version           | string | 0.14.0         | **Binary Install Only** - Specify which version to install   |
| restic_install_binary_location          | string | /usr/local/bin | Path to the installed binary                                 |
| restic_install_binary_architecture      | string | amd64          | Specify which architecture from the binary                   |
| restic_install_binary_perms             | string | 0700           | Permissions of the restic binary                             |
| restic_install_binary_download_location | string | /tmp           | Path where the temporary archive will be extracted           |

#### Restic user related vars

| Variable                | Type   | Default           | Description                                |
|-------------------------|--------|-------------------|--------------------------------------------|
| restic_user_create      | bool   | true              | Create restic user                         |
| restic_user             | string | restic            | Name of the restic user                    |
| restic_user_group       | string | restic            | Restic user primary group                  |
| restic_user_groups      | list   |                   | List of groups user will be added to       |
| restic_user_shell       | string | /usr/sbin/nologin | Full path to the shell used by the user    |
| restic_user_home        | string | /nonexistent      | Full path to the user's home directory |
| restic_user_create_home | bool   | false             | Create home if it does not exist           |
| restic_user_system      | bool   | true              | Specify if user should be a system user    |

#### Restic configuration variables

| Variable                                | Type   | Default                              | Description                                                   |
|-----------------------------------------|--------|--------------------------------------|---------------------------------------------------------------|
| restic_backup_configs_location          | string | /var/restic/                         | Path to restic cache dir                                      |
| restic_backup_configs_location_perms    | int    | 0700                                 | Perms of the restic cache dir                                 |
| restic_backup_sh_location               | string | /bin/sh                              | Path to the shell used to start restic                        |
| restic_backup_name                      | string | restic_backup                        | Name of the Restic backup config                              |
| restic_backup_filestobackup_dst         | string | /var/restic/{{ restic_backup_name }} | Path to the file containing the list of dirs/files to back up |
| restic_backup_configs_perms             | int    | 0600                                 | Perms of the file containing the list of dirs/files to backup |
| restic_backup_config                    | list   |                                      | List of all environment variables used by Restic              |
| restic_backup_config.RESTIC_COMPRESSION | string | auto                                 | Enable/Disable Restic compression                             |
| restic_backup_config.RESTIC_CACHE_DIR   | string | /var/restic/cache                    | Path to the Restic cache dir                                  |
| restic_backup_config.TMPDIR             | string | /tmp                                 | Path to a tmp filesystem which Restic can use                 |

#### Cron variables

| Variable                                 | Type   | Default                                    | Description                                                            |
|------------------------------------------|--------|--------------------------------------------|------------------------------------------------------------------------|
| restic_backup_cron                       | bool   | false                                      | Enable/Disable cron backup                                             |
| restic_backup_cron_plan                  | list   |                                            | List of cron planning                                                  |
| restic_backup_cron_plan.minute           | string | 0                                          |                                                                        |
| restic_backup_cron_plan.hour             | string | 0                                          |                                                                        |
| restic_backup_cron_plan.day              | string | *                                          |                                                                        |
| restic_backup_cron_plan.month            | string | *                                          |                                                                        |
| restic_backup_cron_plan.weekday          | string | *                                          |                                                                        |
| restic_backup_cron_script_location       | string | /usr/local/bin/{{ restic_backup_name }}.sh | Path to the script executed by cron                                    |
| restic_backup_cron_script_location_perms | int    | 0700                                       | Permissions of the script (beware, creds are stored inside the script) |

#### Systemd variables

| Variable                              | Type   | Default                                                               | Description                           |
|---------------------------------------|--------|-----------------------------------------------------------------------|---------------------------------------|
| restic_backup_systemd                 | bool   | true                                                                  | Enable/Disable systemd service/timer  |
| restic_backup_systemd_location        | string | /etc/systemd/system                                                   | Path to the systemd config directory  |
| restic_backup_systemd_timer_plan      | string | daily                                                                 | Systemd timer plan                    |
| restic_backup_systemd_config_location | string | {{ restic_backup_systemd_service_location }}/{{ restic_backup_name }} | Path to the systemd config for restic |

## Molecule testing

The role contains a default molecule scenario. The scenario deploy the role on different platforms :

| OS        | Installation method | Scheduling method |
|-----------|---------------------|-------------------|
| Debian 11 | Binary              | Cron              |
| Debian 11 | Repo                | Systemd           |
| Rocky 9   | Binary              | Systemd           |
| Rocky 9   | Repo                | Cron              |

 It is integrated in the CI/CD with the `restic_repo_init` tag.


