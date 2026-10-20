# Ansible development environment

Setup a machine to write Ansible roles. Includes:
- [ansible](https://github.com/ansible)
- [ansible-lint](https://github.com/ansible/ansible-lint)
- [molecule](https://molecule.readthedocs.io/en/latest/)
- [travis (cli)](https://github.com/travis-ci/travis.rb)
- [ara](https://github.com/openstack/ara)

## Download

```
ansible-galaxy collection install robertdebock.development_environment
```

## Requirements

- One machine
- About 4GB or RAM will do.
- 2 CPUs, 1 will also do.
- Personalized `inventory/group_vars/all.yml`.

## Mac / Linux

Vagrant requires a different file for `Virtualbox` (typically Mac OS X) and `libvirt` (typically Linux).

For Virtualbox / Mac OS X:

```sh
rm Vagrantfile
ln -s Vagrantfile.virtualbox Vagrantfile
```

For libvirt / Linux:


```sh
rm Vagrantfile
ln -s Vagrantfile.libvirt Vagrantfile
```

Now change a few files:

- `files/gitconfig` should contain your details.
- `files/id_rsa` should contain an ssh-key used to commit to GitHub.
- `inventory/hosts` should contain your machine.
- `inventory/group_vars/all.yml` should contain your details.

## macOS with OrbStack (no Vagrant)

On macOS you can skip Vagrant and run the whole environment inside an [OrbStack](https://orbstack.dev) Linux machine. OrbStack ships a built-in SSH server, so there is no Vagrant box and no in-VM sshd to configure.

1. Install OrbStack from https://orbstack.dev/download.
2. Create the machine (the name matters — OrbStack selects the machine by its name):

   ```sh
   orb create fedora orbstack --memory 4G --cpus 2
   ```

3. In `inventory/hosts`, uncomment the `orbstack` host in the `[orbstack]` group.
4. Personalize `inventory/group_vars/all.yml`, `files/gitconfig` and `files/id_rsa` (as for Vagrant above).

The connection facts live in `inventory/host_vars/orbstack.yml` (built-in SSH server: `localhost:32222`, key `~/.orbstack/ssh/id_ed25519`, connect as `root`). `playbook.orbstack.yml` is identical to `playbook.yml` but omits the `docker` and `vagrant` roles — OrbStack is already the Docker engine, and Apple Silicon has no nested-KVM Vagrant support.

Run it (point `-i` at the `hosts` file to skip the Vagrant-only `vagant.py` inventory script):

```sh
./playbook.orbstack.yml -i inventory/hosts
```

Afterwards, `molecule`'s docker driver and `ara` run natively on the Mac through OrbStack (no in-VM docker daemon), and the `ara` UI is available at http://localhost:9191.

## Install

- **Vagrant** (VirtualBox/libvirt): `./playbook.yml`
- **OrbStack** (macOS): `./playbook.orbstack.yml -i inventory/hosts`

Note: with the `orbstack` host uncommented it is part of `hosts: all`, so `./playbook.yml` also targets it; re-comment it while working the Vagrant flow, or comment the `fedora` host to avoid an unreachable-host error in the recap.

Preparing your system will take about 15 minutes or so (mostly package and pip installs; the `git` role also clones the configured repositories).

## Code

You are now ready to code! Have fun using these commands:

```sh
# See if your code meets all rules.
ansible-lint .
# Test all scenarios.
molecule test
# Test a specific scenario.
molecule test --scenario-name fedora-latest
```

You can see the playbook runs on https://localhost:9191/
