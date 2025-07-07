# Ansible-Role: AttackMate

This ansible role downloads [AttackMate](https://github.com/ait-aecid/attackmate) from the official repository
using git and installs all dependencies in a virtual environment.
It is further possible to roll out playbooks.

## Requirements

- Debian or Ubuntu

(Currently there are no packages defined for RedHat distributions)

## Role Variables

| Variable name                  | Type         | Default                                   | Description                                              |
| ------------------------------ | ------------ | ----------------------------------------- | -------------------------------------------------------- |
| attackmate_url                 | url          | https://github.com/ait-aecid/attackmate.git | Official attackmate repository |
| attackmate_version             | version-str  | main | Version/Branch of the Git-Repository in attackmate_url |
| attackmate_shared_dir          | path         | /usr/local/share | Installation path |
| attackmate_dest                | path         | `{{ attackmate_shared_dir }}/attackmate` | Installation path of the attackmate repository |
| attackmate_sliverfix           | bool         | True | [Install sliver-fix](https://aeciddocs.ait.ac.at/attackmate/development/installation/sliverfix.html#sliver-fix) |
| attackmate_grpc_dest           | path | `{{ attackmate_shared_dir }}/grpc` | Temporary install grpc to this path if sliverfix is enabled |
| attackmate_bindir              | path | /usr/local/bin | Installpath for the tmux-wrapper |
| attackmate_tmux                | bool | True | Deploy tmux-wrapper |
| attackmate_tmux_session        | str  | attackmate | Use this existing session-name for the tmux-wrapper |
| attackmate_tmux_window         | str  | attackmate | The name of the tmux-window for attackmate |
| attackmate_config_dir          | path | /etc/attackmate | Path to the config-directory |
| attackmate_playbook_path       | path | `{{ attackmate_config_dir }}/playbooks` | Path to the playbooks-directory |
| attackmate_playbooks           | list of playbook-templates(j2) | `[]` | List of playbooks to deploy |
| attackmate_config_tpl          | str  | attackmate.yml.j2 | Name of the config-template(jinja) |
| attackmate_sliver_config       | path | **None** | Path to the generated sliver-config. (only needed for sliver-commands) |
| attackmate_msf_server          | hostname | **None** | Hostname of the Metasploit rpcd. (only needed for msf-commands) |
| attackmate_msf_passwd          | password | **None** | Password for the Metasploit rpcd. (only needed for msf-commands) |
| command_delay                  | float | **None** | delay in seconds before commands for the CommandConfig |

## Example Playbook

```yaml
- name: Install attackmate
  become: true
  hosts: localhost
  roles:
    - role: attackmate
      vars:
        attackmate_sliverfix: True
        attackmate_version: development
        attackmate_msf_server: localhost
        attackmate_msf_passwd: hackerman
        attackmate_playbooks:
          - upgradeshell.j2
          - attackchain.j2
        command_delay: 2
```

This role installs to executables:

* **/usr/local/bin/attackm8**: a wrapper for attackmate that uses the virtual environment
* **/usr/local/bin/attackmate-tmux**: a wrapper that executes attackmate in a tmux-session

## Role testing with molecule

### Role testing locally

If you want to test this role locally using Molecule (https://ansible.readthedocs.io/projects/molecule/) and Docker we provided a Molecule configuration.

Requirements to run Molecule:

1. Install docker: https://docs.docker.com/engine/install/ubuntu/ (Make sure you have permission to run Docker commands (i.e., your user is in the docker group))
2. Install uv:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv venv
   ```
3. Install molecule and ansible:
   ```bash
   uv pip install molecule ansible molecule-docker
   ```
4. run 
   ```bash
   uv run molecule test

   ```
### Role testing with github action

Additionally a github action for molecule tests is being triggered on push and pull requests.

## License

GPL-3.0

## Author

- Wolfgang Hotwagner
