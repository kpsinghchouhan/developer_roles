# kpsinghchouhan.developer_roles.gh

Installs [GitHub CLI](https://cli.github.com/) (`gh`).

## Supported platforms

| Platform | Package installation                                                                  |
| -------- | ------------------------------------------------------------------------------------- |
| Ubuntu   | `ansible.builtin.apt` from the GitHub CLI apt repository (needs `become`)             |
| macOS    | `community.general.homebrew` (runs as user)                                           |

On Ubuntu the role adds GitHub's archive keyring as `/etc/apt/keyrings/github-cli.gpg` and the
repository as `/etc/apt/sources.list.d/github-cli.sources`, with `ansible.builtin.deb822_repository`.
Ubuntu's own `gh` package is older than the GitHub release.

## Requirements

- `community.general` collection (declared as a collection dependency), for Homebrew on macOS.
- Homebrew already installed on macOS hosts.
- Privilege escalation on Ubuntu for the repository and package install.
- Internet access to `cli.github.com` on Ubuntu hosts.
- Facts gathered for the play (`gather_facts: true`, the default). The role uses `os_family`.

## Role Variables

| Variable              | Default                                                           | Description                                                          |
| --------------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------- |
| `gh_package_state`    | `"present"`                                                       | `present` or `latest`. Use `latest` to upgrade an existing gh.       |
| `gh_apt_keyring_url`  | `"https://cli.github.com/packages/githubcli-archive-keyring.gpg"` | URL of the apt archive keyring. Ubuntu only.                         |
| `gh_apt_repo_url`     | `"https://cli.github.com/packages"`                               | URL of the apt repository. Ubuntu only.                              |
| `gh_apt_repo_name`    | `"github-cli"`                                                    | Name of the apt source and keyring files. Ubuntu only.               |
| `gh_apt_repo_channel` | `"stable"`                                                        | Suite of the apt repository. Ubuntu only.                            |

All variables are validated by `meta/argument_specs.yml`.

If `gh` is already installed from Ubuntu's repository, `present` leaves that
version in place. Set `gh_package_state: latest` to upgrade it from the GitHub repository.

## Dependencies

None.

## Example Playbook

```yaml
- name: Install developer tools
  hosts: workstations
  roles:
    - role: kpsinghchouhan.developer_roles.gh
    - role: kpsinghchouhan.developer_roles.git
      git_user_name: "Jane Doe"
      git_user_email: "jane@example.com"
```

The role does not log in to GitHub. Run `gh auth login` once on the host. The
`git` role's default credential helper (`gh auth git-credential`) then uses that login.

## Role Idempotency

True. A second run with the same variables reports no changes, unless
`gh_package_state: latest` and a newer release is available. Check mode is supported.

## Role Atomicity

False. On Ubuntu the keyring, repository and package are separate steps. A failed run can
leave the repository configured without the package installed; re-running the role completes it.

## Roll-back capabilities

None. To remove the install on Ubuntu, remove the `gh` package,
`/etc/apt/sources.list.d/github-cli.sources` and `/etc/apt/keyrings/github-cli.gpg`. On macOS, run
`brew uninstall gh`.

## License

GPL-2.0-or-later

## Author Information

Krishana Pal Singh Chouhan
