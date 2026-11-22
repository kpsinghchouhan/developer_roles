# kpsinghchouhan.developer_roles.starship

Installs the [starship](https://starship.rs) prompt, copies its config file to
`~/.config/starship.toml` and enables it in the rc file of the user's login shell (bash or zsh).

## Supported platforms

| Platform | Package installation                                                                       |
| -------- | ------------------------------------------------------------------------------------------ |
| Ubuntu   | `ansible.builtin.apt` (needs `become`)                                                     |
| macOS    | `community.general.homebrew` (runs as user)                                                |

On Ubuntu the role installs the `starship` package from the distribution repositories. Ubuntu
24.04 and older have no starship package, so the role needs Ubuntu 25.04 or later.

## Requirements

- `community.general` collection (declared as a collection dependency), for Homebrew on macOS.
- Homebrew already installed on macOS hosts.
- Privilege escalation on Ubuntu for the APT install.
- Facts gathered for the play (`gather_facts: true`, the default). The role uses `os_family`,
  `user_id`, `user_gid`, `user_shell` and `env`.
- A [Nerd Font](https://www.nerdfonts.com/) in the terminal if your config uses Nerd Font symbols.
  The default config does not.

## Role Variables

| Variable                 | Default                         | Description                                                                         |
| ------------------------ | ------------------------------- | ----------------------------------------------------------------------------------- |
| `starship_package_state` | `"present"`                     | `present` or `latest`. Use `latest` to upgrade an existing starship.                |
| `starship_config_src`    | `"starship.toml"`               | Source of `starship.toml`: a file in the role's `files/`, or a path on the controller. |
| `starship_config_home`   | `ansible_facts['env']['HOME']`  | Home directory that gets `.config/starship.toml` and the rc file.                   |
| `starship_config_mode`   | `"0644"`                        | File mode of `starship.toml`.                                                       |
| `starship_shell`         | login shell of the user         | Shell whose rc file gets `eval "$(starship init <shell>)"`.                         |

All variables are validated by `meta/argument_specs.yml`.

## Config file

The role owns `~/.config/starship.toml` and replaces it on every run. The default is a short config
in [files/starship.toml](files/starship.toml). To use your own, keep it next to your playbook:

```yaml
starship_config_src: "{{ playbook_dir }}/files/starship.toml"
```

The role adds one line to the rc file of the connecting user's login shell (`~/.bashrc` or
`~/.zshrc`, from the `user_shell` fact), and creates the file if it is missing. The rest of the
file is left alone. Set `starship_shell` to target a different shell.

## Dependencies

None.

## Example Playbook

```yaml
- name: Install developer tools
  hosts: workstations
  roles:
    - role: kpsinghchouhan.developer_roles.gh
    - role: kpsinghchouhan.developer_roles.starship
      starship_config_src: "{{ playbook_dir }}/files/starship.toml"
      starship_shell: zsh
```

Open a new shell after the role runs to see the prompt.

## Role Idempotency

True. A second run with the same variables reports no changes, unless
`starship_package_state: latest` and a newer package is available. Check mode is supported.

## Role Atomicity

False. The package, config file and rc file are separate steps. A failed run can leave some of
them in place; re-running the role completes it.

## Roll-back capabilities

None. To remove starship on Ubuntu, run `sudo apt remove starship`. On macOS, run
`brew uninstall starship`. Then delete `~/.config/starship.toml` and the `starship init` lines in
your shell rc file.

## License

GPL-2.0-or-later

## Author Information

Krishana Pal Singh Chouhan
