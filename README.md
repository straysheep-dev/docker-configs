# docker-configs

Various docker configuration files.

> [!NOTE]
> For managing submodules, see [straysheep.dev/blog/resources/#git](https://straysheep.dev/blog/2019/07/15/-resources/#git)

This project is similar to [straysheep-dev/packer-configs](https://github.com/straysheep-dev/packer-configs). The primary use-case is for CI/CD with [molecule](https://ansible.readthedocs.io/projects/molecule/getting-started/) to automate testing of Ansible roles across multiple operating systems from one development machine.

`systemd`-enabled containers are supported here across the major distros (and where possible). See the sources in the [references](#references) below for more details and examples of this.


## Usage

**Building Images with Dockerfiles**

See [build, tag, and publish an image](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/).

```bash
# Tag the file with any name you want, and point to the dockerfile with -f
# This also assumes you're in the cwd of the dockerfile with the "." on the end
docker build -t local/my-image-name -f ./some.Dockerfile .
```

**Interactively Running Images**

If you just want to jump into a standard image pulled from Docker Hub, or one you've built:

> [!TIP]
> See [Using Kali Linux Docker Images](https://www.kali.org/docs/containers/using-kali-docker-images/).

```bash
# Example
docker run --tty --interactive <tag/image-name>

# Download and run Kali
docker run --tty --interactive kalilinux/kali-rolling

# Download and run Fedora
docker run --tty --interactive fedora:latest

```

If the image you built uses systemd, you need to start it with `systemd` executed in the background first. The arguments required are the [same that you'd use for running molecule containers with systemd support](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container). You can see an example of this in [geerlingguy's build.yml using GitHub actions to build and test Docker containers](https://github.com/geerlingguy/docker-rockylinux9-ansible/blob/4c366b8059f5bf993e30b7d38da37b9900b6f17f/.github/workflows/build.yml#L26).

- See the [`docker run` command reference](https://docs.docker.com/reference/cli/docker/container/run/#description)
- `-d` is most important here, it runs as a daemon in the background so `systemd` can start within the container as PID 1
- `--name` can be anything you want to name that instance of the running container
- `--hostname` is also independant of the container image name
- `local/kali-molecule` is the same arg as `-t <tag/name>` when you either pulled or built the image

```bash
docker run -d \
  --name kali-molecule \
  --hostname kali-molecule \
  --privileged \
  --cgroupns=host \
  --tmpfs /run \
  --tmpfs /tmp \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -e container=docker \
  local/kali-molecule /sbin/init

```

*Then* interactively execute a shell in the running container once it starts:

```bash
docker exec -it kali-molecule /bin/bash

```


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