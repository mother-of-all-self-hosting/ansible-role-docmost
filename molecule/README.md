<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard Docmost installation, against an image pulled from upstream.

A freshly installed Docmost serves its frontend with `200` whether or not anybody has completed its first-run setup, so the verification starts from a negative control which asserts exactly that: `/` answers `200` while every request that reaches the database answers `Workspace not found`, Postgres holds no workspace row, and Valkey holds no session. It then has to move each of those off its starting value:

- the first-run workspace setup is completed over the API, and the endpoint that answered `404` starts returning the new workspace
- logging in with the stored password returns a token, which the version endpoint refuses to answer without
- the running Docmost reports the version that `docmost_version` names
- a page is created over the API, read back with its body, and then read straight out of Postgres
- logging in leaves session state in Valkey, alongside the job queues Docmost runs there
- a file uploaded over the API is readable on the host underneath the role's data path, which is what shows that the storage bind mount is the one Docmost writes to

### `default-selfbuild`

Tests a standard Docmost installation with self-building the container image.

Since what is different here is where the image came from, this is what its verification concentrates on: the clone the role made sits on the tag matching `docmost_version`, the service runs the image built out of it, and that image carries no registry digest — which is what tells a genuine build apart from a pull that happened to be tagged the same way. It then completes the first-run setup, logs in, checks the version the built image reports, and round-trips a page, to show that the image it built runs. Everything that does not depend on how the image was produced is left to the `default` scenario.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
