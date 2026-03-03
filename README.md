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
| attackmate_url                 | url          | https://github.com/ait-aecid/attackmate.git | Official attackmate repository                         |
| attackmate_version             | version-str  | main | Version/Branch of the Git-Repository in attackmate_url                                        |
| attackmate_shared_dir          | path         | /usr/local/share                          | Installation path                                        |
| attackmate_dest                | path         | `{{ attackmate_shared_dir }}/attackmate`  | Installation path of the attackmate repository           |
| attackmate_sliverfix           | bool         | True                                      | [Install sliver-fix](https://aeciddocs.ait.ac.at/attackmate/development/installation/sliverfix.html#sliver-fix) |
| attackmate_grpc_dest           | path         | `{{ attackmate_shared_dir }}/grpc`        | Temporary install grpc to this path if sliverfix is enabled |
| attackmate_bindir              | path         | /usr/local/bin                            | Installpath for the tmux-wrapper                         |
| attackmate_tmux                | bool         | True                                      | Deploy tmux-wrapper                                      |
| attackmate_tmux_session        | str          | attackmate                                | Use this existing session-name for the tmux-wrapper      |
| attackmate_tmux_window         | str          | attackmate                                | The name of the tmux-window for attackmate               |
| attackmate_config_dir          | path         | /etc/attackmate                           | Path to the config-directory                             |
| attackmate_playbook_path       | path         | `{{ attackmate_config_dir }}/playbooks`   | Path to the playbooks-directory                          |
| attackmate_playbooks           | list of playbook-templates(j2) | `[]`                    | List of playbooks to deploy                              |
| attackmate_config_tpl          | str          | attackmate.yml.j2 | Name of the config-template(jinja) |
| attackmate_sliver_config       | path         | **None**                                  | Path to the generated sliver-config. (only needed for sliver-commands) |
| attackmate_msf_server          | hostname     | **None**                                  | Hostname of the Metasploit rpcd. (only needed for msf-commands) |
| attackmate_msf_passwd          | password     | **None**                                  | Password for the Metasploit rpcd. (only needed for msf-commands) |
| attackmate_playwright          | bool         | True                                      | Whether to install Playwright and its dependencies |
| command_delay                  | float        | **None**                                  | delay in seconds before commands for the CommandConfig |
| attackmate_remote_config       | dict         | {} | Optional map of named remote AttackMate connections. Each entry requires url, username, password, and optionally cafile. If empty, no remote_config section is written to the config file.|

## Additional role Variables for installation as Api server 
| Variable name                  | Type         | Default                                   | Description                                              |
| ------------------------------ | ------------ | ----------------------------------------- | -------------------------------------------------------- |
| attackmate_api_server          | bool         | False                                     | Install the attackmate-api-server |
| attackmate_api_server_url      | url          | https://github.com/ait-testbed/attackmate-api-server.git | Repository URL for the api server |
| attackmate_api_server_version  | version-str  | main                                      | Version/Branch of the api server repository |
| attackmate_api_server_dest     | path         | {{ attackmate_shared_dir }}/attackmate-api-server | Installation path of the api server |
| attackmate_api_bin_path        | path         | /usr/local/bin/attackmate-api | Installation path for the attackmate-api executable |
| attackmate_api_service_path    | path         | /etc/systemd/system/attackmate-api.service | Path for the systemd service unit file |
| attackmate_api_log_dir         | path         | /var/log/attackmate-api | Directory for API server log files |
| attackmate_api_logs_to_disk    | bool         | False | Whether to write playbook logs to disk |
| attackmate_ssl_key_path        | path         | /etc/ssl/private/attackmate.key | Path for the generated RSA private key |
| attackmate_ssl_cert_path       | path         | /etc/ssl/certs/attackmate.pem | Path for the generated self-signed certificate |
| attackmate_api_plain_ users    | dict         | {} | Map of username to plaintext password used to generate argon2 hashes at deploy time. Format: {"username": "password", ...}. Leave empty to skip user generation. Never commit plaintext passwords! |


> [!WARNING]
> `attackmate_api_plain_users` contains plaintext passwords and must **never** be committed to version control.
> Only define this variable locally on the machine you are running the playbook from, either by passing it via
> `--extra-vars` at runtime or in a local vars file that is excluded from your repository via `.gitignore`.
> The hashing and deployment happen in memory only — the plaintext passwords are never written to the target host.

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
        attackmate_remote_config:
          primary_node:
            url: "https://10.0.0.5:5000"
            username: admin
            password: securepassword
            cafile: "/path/to/cert.pem"
```

This role installs to executables:

* **/usr/local/bin/attackm8**: a wrapper for attackmate that uses the virtual environment
* **/usr/local/bin/attackmate-tmux**: a wrapper that executes attackmate in a tmux-session

## Installing as API Server

AttackMate can optionally be installed together with the [AttackMate API Server](https://github.com/ait-testbed/attackmate-api-server),
which exposes AttackMate's functionality via a REST API and allows remote instances to be controlled over the network.
The API server is installed into the same virtual environment as AttackMate, since it depends on it.

To enable the API server, set `attackmate_api_server: True` in your playbook:
```yaml
- name: Install attackmate with API server
  become: true
  hosts: localhost
  roles:
    - role: attackmate
      vars:
        attackmate_api_server: True
        attackmate_api_plain_users:
          admin: "securepassword"

```

 installs to executables:

* **/usr/local/bin/attackmate-api-server**: symlink to the attackmate-api-server executable in the virtual environment

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
