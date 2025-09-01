# docker-configs

Various docker configuration files.

> [!NOTE]
> Set each submodule's branch with: `git submodule set-branch --branch <branch> ./path/to/submodule` (don't include a trailing slash `/` in the path)
>
> Pull the latest changes for each submodule with: `git submodule foreach 'git pull'`

This project is similar to [straysheep-dev/packer-configs](https://github.com/straysheep-dev/packer-configs). The primary use-case is for CI/CD with [molecule](https://ansible.readthedocs.io/projects/molecule/getting-started/) to automate testing of Ansible roles across multiple operating systems from one development machine.

`systemd`-enabled containers are supported here across the major distros (and where possible). See the sources in the [references](#references) below for more details and examples of this.


## References

This project was built from combining and working with the following sources:

- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- [geerlingguy/ansible-role-docker: molecule.yml](https://github.com/geerlingguy/ansible-role-docker/blob/master/molecule/default/molecule.yml)
- [Docker Hub: Rocky Linux systemd Example](https://hub.docker.com/r/rockylinux/rockylinux)
- [Ansible Docs: molecule.yml](https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml)
- [Ansible Docs: molecule/guides/systemd-container](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container)
- [geerlingguy/docker-fedora42-ansible](https://github.com/geerlingguy/docker-fedora42-ansible/blob/master/Dockerfile)
- [geerlingguy/docker-ubuntu2404-ansible](https://github.com/geerlingguy/docker-ubuntu2404-ansible/blob/master/Dockerfile)
- [geerlingguy/docker-debian12-ansible](https://github.com/geerlingguy/docker-debian12-ansible/blob/master/Dockerfile)

Additional notes and usage information can be found on my blog:

- [straysheep.dev: Ansible](https://straysheep.dev/blog/2023/08/20/simple-ansible-ansible/)
- [straysheep.dev: Ansible Molecule](https://straysheep.dev/blog/2025/07/25/simple-ansible-ansible-molecule/)


## Molecule Usage

You can add as many `-name: <name>` sections under `platforms:` for as many OS's you'd like to test on as needed.

> [!NOTE]
> With `pre_build_image: false` this will always build the image locally using the specified Dockerfile.

> [!WARNING]
> `systemd`-enabled containers running with `cgroupns_mode: host` and `privileged: true` ***can compromise the host***. Even if you trust what the container is running it's best to use a development environment (VM) for building and testing.

An example [molecule.yml file](https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml):

```yml
---
# molecule/default/molecule.yml
# SPDX-License-Identifier: MIT
#
# The molecule.yml file configures Molecule itself.
#
# Built from the following sources:
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/molecule/default/molecule.yml
# - https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml
# - https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container

role_name_check: 1
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: <system>-latest
    image: <system>:latest
    command: /sbin/init  # It's critical to include the command here so systemd is PID 1
    pre_build_image: false
    cgroupns_mode: host
    privileged: true
    tmpfs:
      - /run
      - /tmp
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw
    dockerfile: <system>.Dockerfile
provisioner:
  name: ansible
  playbooks:
    converge: converge.yml
verifier:
  name: ansible
```


## License

Unless otherwise noted in a submodule or SPDX license identifier, most files in each submodule are licensed under the [MIT License](LICENSE) by default.